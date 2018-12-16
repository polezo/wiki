This tutorial will attempt to bridge the knowledge-gap between market-making on centralized cryptocurrency exchanges (e.g Binance) and market-making on 0x, a decentralised exchange protocol built on Ethereum (see: [technical protocol specification](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md)).

### High-level primer

There are three core activities all centralized exchanges perform:

1. Custody user funds
2. Host and distribute orders
3. Settle trades

Within the 0x protocol, these tasks are not all undertaken by a single entity nor in the same way.

#### 1. Custody of user funds

Trading on a CEX (centralized exchange) requires that you deposit your assets into an Ethereum addresses the exchange controls. After that point, they have full custody over your funds and must secure them from hackers, rogue employees and other attackers. When trading on 0x, your assets never need to leave your Ethereum address until a trade is settled.

#### 2. Host and distribute orders

0x orders can be hosted by anyone, rather than there being a single entity hosting the orders. Any entity that hosts and distributes 0x orders is called a "relayer". Currently most relayers are centralized entities hosting an API from which orders can be retrieved.

A 0x order is any data packet that contains an [order's details](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#order-message-format) together with a [valid signature](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#signature-types) (e.g. a valid elliptic curve signature generated from the traders Ethereum address) of it's contents.

#### 3. Settlement of trades

When two entities agree on the terms of a trade, the 0x order is submitted to the Ethereum blockchain and settled via the 0x protocol smart contract.

In order for the 0x smart contracts to move funds on the behalf of traders, the traders must first give the contracts explicit permission. This is done by setting an ["allowance"](https://tokenallowance.io/) for a certain number of assets that 0x can then transfer from their Ethereum address. Then, in the event that the trader cryptographically agrees to a trade by creating an order with a valid signature, the order can be filled and settled through the 0x smart contracts. These rules are all enforced by the open-source smart contracts deployed to the Ethereum blockchain. At no point does 0x protocol have control over the private key of the trader's Ethereum address, and the allowance can be revoked by the trader at any time.

#### What about order matching?

There are two dominant relayer strategies in existence today: [matching](https://0xproject.com/wiki#Matching) and [open orderbook](https://0xproject.com/wiki#Open-Orderbook). Open orderbook relayers host orders that anyone can fill by directly sending a transaction (with the order details and desired fill amount) to the Ethereum blockchain. Using this model, the order in which transactions are filled is determined by the same algorithm used by miners to decide which transactions to include in the next block (more on this later).

Matching relayers require traders to create the inverse 0x order to the one they wish to fill and submit it back to the matcher. The matching relayer will then fill both orders in one atomic transaction to Ethereum on the trader's behalf.

In both models, the relayer never has custody over a trader's assets.

### Creating orders

0x orders only exist off-chain and are completely free to create. In order to create a valid 0x order, you can use the [@0x/order-utils](https://0xproject.com/docs/order-utils) Typescript/Javascript library, or alternatively the [0x-order-utils.py](http://0x-order-utils-py.s3-website-us-east-1.amazonaws.com/) Python library. These libraries will help you:

1. Generate an order in the proper format (e.g encoding/decoding [`assetData`](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#assetdata))
2. Generating a proper hash for the order contents
3. Sign the order with your elliptic curve signature

For a step-by-step walk-through on the above steps, as well as allowance setting, take a look at the [Create, validate, fill order tutorial](https://0xproject.com/wiki#Create,-Validate,-Fill-Order).

Note that the `salt` field of an order should be set to the current unix timestamp in milliseconds for full compatibility with the `[cancelOrdersUpTo`](#cancelling-orders) function. For low level details of how an order is created, please reference the [orders](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#orders) section of the protocol specification.

Once you have a valid 0x order, it can be submitted to a relayer. You can use the [POST /v2/order](https://github.com/0xProject/standard-relayer-api/blob/master/http/v2.md#post-v2order) endpoint of the Standard Relayer API for this purpose (see: [submitOrderAsync in 0x Connect](https://0xproject.com/docs/connect#HttpClient-submitOrderAsync)).

If you want to be notified whenever the order is filled by a trader, you should also submit the order to an [OrderWatcher](https://0xproject.com/docs/order-watcher) instance you are running.

### Cancelling orders

There are multiple ways to cancel 0x orders, each of which is described in the [cancelling orders](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#cancelling-orders) section of the protocol specification.

The `cancelOrdersUpTo` approach is perhaps the least straight forward but also the most powerful for a market maker. It allows for the cancellation of an arbitrary number of orders for a fixed amount of gas. By creating orders where the `salt` field is equal to the current unix timestamp in milliseconds, you can cancel all orders that were created at or before a certain time in a single `cancelOrdersUpTo` call (a fixed size transaction). Note that any future orders created with a `salt` that is below the largest `salt` argument of a `cancelOrdersUpTo` call by your address will automatically be invalid.

Explicitly cancelling orders always requires an on-chain transaction. However, you may create short lived orders and replace the expired orders completely off-chain. Due to the variability of transaction inclusion times in blocks, it is not recommended to set extremely agressive expiration times for orders (empirically, most market makers set order expirations to ~5 minutes).

### Fetching off-chain orders

Since 0x orders live off-chain, they must be fetched from relayers. All relayers host APIs that allow you to pull orders from their orderbook. In order to make life easier for traders, many implement the [Standard Relayer API](https://github.com/0xProject/standard-relayer-api/), allowing you to use a single client to fetch orders from multiple relayers. Standard Relayer API clients exist in [Python](https://pypi.org/project/0x-sra-client/) and [Typescript/Javascript](https://0xproject.com/docs/connect).

To learn more about fetching orders from relayers, check out our [Find, submit, fill order from relayer tutorial](https://0xproject.com/wiki#Find,-Submit,-Fill-Order-From-Relayer).

### Trading on Ethereum

When trading on a centralized exchange, you must integrate exclusively with the CEX's trading API in order to know the state of the world (e.g orderbook, filled/cancellation state). When trading on 0x, your sources of truth are the Ethereum blockchain and the off-chain orders in existence. This section will dive deeper into the on-chain state you'll need.

#### Ethereum node management

In order to monitor the Ethereum blockchain, you must interface with an Ethereum node. You can either integrate your market maker with a hosted node service such as [Infura.io](https://infura.io/) or you can [host your own node](https://0xproject.com/wiki#How-To-Deploy-A-Parity-Node). While developing your market maker, you might want to [Setup Ganache](https://0xproject.com/wiki#Ganache-Setup-Guide), a fake Ethereum node used for testing purposes.

#### Probablistic finality

Since trades on 0x are settled by being included my miners into a block, settlement is subject to the same probabalistic finality as any other state-transition in Ethereum. Probabilistic finality means that no trade settlement is ever 100% final. It becomes exponentially more final with every additional Ethereum block that is mined ontop of the one it was included in.

What this means is that for a period of time settlement can still revert. This happens when there is a block re-organization, causing a valid next block to become "uncled" (i.e. no longer part of the main chain). It does not happen often, but it is an important edge-case to handle. [@0x/order-watcher](https://0xproject.com/wiki#0x-OrderWatcher) can help you handle all possible state changes related to the 0x orders you care about (including state reversions).

#### Block timestamps

In Ethereum, the only requirement on a block's `timestamp` is that it is greater then the previous block timestamp. What this means is that the `timestamp` of the next block could be before or after the current unix timestamp.

Because the 0x Protocol checks order expiration using block timestamps, the order will only be considered expired once a block timestamp exists that is larger then the order expiration. You can use the current time as an approximation of the block time, however there is still a chance the "expired" order gets filled if the next block timestamp falls between the last block timestamp and the current timestamp.

#### Miner discretion

In most blockchains, the miner decides which valid transactions to include in the next block they attempt to mine. They are expected to be economically rational agents who will prioritize transactions that pay them the most fees, however they have full discretion over transaction ordering.

Let's say a trader submits an order cancellation transaction; there are no guarantees about the order in which this transaction will be included in the blockchain. You can try and incentize miners to include it sooner by increasing the `gasPrice` (read: fee) paid for the transaction, but so can any other trader attempting to fill the order (read more about this problem in our [front-running, griefing and the perils of virtual settlement](https://blog.0xproject.com/front-running-griefing-and-the-perils-of-virtual-settlement-part-1-8554ab283e97) blog post series). Because of this race-condition, we recommend you use shorter expiration times on your orders rather than relying heavily on on-chain cancellations. Alternatively, you can use our [cancelOrdersUpTo](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#cancelordersupto) feature to cancel multiple orders in a single transaction.

### Developer tooling

#### What is the state of 0x developer tooling?

Currently we have the best tooling support for Javascript/Typescript and are actively improving support for Python. Visit the [developers section](https://0xproject.com/docs) of our site for a full-list of developer tools avaiable.

If you plan on implementing custom infrastructure, you will need the following functionality at a bare minimum:

-   Interacting with relayers over Websockets or HTTPS
-   Encoding the [`assetData`](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#assetdata) of an order using [ABIv2](https://solidity.readthedocs.io/en/latest/abi-spec.html)
-   Hashing orders
-   Cryptographically signing orders with the address you wish to trade with

In addition, it is highly recommended to be connected to an Ethereum node so that you can:

-   Read the state of orders
-   Be notified when the state of an order changes (by subscribing to the relevant events)
-   Set allowances to the 0x smart contracts for assets you wish to sell
-   Fill or cancel orders

#### Example projects

Here is a short list of example market making projects built on 0x:

-   [Maker's market making bot in Python](https://github.com/makerdao/market-maker-keeper)
-   [Hummingbot -- open-source market making bot](https://www.hummingbot.io/)

TODO:

-   Filling orders
-   Explanation of atomicity
-   Hedging and inventory considerations
