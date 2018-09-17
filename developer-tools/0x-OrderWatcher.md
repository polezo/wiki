Many applications built on top of the 0x protocol will want to react to changes in an order's fillability. The canonical example is a relayer wanting to prune their orderbook of any orders that have become unfillable. Another example is a trader using the standard relayer API who wants to react quickly to orders retrieved from relayers becoming unfillable.

At 0x - we've implemented an orderWatcher to facilitate this task. It's quite an advanced tool that requires understanding the underlying mechanisms involved and so we've written this article to walk you through the design choices we made and how to use it.

You can use OrderWatcher with the Ethereum node of your choice.

### OrderWatcher interface

From an interface point of view - the orderWatcher is a daemon. You can start & stop its subscription to order state changes as well as add orders you would like to track and remove orders that are no longer relevant. You can find the full interface description on the [@0xproject/order-watcher docs](https://0xproject.com/docs/order-watcher).

Once the orderWatcher is started and an order has been added to it, it will emit orderStateChange events every time any of the state backing an order's fillability (i.e order expirations, fills, cancels, etc...) changes. There events are emitted with all the necessary information for the subscriber to then decide with their own custom rules whether or not they consider the order still valid.

### Order Validity

Order validity is not binary. An order can be partially filled, cancelled and fillable at the same time. Imagine an order that sells 4 tokens. Let's say it has already been filled for 1 and the maker partially cancelled it for 1. Now the order might be fillable for 2, but the maker moves one token out of the backing address. You can now only fill it for 1.

|filled|cancelled|unfunded|fillable|

An order is considered valid by the orderWatcher when it can be filled for a non-zero amount. For this to be possible the next conditions need to be satisfied:

-   It has a valid structure (schema)
    -   This is checked before accepting the order
-   It has a valid signature
    -   This is checked before accepting the order
-   It's not expired
    -   Is dynamic and depends on time
-   It's not fully filled or cancelled
    -   Is dynamic and depends on blockchain state
-   Maker has enough funds & allowance to perform the trade and pay fees
    -   Is dynamic and depends on blockchain state

You're free to implement your own logic on top of this information. Some decisions a relayer must make are:

-   Whether to accept orders that are partially fillable due to the maker having less available funds or allowance than the order specifies (unfunded - not to be confused with partially filled)
-   What the minimum partially fillable amount is that they are willing to accept (this lets them avoid dust orders)
-   Whether to punish users whose orders become unfillable through an allowance/balance change (griefing)

### Naive approach

The naive approach to order watching is to write a worker service that simply iterates over a set of orders, and calls the [zeroEx.exchange.validateOrderFillableOrThrowAsync](https://0xproject.com/docs/0x.js/#exchange-validateOrderFillableOrThrowAsync) method on each one, discarding those that are no longer fillable. This method checks the last three conditions listed above.

The orderWatcher takes a more sophisticated approach to this problem by mapping each order to the underlying state that could impact its validity. Whenever the underlying state changes, it knows exactly which orders need to be re-evaluated. Since there are still edge-cases in our current approach (more details on this later), the orderWatcher also runs a naive iterator on a lengthier configurable interval in order to clean up any orders that might have been missed.

### State finality

The order watcher works on the state layer with 1 confirmation (latest block). This means it will react to events emitted from transactions that have been mined into the latest block. These events could still be reverted if this latest block gets uncles (i.e during a block re-org). In these cases, the OrderWatcher will handle the block-reorg correctly and re-emit an event correcting the orders validity.

### Understanding blockchain state layers

Ethereum is a state-machine with different state layers, each with its own degrees of certainty and latency. The 1st state layer includes all the newest transactions mined within the latest block. It is still quite likely to change since many miners are working to mine the next block, whereas transactions with 15 block confirmations are highly unlikely to change. Waiting for 15 confirmations however, has the noticeable downside that it takes ~3m45s for a transaction to reach that state.

JSON RPC allows the caller to specify the state layer they want to access by [specifying a block number](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_call), 'latest' or 'pending'.

The easiest thing to do would be to simply validate orders at an “immutable” state layer (say 10 confirmations) and only if an order is invalid at that point, prune the order. Doing so however, would mean that our understanding of the market is outdated by up to 3mins. This is an unacceptably large amount of latency.

On the other hand - we can't afford to react immediately to changes in the 1st state layer because the transaction making an order invalid might not end up being included in the canonical chain due to chain reorgs and we will have discarded a valid order.

The solution we are proposing here is to shadow orders that are deemed invalid rather then removing them from our set of orders immediately. In the relayer orderbook pruning example, as soon as the order appears invalid within the 1st state layer we recommend flagging it as such and to stop broadcasting it to UI/API clients. If the order becomes valid again later (e.g due to a chain re-org) it should be unshadowed and the order shown once again. Since we don't want our DB to grow indefinitely - it might make sense to still run an iterative cleanup worker that checks the validity of flagged orders at a much higher confirmation depth and removes them conclusively if they are deemed invalid at that point.

Lifeycle of an order:

<div align="center">
    <img src="https://s3.eu-west-2.amazonaws.com/0x-wiki-images/order_states.png" style="padding-bottom: 20px; padding-top: 20px; max-width: 507px;" width="80%" />
</div>

### Performance optimizations

A new block is mined approx. every ~12sec and if we want to keep our order book up to date - we need to revalidate all orders we are tracking every time this happens. Since this can happen every couple of seconds and we might be tracking millions of orders, an iterative approach will not be performant enough.

Let's look into order validation more deeply and optimize it. When the order is already on an order book and its schema and signature have already been validated - order validation is essentially a [pure function](https://en.wikipedia.org/wiki/Pure_function) that depends on blockchain state and time.

<div align="center">
    <img src="https://s3.eu-west-2.amazonaws.com/0x-wiki-images/order_state_deps.png" style="padding-bottom: 20px; padding-top: 20px; max-width: 627px;" width="80%" />
</div>

Time is the easy part. OrderWatcher simply pushes your orders onto a min heap by expiration time and emit events whenever they expire. You might want to be notified before they expire. To do so, you can configure orderWatcher to notify you X seconds before an order expires. If an order expires in 10 seconds - there is no reason to keep it on the order book, because there is a very small probability the transaction will make it into a block within 10 seconds.

Blockchain state is slightly more complex. We'd like to get notified when any of those four state fields in the above diagram change for any order. Luckily those are token balance and allowance changes. According to the ERC20 standard, any token transfer/allowance change MUST emit a corresponding event ([Transfer](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20-token-standard.md#transfer-1) & [Approval](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20-token-standard.md#approval) respectively). We can therefore listen for those events on the pending state level and use it as a proxy for the underlying state change.

Unfortunately not every balance change is a transfer. Some tokens implement additional logic like mint/burn and there is no standard about emitting events in such cases. This means the orderWatcher might miss some state changes. That's why we've implemented a cleanup job that periodically runs over all orders and checks them iteratively. This iterative process can run much more infrequently then if it were used as the primary mechanism for order watching.

<div align="center">
    <img src="https://s3.eu-west-2.amazonaws.com/0x-wiki-images/state_to_order_mapping.png" style="padding-bottom: 20px; padding-top: 20px; max-width: 761px;" width="80%" />
</div>

### Infrastructure requirements (to watch at mempool level)

Future work

Smart contract events are only a proxy for state changes, not the state change itself. What we're actually interested in is the results of balanceOf and allowance functions. [EIP 781](https://github.com/ethereum/EIPs/issues/781) proposes a way to watch arbitrary state directly in a performant way. It was created as a result of our OrderWatcher research and we're putting resources towards making it a reality. OrderWatcher v2 will remove its reliance on events, making it more robust and eliminating the need for a cleanup job.
