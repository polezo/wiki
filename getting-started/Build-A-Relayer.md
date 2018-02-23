A Relayer is an application responsible for creating and discovering 0x orders. This concept is a layer above the 0x Exchange smart contracts. Picture it as the interface a user would see and interact with. The 0x Exchange smart contract handles the core exchange functionality.

## 0x Protocol Overview
In 0x protocol, orders are transported off-chain, massively reducing gas costs and eliminating blockchain bloat. Relayers help broadcast orders and collect a fee each time they facilitate a trade. Anyone can build a relayer.

The simplest example of a Relayer is a website allowing users to create, discover and submit orders. The user interface is the responsibility of the Relayer.

![Relayer](https://0xproject.com/images/landing/relayer_diagram.png)

To aid this process we have written a Javascript library called 0x.js. This library helps create and submit orders to the Exchange contract. 

Before getting started with 0x.js and the 0x protocol, it is helpful to introduce a few concepts. There are two sides to every exchange, makers and takers. The maker creates an order for an amount of TokenA in exchange for an amount of TokenB. Makers create orders and submit them to a Relayer. Takers discover orders via a Relayer and submit these to the Exchange contract. The exchange contract performs an atomic swap, exchanging the maker and taker tokens.

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
    // The address that will receive ALL fees
    feeRecipient: string;
    // When the order will expire (in unix time)
    expirationUnixTimestampSec: BigNumber;
};
```

You can see that a Relayers job is to collect these orders in an off-chain database. This collection of orders is an Orderbook. A Relayer displays the Orderbook to potential takers. The incentive here is for a Relayer to collect fees from the orders. By being a fee recipient Relayers can earn fees in ZRX token.

We have a tutorial on how to [Create, Validate and Fill Orders](https://0xproject.com/wiki#Intro-Tutorial:-Create,-Validate,-Fill-Order) if you're ready to start developing on 0x. This tutorial will take you through setting up 0x.js, creating an order and show you how to sign an order.

There are many strategies a Relayer can choose from. The simplest and most basic is an Open Orderbook strategy. This allows anyone to take the order and submit it to the Exchange contract. An Over The Counter (OTC) strategy defines a Taker address. Only the specific Taker can be a counter party to that exchange. For more information read our wiki article on [Relayer Strategies](https://0xproject.com/wiki#Open-Orderbook)

## Shared Liquidity
One great feature of 0x Protocol is the idea of shared liquidity. There is a self-reinforcing cycle where low liquidity results in a low number of users.  We have defined a number of API endpoints called the [Standard Relayer API](https://github.com/0xProject/standard-relayer-api).  By making use of shared liquidity, a Relayer is able increase the liquidity of their Orderbooks.



* ~~What is a relayer~~
* ~~Makers~~
* ~~Takers~~
* ~~OrderBook~~
* ~~OrderBook Strategy~~
* ~~0x.js~~
* ~~0x Exchange~~
* ~~Shared liqduitiy~~
* Advantages of a Relayer over a Centralised exchange