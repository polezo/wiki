In this tutorial, we will show you how you can use the [0x.js](https://github.com/0xProject/0x.js) library to create, validate and fill orders via the 0x protocol. You can find all the `0x.js` documentation [here](https://0xproject.com/docs/0x.js).

### Setup

Since 0x.js helps you build apps that interface with Ethereum, you will need to use it in conjunction with an Ethereum node. For development, we recommend using [Ganache CLI](https://github.com/trufflesuite/ganache-cli). Since there is some setup required to getting started with Ganache and the 0x smart contracts, we are going to use a [0x starter project](https://github.com/0xProject/0x-starter-project) which handles a lot of this for us.

Clone the repo:

```
git clone https://github.com/0xProject/0x-starter-project.git
```

Install all the dependencies (we use Yarn: `brew install yarn --without-node`):

```
yarn
```

Pull the latest Ganache 0x snapshot with all the 0x contracts pre-deployed:

```
yarn download_snapshot
```

In a separate terminal, navigate to the project directory and start Ganache:

```
yarn ganache-cli
```

You can now run the code we are going to walk you through in the rest of this tutorial:

```
yarn scenario:fill_order_erc20
```

### Importing packages

The first step to interact with `0x.js` is to import the following relevant packages:

```typescript
import {
    assetDataUtils,
    BigNumber,
    ContractWrappers,
    generatePseudoRandomSalt,
    Order,
    orderHashUtils,
    signatureUtils,
    SignerType,
} from '0x.js';
import { Web3Wrapper } from '@0xproject/web3-wrapper';
```

**0x.js** is a package that pulls in a number of underlying 0x packages and exposes their respective functionality. You can choose to pull these packages directly without using 0x.js. These packages allow you to interact with the 0x smart contracts (contract wrappers) and create, sign and validate orders (order utils).
**BigNumber** is a JavaScript library for arbitrary-precision decimal and non-decimal arithmetic.
**Web3Wrapper** is a package for interacting with an Ethereum node, retrieving account information. This is an optional package and you can choose to use alternatives like Web3.js or Ethers.js.

### Provider and constructor

Now we need to instantiate an instance of ContractWrappers and a Provider. In our case, since we are using our local node (ganache), we will use **http://localhost:8545**. You can read about what providers are [here](#Web3-Provider-Explained).

```typescript
import { RPCSubprovider, Web3ProviderEngine } from '0x.js';

export const providerEngine = new Web3ProviderEngine();
providerEngine.addProvider(new RPCSubprovider('http://localhost:8545'));
providerEngine.start();
// Instantiate ContractWrappers with the provider
const contractWrappers = new ContractWrappers(providerEngine, { networkId: NETWORK_CONFIGS.networkId });
```

### Retreiving Accounts from the Provider

We will use Web3Wrapper to retrieve accounts from the provider. The first account will the the maker, and the second account will be the taker for the purposes of this tutorial.

```typescript
const web3Wrapper = new Web3Wrapper(providerEngine);
const [maker, taker] = await web3Wrapper.getAvailableAddressesAsync();
```

### Declaring decimals and addresses

Since we are dealing with a few contracts, we will specify them now to reduce the syntax load. Fortunately for us, `0x.js` library comes with a couple of contract addresses that can be useful to have at hand. One thing that is important to remember is that there are no decimals in the Ethereum virtual machine (EVM), which means you always need to keep track of how many "decimals" each token possesses. Since we will sell some ZRX for some ETH and since they both have 18 decimals, we can use a shared constant.

```typescript
// Token Addresses
const zrxTokenAddress = contractWrappers.exchange.getZRXTokenAddress();
const etherTokenAddress = contractWrappers.etherToken.getContractAddressIfExists();
```

0x Protocol uses the ABI encoding scheme for asset data. For example, the ERC20 Token address which is being traded on 0x needs to be encoded. Encoding the address informs the 0x smart contracts on which type of asset is being traded (e.g ERC20 or ERC721) and has optional parameters for different token types (e.g ERC721 token id). In this tutorial we are trading 5 ZRX (ERC20) for 0.1 WETH (ERC20).

```typescript
const makerAssetData = assetDataUtils.encodeERC20AssetData(zrxTokenAddress);
const takerAssetData = assetDataUtils.encodeERC20AssetData(etherTokenAddress);
// the amount the maker is selling of maker asset
const makerAssetAmount = Web3Wrapper.toBaseUnitAmount(new BigNumber(5), DECIMALS);
// the amount the maker wants of taker asset
const takerAssetAmount = Web3Wrapper.toBaseUnitAmount(new BigNumber(0.1), DECIMALS);
```

### Approvals and WETH Balance

To trade on 0x, the participants (maker and taker) require a small amount of initial set up. They need to approve the 0x smart contracts to move funds on their behalf. In order to give 0x protocol smart contract access to funds, we need to set _allowances_ (you can read about allowances [here](https://tokenallowance.io/)). In this tutorial the taker asset is WETH (or Wrapper ETH, you can read about WETH [here](https://weth.io/).), as ETH is not an ERC20 token it must first be converted into WETH to be used on 0x. Concretely, "converting" ETH to WETH means that we will deposit some ETH in a smart contract acting as a ERC20 wrapper. In exchange of depositing ETH, we will get some ERC20 compliant tokens called WETH at a 1:1 conversion rate. For example, depositing 10 ETH will give us back 10 WETH and we can revert the process at any time. The ContractWrappers package has helpers for general ERC20 tokens as well as the WETH token.

```typescript
// Allow the 0x ERC20 Proxy to move ZRX on behalf of makerAccount
const makerZRXApprovalTxHash = await contractWrappers.erc20Token.setUnlimitedProxyAllowanceAsync(
    zrxTokenAddress,
    maker,
);
await web3Wrapper.awaitTransactionMinedAsync(makerZRXApprovalTxHash);

// Allow the 0x ERC20 Proxy to move WETH on behalf of takerAccount
const takerWETHApprovalTxHash = await contractWrappers.erc20Token.setUnlimitedProxyAllowanceAsync(
    etherTokenAddress,
    taker,
);
await web3Wrapper.awaitTransactionMinedAsync(takerWETHApprovalTxHash);

// Convert ETH into WETH for taker by depositing ETH into the WETH contract
const takerWETHDepositTxHash = await contractWrappers.etherToken.depositAsync(
    etherTokenAddress,
    takerAssetAmount,
    taker,
);
await web3Wrapper.awaitTransactionMinedAsync(takerWETHDepositTxHash);
```

At this point, it is worth mentioning why we need to await all those transactions. Calling an 0x.js function returns immediately after submitting a transaction with a transaction hash, so the user interface (UI) might show some useful information to the user before the transaction is mined (it sometimes takes long time). In our use-case we just want it to be confirmed, which happens immediately on ganache. It is nevertheless a good habit to interact with the blockchain with these async/await calls.

### Creating an order

Users that create an order are called **Makers** and they need to specify some information in their order so the 0x Exchange smart contract knows what to do with them.

```typescript
// Set up the Order and fill it
const randomExpiration = getRandomFutureDateInSeconds();
const exchangeAddress = contractWrappers.exchange.getContractAddress();

// Create the order
const order: Order = {
    exchangeAddress,
    makerAddress: maker,
    takerAddress: NULL_ADDRESS,
    senderAddress: NULL_ADDRESS,
    feeRecipientAddress: NULL_ADDRESS,
    expirationTimeSeconds: randomExpiration,
    salt: generatePseudoRandomSalt(),
    makerAssetAmount,
    takerAssetAmount,
    makerAssetData,
    takerAssetData,
    makerFee: ZERO,
    takerFee: ZERO,
};
```

where the fields are:

-   **exchangeAddress** : The Exchange address.
-   **makerAddress** : Ethereum address of our **Maker**.
-   **takerAddress** : Ethereum address of our **Taker**.
-   **senderAddress** : Ethereum address of a required **Sender** (none for now).
-   **feeRecipientAddress** : Ethereum address of our **Relayer** (none for now).
-   **expirationTimeSeconds**: When will the order expire (in unix time).
-   **salt**: Random number to make the order (and therefore its hash) unique.
-   **makerAssetAmount**: The amount of token the **Maker** is offering.
-   **takerAssetAmount**: The amount of token the **Maker** is requesting from the **Taker**.
-   **makerAssetData**: The token address the **Maker** is offering.
-   **takerAssetData**: The token address the **Maker** is requesting from the **Taker**.
-   **makerFee**: How many ZRX the **Maker** will pay as a fee to the **Relayer**.
-   **takerFee** : How many ZRX the **Taker** will pay as a fee to the **Relayer**.

The `NULL_ADDRESS` is used for the `taker` field since in our case we do not care who the taker will be and using `NULL_ADDRESS` will allow anyone to fill our order.

### Signing the order

Now that we created an order as a **Maker**, we need to prove that we actually own the address specified as `makerAddress`. After all, we could always try pretending to be someone else just to annoy an exchange and other traders! To do so, we will sign the orders with the corresponding private key and append the signature to our order.

You first obtain the order hash via the following:

```typescript
// Generate the order hash and sign it
const orderHashHex = orderHashUtils.getOrderHashHex(order);
```

Now that we have the order hash, we can sign it and append the signature to the order;

```typescript
const signature = await signatureUtils.ecSignOrderHashAsync(providerEngine, orderHashHex, maker, SignerType.Default);
const signedOrder = { ...order, signature };
```

With this, anyone can verify that the signature is authentic and this will prevent any change to the order by a third party. If the order is changed by even a single bit, then the hash of the order will be different and therefore invalid when compared to the signed hash.

Now let's verify whether the order is valid

```typescript
await contractWrappers.exchange.validateFillOrderThrowIfInvalidAsync(signedOrder, takerAssetAmount, taker);
```

If something was wrong with our order, this function would throw an informative error. If it passes, then the order is currently fillable. A relayer should constantly be [pruning their orderbook](#0x-OrderWatcher) of invalid orders using this method.

### Filling the order

Finally, now that we have a valid order, we can try to fill it while acting as a **Taker**. `takerAssetAmount` is simply the amount of tokens (in our case WETH) the **Taker** wants to fill. For this tutorial, we will be completely filling the order. Orders may also be partially filled.

Now let's fill the order:

```typescript
txHash = await contractWrappers.exchange.fillOrderAsync(signedOrder, takerAssetAmount, taker, {
    gasLimit: TX_DEFAULTS.gas,
});
await web3Wrapper.awaitTransactionMinedAsync(txHash);
```

This function broadcasts the transaction and returns its hash that we can use to get information about the transaction itself, such as when it is successfully mined.

Congratulations! You have now successfully created, validated and filled an order using 0x.js and the 0x protocol!
