A Relayer is an application responsible for creating and allowing users to discover 0x orders. It is a layer that sits above the 0x Exchange smart contract. Picture it as the interface a user would see and interact with. The 0x Exchange smart contract handles the core exchange functionality.

## Advantages of a Decentralised Exchange
There are many advantages to a Decentralised Exchange versus a Centralised Exchange. One of the largest problems with Centralised Exchanges are their deposits into a "hot" wallet. This often holds millions of dollars of tokens. These wallets have historically been the target of a hack, with hundreds of millions of dollars worth of tokens stolen. With a Decentralised Exchange, a user trades from their own wallet. Eliminating the attack vector of a single "hot" wallet.

On 0x, there is no deposit or withdraw, so users are able to trade across all Relayers. In fact, Relayers will often display eachothers orders to increase their Orderbook depth. Having no deposit reduces the transaction fees for a user and allows them to remain in control of their funds at all time. Users can even trade directly from their hardware device such as a Ledger Nano S. 

## 0x Protocol Overview
In 0x protocol, orders are transported off-chain, massively reducing gas costs and eliminating blockchain bloat. Relayers help broadcast orders and collect a fee each time they facilitate a trade. Anyone can build a relayer.

The simplest example of a Relayer is a website allowing users to create, discover and fill orders. The Relayer is the user interface infront of all of this.

![Relayer](https://0xproject.com/images/landing/relayer_diagram.png)

To aid the process of creating a Relayer we have created a Javascript library called [0x.js](https://github.com/0xProject/0x-monorepo/tree/development/packages/0x.js). This library helps create and fill orders using the 0x Exchange contract. 

Before getting started with 0x.js and the 0x protocol, it is helpful to introduce a few concepts. There are two sides to every exchange, makers and takers. The maker creates an order for an amount of TokenA in exchange for an amount of TokenB. Makers create orders and submit them to a Relayer. Takers discover orders via a Relayer and submit these to the Exchange contract. The Exchange contract performs an atomic swap, exchanging the maker and taker tokens.

## Order
Below is an interface description of what makes up an Order in 0x protocol:
```typescript
interface Order {
    // Ethereum address of the Maker
    maker: string;
    // Ethereum address of the Taker. This is usually empty until a taker submits it to the Blockchain
    taker: string;
    // How many ZRX the Maker will pay as a fee to the Relayer
    makerFee: BigNumber;
    // How many ZRX the Taker will pay as a fee to the Relayer
    takerFee: BigNumber;
    // The amount of tokens the Maker is offering to pay in Exchange
    makerTokenAmount: BigNumber;
    // The amount of tokens the Maker is willing to accept in Exchange
    takerTokenAmount: BigNumber;
    // The address of token the Maker is offering
    makerTokenAddress: string;
    // The address of token the Maker is requesting
    takerTokenAddress: string;
    // Random number for uniqueness
    salt: BigNumber;
    // The address of the 0x Exchange contract
    exchangeContractAddress: string;
    // The address that will receive the fees
    feeRecipient: string;
    // When the order will expire (in unix time)
    expirationUnixTimestampSec: BigNumber;
};
```

Now you should see that a Relayers job is to collect these orders in an off-chain database. This collection of orders is called an Orderbook. A Relayer displays the Orderbook to potential takers. The incentive here is for a Relayer to collect fees from the orders. By being the fee recipient Relayers can earn fees in ZRX token.

We have a tutorial on how to [Create, Validate and Fill Orders](https://0xproject.com/wiki#Intro-Tutorial:-Create,-Validate,-Fill-Order) if you're ready to jump in and start developing on 0x. This tutorial will take you through setting up 0x.js, creating an order and show you how to sign an order.

## Relayer Strategies 
There are many strategies a Relayer can choose from. The simplest and most basic is an Open Orderbook strategy. This allows anyone to take the order and submit it to the Exchange contract. An Over The Counter (OTC) strategy defines a Taker address. Only the specific Taker can be a counter party to that exchange. For more information and more strategies, read our wiki article on [Relayer Strategies](https://0xproject.com/wiki#Open-Orderbook)

## Shared Liquidity
One great feature of 0x Protocol is the concept of shared liquidity. There is a self-reinforcing cycle where low liquidity results in a low number of users. Using shared liquidity helps to solve this. Using shared liquidity a Relayer is able increase the liquidity of their Orderbooks by including orders from other Relayers. We have defined a number of API endpoints called the [Standard Relayer API](https://github.com/0xProject/standard-relayer-api) to help Relayers share liquidity.

## Pruning your Orderbooks
Over time, orders may expire, trades may execute and tokens may be sent. It is best to keep your Orderbooks nice and clean. Since a User trades out of their wallets, they are also able to send tokens out at any time. This has the potential to change the validity of an order. It is important to make sure that all the orders are still valid. Iterating over the Orderbook and checking is one simple way, a more advanced way is to watch for events relating to orders using the [OrderWatcher](https://0xproject.com/wiki#0x-OrderWatcher).

## Next Steps

Now that you have a high level idea of what a Relayer is, it's time to get started learning how to [Create, Validate and Fill Orders](https://0xproject.com/wiki#Intro-Tutorial:-Create,-Validate,-Fill-Order). You may also want to decide on a [Relayer Strategy](https://0xproject.com/wiki#Open-Orderbook), we recommend the Open Orderbook strategy. If you're looking for more orders to add to your Orderbook, take a look at the [Standard Relayer API](https://github.com/0xProject/standard-relayer-api). Remember to keep your Orderbook fresh and pruned by using the [OrderWatcher](https://0xproject.com/wiki#0x-OrderWatcher).
