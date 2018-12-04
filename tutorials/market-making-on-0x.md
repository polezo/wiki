This tutorial will attempt to bridge the knowledge-gap between market-making on centralized cryptocurrency exchanges (e.g Binance) and market-making on 0x, a decentralised exchange protocol built on Ethereum (see: [technical protocol specification](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md)).

### High-level primer

There are three core activities all centralized exchanges perform:

1. Custody user funds
2. Host and distribute orders
3. Settle trades

Within the 0x protocol, these tasks are not all undertaken by a single entity nor in the same way.

#### 1. Custody of user funds

Trading on a CEX requires that you deposit your Ether/tokens into an Ethereum addresses the exchange controls. After that point, they have full custody over you funds, and must secure them from hackers, rogue employees and other mishaps. When trading on 0x, your funds never leave an Ethereum address you control.

#### 2. Host and distribute orders

Instead of there being a single entity hosting 0x orders, they can be hosted by anyone. Any entity that hosts and distributes 0x orders is called a "relayer". Currently most relayers are centralized entities hosting an API from which orders can be retrieved.

A 0x order, is any data packet that contains an [order's details](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#order-message-format) together with a [valid signature](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#signature-types) (e.g., a valid elliptic curve signature generated from the traders Ethereum address) of it's contents.

#### 3. Settlement of trades

When two entities agree on the terms of a trade, the 0x order is submitted to the Ethereum blockchain and settled via the 0x protocol smart contract.

In order for 0x to move funds on the behalf of others, traders must give the 0x smart contract explicit permission. This is done by setting an ["allowance"](https://tokenallowance.io/) for a certain number of tokens that 0x can then transfer from their Ethereum address in the event that the trader cryptographically agrees to a trade (and not for any other purpose). These rules are all enforced by the open-source smart contracts deployed to the Ethereum blockchain. At no point does 0x protocol have control over the private key of the traders Ethereum address, and the allowance can be revoked at any time.

#### What about order matching?

There are two dominant relayer strategies in existence today, [matching](https://0xproject.com/wiki#Matching) and [open orderbook](https://0xproject.com/wiki#Open-Orderbook). Open orderbook relayers host orders that anyone can fill by directly sending a transaction (with the order details and desired fill amount) to the Ethereum blockchain. Using this model, the order in which transactions are filled is determined by the same algorithm used by miners to decide which transactions to include in the next block (more on this later). Matching relayers require traders to create the inverse 0x order to the one they wish to fill and submit it back to them. The matching relayer will then submit both orders in one atomic transaction to Ethereum on the traders behalf.

In both models, the relayer never has custody over a trader's funds.

#### What is the state of 0x Developer Tooling?

Currently we have the best tooling support for Javascript/Typescript and are actively improving our support for Python. Visit the [Developers section](https://0xproject.com/docs) of our site for a full-list of developer tools avaiable.

### Trading on Ethereum

When trading on a centralized exchange, you must integrate exclusively with their trading API in order to know the state of the world (e.g orderbook, filled/cancellation state). When trading on 0x, your sources of truth are the Ethereum blockchain and the off-chain orders in existence.

#### Ethereum node management

In order to monitor the Ethereum blockchain, you must interface with an Ethereum node. You can either integrate your market maker with a hosted node service such as [Infura.io](https://infura.io/) or you can [host your own node ](https://0xproject.com/wiki#How-To-Deploy-A-Parity-Node). While developing your market maker, you might want to [Setup Ganache](https://0xproject.com/wiki#Ganache-Setup-Guide), a fake Ethereum node used for testing purposes.

#### Probablistic finality

Since trades on 0x are settled by being included my miners into the next block, settlement is subject to the same probabalistic finality as any other state-transition in Ethereum. Probabilistic finality means that no trade settlement is ever 100% final. It becomes exponentially more final with every additional Ethereum block that is mined ontop of the one it was included in.

<ADD IMAGE OF BLOCK DEPTH W/ PROBABILITY & UNCLED BLOOCK>

What this means is that for a period of time settlement can get reverted. This happens when there is a block re-org, causing a valid next block to become "uncled" (e.g no longer part of the main chain). It does not happen often, but it is an important edge-case to handle. We built [@0x/order-watcher](https://0xproject.com/wiki#0x-OrderWatcher) to help you handle all possible state changes related to the 0x orders you care about (including state reversions).

#### Block timestamps

In Ethereum, the only requirement on a block's `timestamp` is that it is greater then the last timestamp. What this means is that the `timestamp` of the next block could be before or after the current unix timestamp.

<ADD IMAGE OF TIMELINE>

Because the 0x Protocol checks order expiration using block timestamps, the order will only be considered expired once a block timestamp exists that is larger then the expiry. You can use the current timestamp as an approximation, however there is still a chance the order get's filled if the block timestamp falls between the last block timestamp and the current timestamp.

#### Miner discretion

In most blockchains, the miner can decide which valid transactions to include in the next block they attempt to mine. They are expected to be economically rational agents who will prioritize transactions that pay them the most fees, however they have full discretion over transaction ordering.

Let's say one submits an order cancellation transaction; there are no guarentees about the order in which this transaction will be included in the blockchain. You can try and incentize miners to include it sooner by increasing the gas (read: fee) paid for the transaction, but so can someone trying to fill the order (read more about this problem in our [front-running, griefing and the perils of virtual settlement](https://blog.0xproject.com/front-running-griefing-and-the-perils-of-virtual-settlement-part-1-8554ab283e97) blog post series). Because of this race-condition, we recommend you use shorter expiration times on your orders then rely heavily on cancellations. Alternatively you can use our [cancelOrdersUpTo](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#cancelordersupto) feature to cancel multiple orders in a single transaction.

### Fetching off-chain orders

Since 0x orders live off-chain, they must be fetched from Relayers. All relayers host API's that allow you to pull orders from their orderbook. In order to make life easier for traders, many implement the [Standard Relayer API](https://github.com/0xProject/standard-relayer-api/), allowing you to use a single client to fetch orders from multiple relayers. This is exactly what [0x Connect](https://0xproject.com/docs/connect) does for you. It is an HTTP and WebSocket client for the Standard Relayer API.

To learn more about fetching orders from relayers, check out our [Find, submit, fill order from relayer tutorial](https://0xproject.com/wiki#Find,-Submit,-Fill-Order-From-Relayer).

### Creating orders

In order to create a valid 0x order, you can use our [@0x/order-utils](https://0xproject.com/docs/order-utils) Typescript/Javascript library, or alternatively our [0x-order-utils.py](http://0x-order-utils-py.s3-website-us-east-1.amazonaws.com/) Python library. There libraries will help you:

1. Generate an order in the proper format (e.g encoding/decoding assetData)
2. Generating a proper hash for the order contents
3. Sign the order with your elliptic curve signature

For a step-by-step walk-through on doing the above, as well as allowance setting, take a look at our [Create, validate, fill order tutorial](https://0xproject.com/wiki#Create,-Validate,-Fill-Order).

Once you have a valid 0x order, you need to submit it to a relayer. You can use the [POST /v2/order](https://github.com/0xProject/standard-relayer-api/blob/master/http/v2.md#post-v2order) endpoint of the Standard Relayer API for this purpose (see: [submitOrderAsync in 0x Connect](https://0xproject.com/docs/connect#HttpClient-submitOrderAsync)).

If you want to be notified whenever the order is taken by a trader, you should also submit the order to an [OrderWatcher](https://0xproject.com/docs/order-watcher) instance you are running.

### Cancelling orders

There are multiple ways to cancel 0x orders, each of which is described in the [Cancelling orders](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#cancelorder) section of the protocol specification.

The `cancelOrdersUpTo` approach is perhaps the least straight forward but also the most powerful for a market maker. It allows for the cancellation of a whole swath of orders for a fixed amount of gas. One suggested way of using it is to set the `salt` in orders with prices closer to the market price with a lower value then those with prices further out.

If there is a disadvantageous price movement in the market, you can then calculate how many of the orders closer to the previous market price to cancel. You can then make a single `cancelOrdersUpTo` request that will cancel all orders with a `salt` value below your chosen value.

### Example Projects

-   [Maker's market making bot in Python](https://github.com/makerdao/market-maker-keeper)
-   [Hummingbot -- open-source market making bot](https://www.hummingbot.io/)

// TODO: Add more
