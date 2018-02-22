# I'm new to Ethereum Blockchain Development
Welcome to the exciting world of building applications on the [Ethereum Blockchain](https://www.ethereum.org/). With Ethereum you can deploy applications or "Smart Contracts" that perform operations and persist state when executed. Ethereum is often described as programmable digitial money. A transaction executes and is confirmed and verified by hundreds of different machines (nodes) distributed around the world. If you'd like to read more about the Ethereum Blockchain read on [here](https://blog.coinbase.com/a-beginners-guide-to-ethereum-46dd486ceecf).

Now that you have a basic understanding of what is happening on Ethereum it is time to talk about tokens. Tokens (commonly referred to as [ERC20](https://github.com/ethereum/EIPs/issues/20)) are simple smart contracts that track the token balance of an account. You can request for this smart contract to [transfer](https://github.com/OpenZeppelin/zeppelin-solidity/blob/master/contracts/token/ERC20/BasicToken.sol#L31) tokens from your account to anyone else's. Others may also transfer tokens from their account to you. All of these requests are transactions on the Ethereum Blockchain and cost a small transaction fee. For more information check out the [beginner's guide to Ethereum Tokens](https://blog.coinbase.com/a-beginners-guide-to-ethereum-tokens-fbd5611fe30b).

### 0x Protocol
The 0x Protocol extends on this functionality by performing multiple balance transfers in a single operation. If Alice wants to exchange 100 of TokenA for 200 TokenB with Bob, Alice would want to make sure Bob had enough tokens. Rather than Alice sending her amount of TokenA to Bob and hoping Bob will send TokenB to Alice sometime in the future, the 0x Protocol checks these requirements and performs this exchange in a guaranteed atomic operation. So Alice and Bob will get exactly what they expect. The 0x Protocol is designed to minimise the amount of transactions (and fees) required for Alice and Bob to successfully exchange tokens. 0x protocol uses a process involving off-chain orders with on-chain settlement. You can find more information on the 0x Protocol by reading our [whitepaper](https://0xproject.com/pdfs/0x_white_paper.pdf) or the [beginners guide to 0x](https://blog.0xproject.com/a-beginners-guide-to-0x-81d30298a5e0).

### Interacting with the Ethereum Blockchain
As web browsers do not natively support interacting with the Ethereum Blockchain, a browser extension called [Metamask](https://metamask.io/) is one way to fill that gap. Metamask will handle the users private keys and allow them to submit transactions to the network via an Ethereum node. In order to interact with dApps (distributed applications) a user must have Ether in a wallet they control. The user cannot interact with dApps from an exchange such as Coinbase. For mobile users there are applications like [Toshi](https://www.toshi.org/) and [Cipher Browser](https://www.cipherbrowser.com/) and more are popping up each month. 

### Development

<a href="https://codesandbox.io/s/1qmjyp7p5j" rel="Sandbox"><img width="800" src="https://user-images.githubusercontent.com/27389/36460173-1979adf8-166c-11e8-8bd8-8a09604abe1c.png" /></a>


#### Test Networks
Development and testing are aided by running in your own local test Ethereum node. This allows you to quickly deploy and interact with contracts without spending real ETH. [Ganache](https://github.com/trufflesuite/ganache) is a great tool for local development setup. This software works almost identically to a real Ethereum node, so we recommend you also test your application on a live testnet (i.e Kovan, Rinkeby, or Ropsten) before deploying on to the main Ethereum network.

Rather than running your own Ethereum node to connect to a network, many developers use [INFURA](https://infura.io/). They provide public, scalable Ethereum nodes to connect to the main Ethereum network as well as serval test networks.

It is difficult to get a sufficient amount of Ether to deploy your contracts and begin testing on test networks. Because of this, we have created a faucet which can dispense small amounts of Ether to your account. Navigate over to the [0x Portal](https://0xproject.com/portal/balances) with Metamask installed and connected to the Kovan Network click "Request" to receive some free test Ether!

<a href="https://0xproject.com/portal/balances"><img width="1176" alt="kovan-testnet" src="https://user-images.githubusercontent.com/27389/36271820-9bfde1c2-1234-11e8-9117-c7c4d5656d59.png"></a>

### Wrapped ETH
By now you are likely familiar with the ERC20 standard, Ether (aka ETH) and the plethora of ERC20 tokens out there. Unfortunately, ETH is not an ERC20 token and because of this interacting with both ERC20 tokens and ETH at the same time can lead to scenarios which add complexity to your smart contracts. Less code is better, it is easier to understand and audit. One technique the community has adopted is to adapt the interface of ETH to behave like an ERC20 token. This allows one to treat everything as an ERC20 token and reduces smart contract complexity. We call this WETH or Wrapper ETH. The owner of some Ether would deposit, for example, 1 Ether into the wrapped Ether contract in return for 1 WETH token. When they are finished with the WETH they withdraw back their 1ETH in exchange for 1WETH. This idea is becoming increasingly common in the ecosystem. More information on WETH can be found [here](https://weth.io/).

Now that you have a high-level overview of Ethereum Blockchain development, it is time to check out our other tutorials and examples. Be sure to try out our [Sandbox](https://codesandbox.io/s/1qmjyp7p5j) which provides a development environment ready to run against 
