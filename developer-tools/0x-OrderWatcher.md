Many applications built on top of the 0x protocol will want to react to changes in an order's fillability. The canonical example is a relayer who wants to prune their orderbook of any orders that have become unfillable. Another example is a trader who wants to monitor for changes affecting the orders retrieved from a relayer.

At 0x, we've implemented an OrderWatcher to facilitate this task. It's quite an advanced tool that requires understanding the underlying mechanisms of the Ethereum blockchain, so we've written this article to walk you through our design choices and intended usage patterns.

You can use OrderWatcher with an Ethereum node of your choice.

### OrderWatcher interface

From an interface point of view, the OrderWatcher is a daemon. You can start and stop it from subscribing to order state changes, add orders you would like to track and remove orders that are no longer relevant. You can find the full interface description on the [@0x/order-watcher docs](https://0x.org/docs/order-watcher).

Once the OrderWatcher is started and an order has been added to it, it will emit orderStateChange events every time there is a change in the state backing an order's fillability (i.e order expirations, fills, cancels, etc...). These events are emitted with useful information which subscribers can use when designing custom order validation rules.

### Order validity

Order validity is not binary. Here's an example of how an order can be partially filled, cancelled and fillable at the same time: Imagine an order that sells 4 tokens. Let's say it has already been filled for 1 and the maker partially cancelled it for 1. Now the order might be fillable for 2, but the maker moves one token out of the backing address. You can now only fill it for 1.

```
filled | cancelled | unfunded | fillable
```

An order is considered valid by the OrderWatcher when it can be filled for a non-zero amount. For this to be possible the following conditions must be satisfied:

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

Below are the contract events that the OrderWatcher handles by default:

| 0x Exchange Events |
| ------------------ |
| Fill               |
| Cancel             |
| CancelUpTo         |

| Maker Asset Events | Token Type |
| ------------------ | ---------- |
| Transfer           | ERC20      |
| Approval           | ERC20      |
| Transfer           | ERC721     |
| Approval           | ERC721     |
| ApprovalForAll     | ERC721     |
| Deposit            | WETH       |
| Withdraw           | WETH       |

If any of the above events result in an change in order state you will receive a callback. [OrderState](https://0x.org/docs/order-watcher#types-OrderState) indicates whether the order is [valid](https://0x.org/docs/order-watcher#types-OrderStateValid) or [invalid](https://0x.org/docs/order-watcher#types-OrderStateInvalid), as well as the current [relevant state of the order](https://0x.org/docs/order-watcher#types-OrderRelevantState).

In the case where an event caused the order to remain [valid](https://0x.org/docs/order-watcher#types-OrderStateValid), but changed the amount fillable (e.g., a `Fill` or `Transfer` event) the order state will contain the current calculated relevant state:

```typescript
isValid: true,
orderRelevantState: {
    // The taker amount filled for this order
	filledTakerAssetAmount: BigNumber,
    // The remaining amount that is fillable given the makers balance and allowances, in maker asset
	remainingFillableMakerAssetAmount: BigNumber,
    // The remaining amount that is fillable given the makers balance and allowances, in taker asset
	remainingFillableTakerAssetAmount: BigNumber,
    // The balance of maker asset
	makerBalance: BigNumber,
    // The allowance of maker asset to the ERC20 Transfer Proxy
	makerProxyAllowance: BigNumber,
    // The maker balance of ZRX
	makerFeeBalance: BigNumber,
    // The maker allowances of ZRX to the ERC20 Transfer Proxy
	makerFeeProxyAllowance: BigNumber,
},
orderHash: string,
transactionHash: undefined|string,
```

Note: In some circumstances `filledTakerAssetAmount + remainingFillableTakerAssetAmount` may be lower than the order's total `takerAssetAmount`. This is because OrderWatcher calculates how much is remaining to be filled given the maker's balance and allowance for the asset (as well as fees, if any).

Sometimes an event occurs which causes the order to become [invalid](https://0x.org/docs/order-watcher#types-OrderStateInvalid). Here are a few common causes:

-   Order has expired
-   Order has been fully filled
-   Maker has cancelled the order
-   Maker has reduced their balance of the token to 0
-   Maker has reduced their allowance of the token to 0

When this occurs the order state will be the following:

```typescript
isValid: false,
error: ExchangeContractErrs,
orderHash: string,
transactionHash: undefined|string,
```

The [errors](https://0x.org/docs/order-watcher#types-ExchangeContractErrs) which can invalidate the order include:

| Exchange State               | Description                                                |
| ---------------------------- | ---------------------------------------------------------- |
| OrderCancelled               | The order has been cancelled on chain                      |
| OrderRemainingFillAmountZero | The order has been fully filled                            |
| OrderFillRoundingError       | The order results in a rounding error and cannot be filled |
| OrderFillExpired             | The order has expired                                      |

| Maker State                   | Description                                          |
| ----------------------------- | ---------------------------------------------------- |
| InsufficientMakerBalance      | The maker has 0 maker asset balance                  |
| InsufficientMakerAllowance    | The maker has 0 maker asset allowance                |
| InsufficientMakerFeeBalance   | The maker has 0 ZRX balance and the order has fees   |
| InsufficientMakerFeeAllowance | The maker has 0 ZRX allowance and the order has fees |

Note: The exchange state is quite final and it is best to remove these orders immediately from your orderbook. The maker state can change in the future so it is possible to hide these orders for some period of time. For example, the maker could be in the process of acquiring more maker asset or is yet to set allowances.

It is possible to receive multiple updates for an order as OrderWatcher processes the block. Read more about re-orgs in the State finality section.

### Naive approach

The naive approach to order watching is to write a worker service that simply iterates over a set of orders, calls the [contractWrappers.exchange.validateOrderFillableOrThrowAsync](https://0x.org/docs/0x.js/#ExchangeWrapper-validateOrderFillableOrThrowAsync) method on each one, and discards those that are no longer fillable. This method checks the last three conditions listed above.

OrderWatcher takes a more sophisticated approach by mapping each order to the underlying state that could impact its validity. Whenever the underlying state changes, it knows exactly which orders need to be re-evaluated. Since there are still edge cases in our current approach (more details on this later), OrderWatcher also runs a naive iterator on a lengthier configurable interval to clean up orders that might have been missed.

### State finality

OrderWatcher works on the state layer with one confirmation (latest block). This means it will react to events emitted from transactions that have been mined into the latest block. These events can still be reverted if the latest block gets uncled (i.e., during a block re-org). In these cases, OrderWatcher will handle the block re-org correctly and re-emit an event correcting the order's validity.

In the event where a block re-org occurs you will also be notified with a state update for the removal of the log. For example, if a `Fill` event occurs (OrderWatcher will notify with a state update) followed by a block re-org removing the `Fill`, you will be notified with another state update.

### Understanding blockchain state layers

Ethereum is a state machine with different state layers, each with its own degree of certainty and latency. The first state layer includes all the newest transactions mined within the latest block. It is still quite likely to change since many miners are working to mine the next block, whereas transactions with 15 block confirmations are highly unlikely to change. Waiting for 15 confirmations however, has the noticeable downside of taking ~3m45s.

JSON RPC allows the caller to specify the state layer they want to access by [specifying a block number](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_call), 'latest' or 'pending'.

The easiest thing to do would be to simply validate orders at an “immutable” state layer (say 10 confirmations) and prune the order only if it is invalid at that point in time. However, doing so would mean that our understanding of the market is outdated by up to 3mins. This amount of latency is unacceptable.

On the other hand, we can't afford to react immediately to changes in the first state layer because the transaction making an order invalid might not end up in the canonical chain due to chain re-orgs. As such, we would have discarded a valid order.

The solution we are proposing here is to shadow orders that are deemed invalid rather then immediately removing them from our set of orders. In the relayer orderbook pruning example, as soon as the order appears invalid within the first state layer we recommend flagging it and not broadcasting it to UI/API clients. If the order becomes valid again later (e.g., due to a chain re-org) it should be unshadowed and shown once again. Since we don't want our DB to grow indefinitely, it might make sense to also run an iterative cleanup worker that checks the validity of flagged orders at a much higher confirmation depth and removes them conclusively if they are deemed invalid.

Lifeycle of an order:

<div align="center">
    <img src="https://s3.eu-west-2.amazonaws.com/0x-wiki-images/order_states.png" style="padding-bottom: 20px; padding-top: 20px; max-width: 507px;" width="80%" />
</div>

### Performance optimizations

A new block is mined approx. every ~12sec and if we want to keep our order book up-to-date, we need to revalidate all orders we are tracking every time this happens. Since this can happen every couple of seconds and we might be tracking millions of orders, an iterative approach will not be performant enough.

Let's look into order validation more deeply and optimize it. When the order is already on an order book and its schema and signature have already been validated, order validation is essentially a [pure function](https://en.wikipedia.org/wiki/Pure_function) that depends on blockchain state and time.

<div align="center">
    <img src="https://s3.eu-west-2.amazonaws.com/0x-wiki-images/order_state_deps.png" style="padding-bottom: 20px; padding-top: 20px; max-width: 627px;" width="80%" />
</div>

Time is the easy part. OrderWatcher simply pushes your orders onto a min heap by expiration time and emits events whenever they expire. You might want to be notified before they expire. To do so, you can configure OrderWatcher to notify you X seconds before an order expires. If an order expires in 10 seconds, there is no reason to keep it on the order book because there is a very small probability the transaction will make it into a block within 10 seconds.

Blockchain state is slightly more complex. We'd like to get notified when any of those four state fields in the above diagram change for any order. For ERC20 tokens, token balance and allowance changes MUST emit a corresponding event ([Transfer](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20-token-standard.md#transfer-1) & [Approval](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20-token-standard.md#approval) respectively). We can listen for those events on the pending state level and use them as proxies for underlying state changes.

Unfortunately not every balance change is a transfer. Some tokens implement additional logic like mint/burn and there is no standard for emitting events in such cases. This means the OrderWatcher might miss some state changes. That's why we've implemented a cleanup job that periodically runs over all orders and checks them iteratively. This iterative process can run much more infrequently then if it were used as the primary mechanism for order watching.

<div align="center">
    <img src="https://s3.eu-west-2.amazonaws.com/0x-wiki-images/state_to_order_mapping.png" style="padding-bottom: 20px; padding-top: 20px; max-width: 761px;" width="80%" />
</div>

### Infrastructure requirements (to watch at mempool level)

### Future work

Smart contract events are only a proxy for state changes, not the state change itself. What we're actually interested in is the results of balanceOf and allowance functions. [EIP 781](https://github.com/ethereum/EIPs/issues/781) proposes a way to watch arbitrary state directly in a performant way. It was created as a result of our OrderWatcher research and we're putting resources towards making it a reality. OrderWatcher v2 will remove its reliance on events, making it more robust and eliminating the need for a cleanup job.
