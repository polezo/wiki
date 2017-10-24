In this tutorial, I will show you how you can use the [`0x.js`](https://github.com/0xProject/0x.js) library to create, validate and fill buy and sell orders via the 0x protocol. You can find all the `0x.js` documentation [here](https://0xproject.com/docs/). Before diving into the code, let's make sure you have your environment ready. 

### Setup
---
First of all, we will install TestRPC, which allows us to simulate a network of Ethereum nodes locally. Currently, `0x.js` was built while using testrpc 4.0.1 and some breaking changes were introduced with 4.1 with respect to gas estimations. Hence, for simplicity, let's install testRPC 4.0.1 as follow ; 

`npm install ethereumjs-testrpc@4.0.1 -g`

The rest of the procedure on how to setup testRPC properly for interacting with 0x can be found [here](https://0xproject.com/wiki#Testrpc-Setup-Guide). Make sure you are using the right snapshot! Now you can install `0x.js` with the following command ;  `npm install 0x.js --save`. In addition let's install the web3 bigNumber packages with `npm install web3 --save` and `npm install bignumber.js --save`. That's it! You should now be ready to start coding.

### Importing packages
---
The first step to interact with `0x.js` is to import the relevant packages as follow 
```javascript
const Web3      = require('web3');
const ZeroEx    = require('0x.js').ZeroEx;
const BigNumber = require('bignumber.js');
```

**Web3** is the package allowing us to interact with our node and the Ethereum world. **ZeroEx** is the `0x.js` library, which allows us to interact with the 0x smart contracts and environment. **BigNumber** is a JavaScript library for arbitrary-precision decimal and non-decimal arithmetic. 

### Provider and constructor
---
Now we need to instantiate a zeroEx and HttpProvider. In our case, since we are using our local node, we will use we will use **http://localhost:8545**. You can read about what providers are [here](https://0xproject.com/wiki#Web3-Provider-Explained).

```javascript
// Default provider for TestRPC
const provider = new Web3.providers.HttpProvider('http://localhost:8545')

// Calling constructor
const zeroEx = new ZeroEx(provider);
```
### Declaring decimals and addresses
---
Since we are dealing with a few contracts, we will specify them now to reduce the syntax load. Fortunately for us, `0x.js` library comes with a bunch of contract addresses that can be useful to have at hand. It also comes with a way to interact with the 0x token registry, which act as a store of vetted tokens and their addresses. One thing that is important to remember is that there are no decimals in the Ethereum virtual machine (EVM), which means you always need to keep track of how many "decimals" each token possesses. Since we will sell some ZRX for some ETH and since they both have 18 decimals, we can use a shared constant.

```javascript
//Number of decimals to use (for ETH and ZRX)
const DECIMALS = 18;

// Addresses
const NULL_ADDRESS = ZeroEx.NULL_ADDRESS;                              
const WETH_ADDRESS = await zeroEx.etherToken.getContractAddressAsync();  	 
const ZRX_ADDRESS  = await zeroEx.exchange.getZRXTokenAddressAsync();    	 
const EXCHANGE_ADDRESS = await zeroEx.exchange.getContractAddressAsync(); 
```
Above, **`NULL_ADDRESS`** is the 0x00000... address that acts as a NULL value in Ethereum's address space. **`EXCHANGE_ADDRESS`** is the [**Exchange.sol**](https://github.com/0xProject/contracts/blob/master/contracts/Exchange.sol) contract address, which is the 0x exchange smart contract which allows the maker to exchange ZRX for WETH with the taker. **`WETH_ADDRESS`** and **`ZRX_ADDRESS`** are the addresses of the tokens we will use in this tutorial. Learn more about what the [ERC20 standard](https://theethereum.wiki/w/index.php/ERC20_Token_Standard) and [WETH](https://weth.io/) are. 

### Setting up accounts
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

Note that by default only the first address (**Maker**) possesses ZRX tokens. Let's now specify our **Maker** and **Taker** accounts, where the **Maker** is the one creating (or making) the order and the **Taker** is the one filling (or taking) the order. 

```javascript
const [makerAddress, takerAddress] = accounts;
```

Another step we need to do is to allow the exchange.sol smart contract to interact with our funds. Indeed, 0x allows users to trade their tokens directly from their accounts without the need to send their tokens anywhere. This is possible thanks to the `approve()` and `transferFrom()` functions that are part of the ERC20 token standard. In order to give the exchange.sol contract access to funds, we need to set *allowances* (you can read about allowances [here](https://tokenallowance.io/)). 

``` javascript
// Unlimited allowances to 0x contract for maker and taker
const txHashAllowMaker = await zeroEx.token.setUnlimitedProxyAllowanceAsync(ZRX_ADDRESS,  makerAddress); 
await zeroEx.awaitTransactionMinedAsync(txHashAllowMaker)

const txHashAllowTaker = await zeroEx.token.setUnlimitedProxyAllowanceAsync(WETH_ADDRESS, takerAddress);
await zeroEx.awaitTransactionMinedAsync(txHashAllowTaker)
```

Now the exchange.sol smart contract will be able to allow ZRX/WETH trades between our **Maker** and **Taker**. Another thing we need to do is "convert" ETH to WETH, since ETH is unfortunately not ERC20 compliant. Concretely, "converting" ETH to WETH means that we will deposit some ETH in a smart contract acting as a ERC20 wrapper. In exchange of depositing ETH, we will get some ERC20 compliant tokens called WETH at a 1:1 conversion rate. For example, depositing 10 ETH will give us back 10 WETH and we can revert the process at any time.

```javascript
const ethToConvert = ZeroEx.toBaseUnitAmount(new BigNumber(1), DECIMALS); // Number of ETH to convert to WETH

const txHashWETH = await zeroEx.etherToken.depositAsync(ethToConvert, takerAddress);
await zeroEx.awaitTransactionMinedAsync(txHashWETH)
```

At this point, it might be worth mentioning why we need to await all those transactions. Calling an 0x.js function returns immidiately after submitting a transaction with a transaction hash, so the user interfact (UI) might show some useful information to the user before the transaction is mined (it sometimes takes long time). In our use-case we just want it to be confirmed, which happens immidiately on testrpc. It is nevertheless a good habbit to interact with the blockchain with these async/await calls. 

### Creating an order
---
Users that create an order are called **Makers** and they need to specify some information in their order so the exchange.sol smart contract knows what to do with them.

```javascript
// Generate order
const order = { 
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
 - **exchangeContractAddress** : The exchange.sol address.
 - **salt**: Random number to make the order (and therefore its hash) unique.
 - **makerFee**: How many ZRX the **Maker** will pay as a fee to the **Relayer**.
 - **takerFee** : How many ZRX the **Taker** will pay as a fee to the **Relayer**.
 - **makerTokenAmount**: The amount of token the **Maker** is offering.
 - **takerTokenAmount**: The amount of token the **Maker** is requesting from the **Taker**.
 - **expirationUnixTimestampSec**: When will the order expire (in unix time). 

The `NULL_ADDRESS` is used for the `taker` field since in our case we do not case who the taker will be and using `NULLADDRESS` will allow anyone to fill our order.

### Signing the order
---
Now that we created an order as a **Maker**, we need to prove that we actually own the address specified as `makerAddress`. After all, we could always try pretending to be someone else just to annoy an exchange and other traders! To do so, we will sign the orders with the corresponding private key and append the signature to our order.

You can first obtain the order hash with the following command ; 

``` javascript
const orderHash = ZeroEx.getOrderHashHex(order);
```

Now that we have the order hash, we can sign it and append the signature to the order; 

```javascript
// Signing orderHash -> ecSignature
const ecSignature = await zeroEx.signOrderHashAsync(orderHash, makerAddress);

// Append signature to order
const signedOrder = { 
			...order, 
			ecSignature, 
		     };
```
With this, anyone can verify if the signature is authentic and this will prevent any change to the order by a third party. If the order is changed by even a single bit, then the hash of the order will be different and therefore invalid when compared to the signed hash. 

Now let's actually verify whether the order we created is valid 

```javascript
await zeroEx.exchange.validateOrderFillableOrThrowAsync(signedOrder);
```
If something was wrong with our order, this function would throw an informative error. If it passes, then the order is fillable. 

### Filling the order
---
Finally, now that we have a valid order, we can try to fill it while acting as a **Taker**. To do so, we first specify a few arguments  

```javascript
const shouldThrowOnInsufficientBalanceOrAllowance = true;
const fillTakerTokenAmount = ZeroEx.toBaseUnitAmount(new BigNumber(0.1), DECIMALS);
```
When set to `false`, `shouldThrowOnInsufficientBalanceOrAllowance` will cause the smart contract to verify if the balances or allowances are sufficient and if that's not the case, it will log an error and return the remaining gas to the sender. If set to `true` the fill call will skip this verification step and will throw subsequently (consuming all the gas) if balances or allowances are not sufficient. The former cost slightly more gas for valid orders since it involves an extra verification step, but will save the remaining gas in cases where the fill fails. `fillTakerTokenAmount` is simply the amount of token (in our case WETH) the **Taker** wants to fill. 

Now let's try to fill the order : 
```javascript
const txHash = await zeroEx.exchange.fillOrderAsync(signedOrder, fillTakerTokenAmount, shouldThrowOnInsufficientBalanceOrAllowance, takerAddress);
```

This function broadcasts the transaction and returns its hash that we can use to get information about the transaction itself, such as when it is successfully mined. 

```javascript
const txReceipt = await zeroEx.awaitTransactionMinedAsync(txHash)
console.log(txReceipt.logs);
```
giving us the following successful transaction ;
``` javascript
{ transactionHash: '0xa70f47bd228e62cf4703ef6b2d4904b401ac9c0e3c4489690114cddad3a72cb0',
  transactionIndex: 0,
  blockHash: '0x0b8b07167275bc6e510ab13aca4a8ffc21244299c5b221afa7cd15e9b314d094',
  blockNumber: 140,
  gasUsed: 96843,
  cumulativeGasUsed: 96843,
  contractAddress: null,
  logs: 
   [ { logIndex: 0,
    transactionIndex: 0,
    transactionHash: '0x525572e2b13978f54cbcbdb3ffb82ec4a54a59bd47f3e6c6872eed43a80173e9',
    blockHash: '0x38a6b38a653014bec6b4ced39de624effb67523c1a2ed790aa125dd0644ffb5d',
    blockNumber: 152,
    address: '0x34d402f14d58e001d8efbe6585051bf9706aa064',
    data: '0x00000000000000000000000000000000000000000000000000ecd8fae906aaaa',
    topics: 
     [ '0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef',
       '0x0000000000000000000000005409ed021d9299bf6814279a6a1411a7e866a631',
       '0x0000000000000000000000006ecbe1db9ef729cbe972c83fb886247691fb6beb' ],
    type: 'mined',
    event: 'Transfer',
    args: 
     { _from: '0x5409ed021d9299bf6814279a6a1411a7e866a631',
       _to: '0x6ecbe1db9ef729cbe972c83fb886247691fb6beb',
       _value: [Object] } },
  { logIndex: 1,
    transactionIndex: 0,
    transactionHash: '0x525572e2b13978f54cbcbdb3ffb82ec4a54a59bd47f3e6c6872eed43a80173e9',
    blockHash: '0x38a6b38a653014bec6b4ced39de624effb67523c1a2ed790aa125dd0644ffb5d',
    blockNumber: 152,
    address: '0x48bacb9266a570d521063ef5dd96e61686dbe788',
    data: '0x000000000000000000000000000000000000000000000000016345785d8a0000',
    topics: 
     [ '0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef',
       '0x0000000000000000000000006ecbe1db9ef729cbe972c83fb886247691fb6beb',
       '0x0000000000000000000000005409ed021d9299bf6814279a6a1411a7e866a631' ],
    type: 'mined',
    event: 'Transfer',
    args: 
     { _from: '0x6ecbe1db9ef729cbe972c83fb886247691fb6beb',
       _to: '0x5409ed021d9299bf6814279a6a1411a7e866a631',
       _value: [Object] } },
  { logIndex: 2,
    transactionIndex: 0,
    transactionHash: '0x525572e2b13978f54cbcbdb3ffb82ec4a54a59bd47f3e6c6872eed43a80173e9',
    blockHash: '0x38a6b38a653014bec6b4ced39de624effb67523c1a2ed790aa125dd0644ffb5d',
    blockNumber: 152,
    address: '0xb69e673309512a9d726f87304c6984054f87a93b',
    data: '0x0000000000000000000000006ecbe1db9ef729cbe972c83fb886247691fb6beb00000000000000000000000034d402f14d58e001d8efbe6585051bf9706aa06400000000000000000000000048bacb9266a570d521063ef5dd96e61686dbe78800000000000000000000000000000000000000000000000000ecd8fae906aaaa000000000000000000000000000000000000000000000000016345785d8a0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003dd2a4560a7ad2ed8525438c3bbe644c3a6f6ffa7bbec8307acd06ed14a39992',
    topics: 
     [ '0x0d0b9391970d9a25552f37d436d2aae2925e2bfe1b2a923754bada030c498cb3',
       '0x0000000000000000000000005409ed021d9299bf6814279a6a1411a7e866a631',
       '0x0000000000000000000000000000000000000000000000000000000000000000',
       '0xd4612851fa69e7fab483b481467f7ab6ef0501ca7fe73230cafdbf937b04e9ff' ],
    type: 'mined',
    event: 'LogFill',
    args: 
     { maker: '0x5409ed021d9299bf6814279a6a1411a7e866a631',
       taker: '0x6ecbe1db9ef729cbe972c83fb886247691fb6beb',
       feeRecipient: '0x0000000000000000000000000000000000000000',
       makerToken: '0x34d402f14d58e001d8efbe6585051bf9706aa064',
       takerToken: '0x48bacb9266a570d521063ef5dd96e61686dbe788',
       filledMakerTokenAmount: [Object],
       filledTakerTokenAmount: [Object],
       paidMakerFee: [Object],
       paidTakerFee: [Object],
       tokens: '0xd4612851fa69e7fab483b481467f7ab6ef0501ca7fe73230cafdbf937b04e9ff',
       orderHash: '0x3dd2a4560a7ad2ed8525438c3bbe644c3a6f6ffa7bbec8307acd06ed14a39992' }]
}
```
Congratulation! You now successfully created, validated and filled an order using 0x.js and the 0x protocol!

### Conclusion
---
This was a simple tutorial on what you can do with `0x.js` and the 0x protocol, but there is obviously more. In the future, we will make some tutorials on how to maintain a orderbook efficiently, since many orders become invalid over time, be it because they were filled, expired or certain conditions changed (balance, allowance, etc.), how to use various relayer strategies and more. The goal of 0x is to provide tools easy to use so that anyone can set-up their own relayer. As a matter of fact, this was my first javascript script ever and the ride was smoother than I expected! 

You can find the the full tutorial script [here](https://gist.github.com/PhABC/05dc653fa483f5bb3421ef0cb1949845). 

*Note that I am using nodejs v8.7.0 and npm v5.5.1*.
