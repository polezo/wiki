# **0x Tutorial 101 :** Create, Verify and Fill an Order Using 0x.js

In this tutorial, I will show you how you can use the `0x.js` library to create, validate and fill buy and sell orders via the 0x protocol. You can find all the `0x.js` documentation [here](https://0xproject.com/docs/). Before diving into the code, let's make sure you have your environment ready. 

## Setup
---
First of all, we will install TestRPC, which allows us to simulate a network of Ethereum nodes locally. Currently, `0x.js` was built while using testrpc 4.0.1 and some breaking changes were introduced with 4.1 with respect to gas estimations. Hence, for simplicity, let's install testRPC 4.0.1 as follow ; 

`npm install ethereumjs-testrpc@4.0.1 -g`

The rest of the procedure on how to setup testRPC properly for interacting with 0x can be found [here](https://0xproject.com/wiki#Testrpc-Setup-Guide). Make sure you are using the right snapshot! Now you can install `0x.js` with the following command ;  `npm install 0x.js --save`. In addition let's install the web3 bigNumber packages with `npm install web3` and `npm install bignumber`. That's it! You should now be ready to start coding.

## Importing Packages
---
The first step to interact with `0x.js` is to import the relevant packages as follow 
```javascript
var Web3      = require('web3');
var ZeroEx    = require('0x.js').ZeroEx;
var BigNumber = require('bignumber.js');
```

**Web3** is the package allowing us to interact with our node and the Ethereum world. **ZeroEx** is the `0x.js` library, which allows us to interact with the 0x smart contracts and environment. **BigNumber** is a JavaScript library for arbitrary-precision decimal and non-decimal arithmetic. 

## Provider and Constructor
---
Now we need to instantiate a zeroEx and HttpProvider instance while specifying our provider. In our case, since we are using our local node, we will use we will use **http://localhost:8545**. You can read about what providers are [here](https://0xproject.com/wiki#Web3-Provider-Explained).

```javascript
// Default provider for TestRPC
const provider = new Web3.providers.HttpProvider('http://localhost:8545')

// Calling constructor
var zeroEx = new ZeroEx(provider);
```
## Declaring Decimals and Addresses
---
Since we are dealing with a few contracts, we will specify them now to reduce the syntax load. Fortunately for us, `0x.js` library comes with a bunch of contract addresses that can be useful to have at hand. It also comes with a way to interact with the 0x token registry, which act as a storage for vetted tokens and their address. One thing that is important to remember is that there are no decimals in the Ethereum Virtual Machine, which means you always need to keep track of how many "decimals" each token possess. Since we will sell some ZRX for some ETH and since they both have 18 decimals, we can use a shared constant.

```javascript
//Number of decimals to use (for ETH and ZRX)
const DECIMALS = 18;

// Addresses
const NULL_ADDRESS = ZeroEx.NULL_ADDRESS;                              
const WETH_ADDRESS = await zeroEx.etherToken.getContractAddressAsync();  	 
const ZRX_ADDRESS  = await zeroEx.exchange.getZRXTokenAddressAsync();    	 
const EXCHANGE_ADDRESS = await zeroEx.exchange.getContractAddressAsync(); 
```
Above, **`NULL_ADDRESS`** is the 0x00000... address that is used by default when creating a vector. **`EXCHANGE_ADDRESS`** is the **exchange.sol** contract address, which is the 0x exchange smart contract allowing us to perform atomic swaps between ERC20 compliant tokens. **`WETH_ADDRESS`** and **`ZRX_ADDRESS`** are the address of the tokens we will use in this tutorial. Learn more about what the [ERC20 standard](https://theethereum.wiki/w/index.php/ERC20_Token_Standard) is and what [WETHs](https://weth.io/) are. 

## Setting up Accounts
---
TestRPC comes with a few test accounts and tokens, which is quite convenient when prototyping. We can get the list of our account addresses as follow ; 
``` javascript 
const accounts =  await zeroEx.getAvailableAddressesAsync();
console.log(accounts)
```
giving us
```javascript
[ '0x5409ed021d9299bf6814279a6a1411a7e866a631',
  '0x6ecbe1db9ef729cbe972c83fb886247691fb6beb',
  '0xe36ea790bc9d7ab70c55260c66d52b1eca985f84',
  '0xe834ec434daba538cd1b9fe1582052b880bd7e63',
  '0x78dc5d2d739606d31509c31d654056a45185ecb6',
  '0xa8dda8d7f5310e4a9e24f8eba77e091ac264f872',
  '0x06cef8e666768cc40cc78cf93d9611019ddcb628',
  '0x4404ac8bd8f9618d27ad2f1485aa1b2cfd82482d',
  '0x7457d5e02197480db681d3fdf256c7aca21bdc12',
  '0x91c987bf62d25945db517bdaa840a6c661374402' ]
 
```

Let's now specify our **Maker** and **Taker** accounts, where the **Maker** is the one creating (or making) the order and the **Taker** is the one filling (or taking) the order. 

```javascript
const makerAddress = accounts[0];
const takerAddress = accounts[1];
```

Another step we need to do is to allow the exchange.sol smart contract to interact with our funds. Indeed, 0x allows users to trade their tokens directly from their accounts without the need to send their tokens anywhere. This is possible thanks to the `approve()` and `transferFrom()` functions that are part of the ERC20 token standard. In order to give the exchange.sol contract access to funds, we need to set *allowances* (you can read about allowances [here](https://tokenallowance.io/)). 

``` javascript
// Unlimited allowances to 0x contract for maker and taker
zeroEx.token.setUnlimitedProxyAllowanceAsync(ZRX_ADDRESS,  makerAddress);
zeroEx.token.setUnlimitedProxyAllowanceAsync(WETH_ADDRESS, takerAddress);
```

Now the exchange.sol smart contract will be able to allow ZRX/WETH trades between our **Maker** and **Taker**. Another thing we need to do is "convert" ETH to WETH, since ETH is unfortunately not ERC20 compliant. Concretely, "converting" ETH to WETH means that we will deposit some ETH in a smart contract acting as a ERC20 wrapper. In exchange of depositing ETH, we will get some ERC20 compliant tokens called WETH at a 1:1 conversion rate. For example, depositing 10 ETH will give us back 10 WETH and we can revert the process at any time.

```javascript
const ETHtoConvert = ZeroEx.toBaseUnitAmount(new BigNumber(1), DECIMALS); // Number of ETH to convert to WETH
zeroEx.etherToken.depositAsync(ETHtoConvert, takerAddress);
```

## Creating an Order
---
Users that create an order are called **Makers** and they need to specify some information in their order so the exchange.sol smart contract knows what to do with them.

```javascript
// Generate order
order = { 
	  maker: makerAddress, 
	  taker: NULL_ADDRESS,
	  feeRecipient: NULL_ADDRESS,
	  makerTokenAddress: ZRX_ADDRESS,
	  takerTokenAddress: WETH_ADDRESS,
	  exchangeContractAddress: EXCHANGE_ADDRESS,
	  salt: ZeroEx.generatePseudoRandomSalt(),
	  makerFee: new BigNumber(0),
	  takerFee: new BigNumber(0),
	  makerTokenAmount: ZeroEx.toBaseUnitAmount(new BigNumber(0.2), DECIMALS); 
	  takerTokenAmount: ZeroEx.toBaseUnitAmount(new BigNumber(0.3), DECIMALS); 
	  expirationUnixTimestampSec: new BigNumber(Date.now() + 3600000),          
	 };
```
where the fields are

 - **maker** : Ethereum address of our **Maker**.
 - **taker** : Ethereum address of our **Taker**.
 - **feeRecipient** : Ethereum address of our **Relayer** (none for now).
 - **makerTokenAddress**: The token address the **Maker** is offering. 
 - **takerTokenAddress**: The token address the **Maker** is requesting from the **Taker**.
 -  **exchangeContractAddress** : The exchange.sol address.
 - **salt**: Random number to make the order unique.
 - **makerFee**: How many ZRX the **Maker** will pay as a fee to the **Relayer**.
 - **takerFee** : How many ZRX the **Taker** will pay as a fee to the **Relayer**.
 -  **makerTokenAmount**: The amount of token the **Maker** is offering.
 - **takerTokenAmount**: The amount of token the **Maker** is requesting from the **Taker**.
 - **expirationUnixTimestampSec**: When will the order expire (in unix time). 

The `NULL_ADDRESS` is used for the `taker` field since we don't know who the taker will be and will allow anyone to fill our order.

## Signing the Order
---
Now that we created an order as a **Maker**, we need to prove that we actually own the address specified as `makerAddress`. After all, we could always try to pretend to be someone else just to annoy an exchange and other traders! To do so, we will sign the orders with the corresponding private key and append the signature to our order.

```javascript
// Signing orderHash -> ecSignature
const ecSignature = await zeroEx.signOrderHashAsync(orderHash, makerAddress);

//Appending signature to order
const signedOrder = { ...order, ecSignature, }
```

With this, anyone can verify if the signature is authentic and this will prevent any change to the order by a third party. If the order is changed even by a single bit, then the hash of the order will be different and therefore invalid when compared to the signed hash. 

You can obtain the order hash with the following command ; 
``` javascript
const orderHash = ZeroEx.getOrderHashHex(order);
```

Now let's actually verify whether the order we created is valid ; 

```javascript
await zeroEx.exchange.validateOrderFillableOrThrowAsync(signedOrder);
```
If something was wrong with our order, this function would throw an informative error. If it passes, then the order is fillable. 

## Filling the Order
---
Finally, now that we have a valid order, we can try to fill it while active as a **Taker**. To do so, we first specify a few arguments ; 

```javascript
shouldThrowOnInsufficientBalanceOrAllowance = true;
fillTakerTokenAmount = ZeroEx.toBaseUnitAmount(new BigNumber(0.1), DECIMALS);;
```
When set to true, `shouldThrowOnInsufficientBalanceOrAllowance` will lead to the smart contract to verify if the balances or allowances are sufficient and the transfer will throw subsequently (consuming all the gas). When set to false, the smart contract will log an error and return. `fillTakerTokenAmount` is simply the amount of token (in our case WETH) the **Taker** wants to fill. 

Now let's try to fill the order : 
```javascript
txHash = await zeroEx.exchange.fillOrderAsync( signedOrder, fillTakerTokenAmount, shouldThrowOnInsufficientBalanceOrAllowance, takerAddress );
```

This function broadcast the transaction and returns its hash that we can use to get information about the transaction itself, such as when it is successfully mined. 

```javascript
txReceipt = await zeroEx.awaitTransactionMinedAsync(txHash)
console.log(txReceipt);
```
giving us the following successful transaction ;
``` javascript
{ transactionHash: '0x48b80ed9a0f49e9f7fb27f60d82789f94619f53e4734c115a2acb28071a0bd9e',
  transactionIndex: 0,
  blockHash: '0x038971b66e765062d31f7c297981d4f90e2417f1812585be2a1295614de620be',
  blockNumber: 128,
  gasUsed: 96843,
  cumulativeGasUsed: 96843,
  contractAddress: null,
  logs: 
   [ { logIndex: 0,
       transactionIndex: 0,
       transactionHash: '0x48b80ed9a0f49e9f7fb27f60d82789f94619f53e4734c115a2acb28071a0bd9e',
       blockHash: '0x038971b66e765062d31f7c297981d4f90e2417f1812585be2a1295614de620be',
       blockNumber: 128,
       address: '0x34d402f14d58e001d8efbe6585051bf9706aa064',
       data: '0x00000000000000000000000000000000000000000000000000ecd8fae906aaaa',
       topics: [Array],
       type: 'mined',
       event: 'Transfer',
       args: [Object] },

     [...]

}
```
Congratulation! You now successfully created, validated and filled an order using 0x.js and the 0x protocol!

## Conclusion
---
This was a simple tutorial on what you can do with `0x.js` and the 0x protocol, but there is obviously more. In the future, we will make some tutorials on how to maintain a orderbook efficiently, since many orders become invalid over time, be it because they were filled, expired or certain conditions changed (balance, allowance, etc.), how to use various relayer strategies and more. The goal of 0x is to provide tools easy to use so that anyone can set-up their own relayer. As a matter of fact, this was my first javascript script ever and the ride was smoother than I expected! 

You can find the full tutorial script [here](https://gist.github.com/PhABC/05dc653fa483f5bb3421ef0cb1949845). 

*Note that I am using nodejs v8.7.0 and npm v5.5.1*.
