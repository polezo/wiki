# I'm new to Ethereum Blockchain Development
Welcome, we're glad you have decided to explore the possibilities of developing on the Ethereum Blockchain. With Ethereum you can deploy applications or Smart Contracts that perform operations and persist state when executed. This "transaction" executes and is confirmed and verified by hundreds of different machines (called nodes) distributed around the world. If you'd like to read more about the Ethereum Blockchain or Blockchains in general, read on here.

Now that you have a basic understanding of what is happening on Ethereum its time to talk about Tokens. Tokens (commonly referred to as [ERC20](https://github.com/ethereum/EIPs/issues/20)) are a simple smart contract that tracks the token balance of an account. You can request for this smart contract to [transfer](https://github.com/OpenZeppelin/zeppelin-solidity/blob/master/contracts/token/ERC20/BasicToken.sol#L31) tokens from your balance to anyone else. Others can also transfer tokens from their balance to you. All of these requests are transactions on the Ethereum Blockchain and cost a small transaction fee.

### 0x Protocol
The 0x Protocol extends on this simple concept, our smart contract performs multiple balance transfers in one operation. If Alice wants to exchange 100 of TokenA for 200 TokenB with Bob, Alice would want to make sure Bob had enough tokens. Rather than Alice sending her amount of TokenA to Bob and hoping Bob will send TokenB to Alice, the 0x Protocol performs this exchange as a guaranteed atomic operation. So Alice and Bob will get exactly what they expected. The 0x Protocol is designed to minimise the amount of transactions (and fees) required for Alice and Bob to successfully exchange tokens. We refer to this as off-chain orders with on-chain settlement. You can find more information on the 0x Protocol by reading our [whitepaper](https://0xproject.com/pdfs/0x_white_paper.pdf).

### Interacting with the Ethereum Blockchain
As web browsers do not natively support interacting with the Ethereum Blockchain, a browser extension called [Metamask](https://metamask.io/) fills that void. Metamask will handle the users keys and allow them to submit transactions to the Ethereum Network via a Node. To interact with dApps (distributed Apps) the Ether funds must be under the users control. The user cannot interact from an exchange such as Coinbase. For mobile users there are applications like Toshi and Cipher Browser. 

### Development

#### Test Networks
Testing and development is aided by running your own local test network. This allows you to quickly deploy contracts without it costing real ETH. [Ganache](https://github.com/trufflesuite/ganache) is the best tool for a local development network. This software is an approximation of an actual network, so we recommend testing on Kovan, Rinkeby, or Ropsten before going to Mainnet.

It is often difficult to get a sufficient amount of ETH to begin your testing on these networks. We have created a faucet which can dispense small amounts of ETH to your account. Navigate over to the [0x Portal](https://0xproject.com/portal/balances) with Metamask installed while you're connected to the Kovan Network and click Request to receive from free test ETH!

<img width="1176" alt="kovan-testnet" src="https://user-images.githubusercontent.com/27389/36271820-9bfde1c2-1234-11e8-9117-c7c4d5656d59.png">

#### Hosted Network Nodes
Rather than running your own Ethereum Node to connect to a network, many developers utilize [INFURA](https://infura.io/) who provide the public with a gateway to Ethereum. You can use INFURA to connect to Main Network and various test networks like Kovan.

### Wrapped ETH
By now you are likely familiar with the ERC20 standard, ETH and the plethora of ERC20 tokens out there. Unfortunately, ETH is not an ERC20 token and because of this interacting with both ERC20 tokens and ETH at the same time can lead to scenarios which adds complexity to your smart contracts. Less code is better, it is easier to understand and audit. One technique the community has adopted is to adapt the interface of ETH to behave like an ERC20 token. This allows one to treat everything as an ERC20 token and reduces smart contract complexity. We call this WETH or Wrapper ETH. The owner of the ETH would deposit, for example, 1ETH into the WETH contract and they would receive back 1 WETH token. When they are finished with the WETH they withdraw back their 1ETH in exchange for 1WETH. This idea is becoming increasingly common in the ecosystem and more information on WETH can be found [here](https://weth.io/).

Now that you have been given a high level overview of Ethereum Blockchain development and how to get started, it is time to check out our other tutorials and examples.
