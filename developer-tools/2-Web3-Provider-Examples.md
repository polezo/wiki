As described in the [Web3 Provider Explained](#Web3-Provider-Explained) section of the wiki, we at 0x have created a number of useful Web3 subproviders. These subproviders aren't only useful for 0x.js, they can be added to any application to provide resiliency, usability and to support hardware wallet functionality.

You can install the 0x subproviders package as follows:

```
npm install @0x/subproviders --save
```

The subproviders work best when they are composed together using the [Web3 Provider Engine](https://github.com/MetaMask/provider-engine). This is re-exported from `@0x/subproviders` so you don't need to install any additional dependencies.

In the first example, we will make use of a browser extension wallet (e.g [Metamask](https://metamask.io/)) composed with an Ethereum node we control. This set up allows all of the account based activity (signing of messages and sending transactions) to route to the browser extension wallet, while allowing the data fetching requests to flow through to a specific Ethereum node of our choosing.

```typescript
import { SignerSubprovider, RPCSubprovider, Web3ProviderEngine } from '@0x/subproviders';
import { Web3Wrapper } from '@0x/web3-wrapper';

// Create a Web3 Provider Engine
const providerEngine = new Web3ProviderEngine();
// Compose our Providers, order matters
// Use the SignerSubprovider to wrap the browser extension wallet
// All account based and signing requests will go through the SignerSubprovider
providerEngine.addProvider(new SignerSubprovider(window.web3.currentProvider));
// Use an RPC provider to route all other requests
providerEngine.addProvider(new RPCSubprovider('http://localhost:8545'));
providerEngine.start();

(async () => {
    // Get all of the accounts through the Web3Wrapper
    const web3Wrapper = new Web3Wrapper(providerEngine);
    const accounts = await web3Wrapper.getAvailableAddressesAsync();
    console.log(accounts);
})();
```

Using the configuration above, all account related requests (e.g signing and sending a transaction) go through the browser extension wallet. All other data fetching requests go through the RPC Subprovider. This example is great as an application can use their own synced Ethereum node, rather than relying on Metamask or Infura uptime.

Within the 0x Subprovider package, we have also added a [Ledger Nano S](https://www.ledgerwallet.com/start/ledger-nano-s) subprovider. By adding this subprovider first into the Web3 Provider Engine, we are able to route all account based requests to the Ledger. We again use the RPC Provider for all other requests.

```typescript
import {
    ledgerEthereumBrowserClientFactoryAsync as ledgerEthereumClientFactoryAsync,
    LedgerSubprovider,
    RPCSubprovider,
    Web3ProviderEngine,
} from '@0x/subproviders';
import { Web3Wrapper } from '@0x/web3-wrapper';

const KOVAN_NETWORK_ID = 42;
// Create a Web3 Provider Engine
const providerEngine = new Web3ProviderEngine();
// Compose our Providers, order matters
// Use the Ledger Subprovider to intercept all account based requests
// All other requests will go through the RPCSubprovider
const ledgerSubprovider = new LedgerSubprovider({
    networkId: KOVAN_NETWORK_ID,
    ledgerEthereumClientFactoryAsync,
});
providerEngine.addProvider(ledgerSubprovider);
// Use an RPC provider to route all other requests
providerEngine.addProvider(new RPCSubprovider('http://localhost:8545'));
providerEngine.start();

(async () => {
    // Get all of the accounts through the Web3Wrapper
    const web3Wrapper = new Web3Wrapper(providerEngine);
    const accounts = await web3Wrapper.getAvailableAddressesAsync();
    console.log(accounts);
})();
```

This above example works for enabling the Ledger Subprovider in Browser based applications, if you want to use the Ledger directly in a Node.js application, you must use a different `ledgerEthereumClientFactoryAsync`, for example:

```typescript
// Import the NodeJS Client Factory, rather than the Browser Client Factory
import {
    ledgerEthereumNodeJsClientFactoryAsync as ledgerEthereumClientFactoryAsync,
    LedgerSubprovider,
} from '@0x/subproviders';
```

The Ledger Subprovider has a number of public methods and can be used directly to set various options. For example, the derivation path and the account index can be changed.

```typescript
public getPath(): string
public setPath(derivationPath: string)
public setPathIndex(pathIndex: number)
public async getAccountsAsync(): Promise<string[]>
public async signTransactionAsync(txParams: PartialTxParams): Promise<string>
public async signPersonalMessageAsync(data: string): Promise<string>
```

### Notes on Ledger Subprovider

It is important to remember that UI components and UX need to be considered when adding the hardware wallet support to your application. A few examples that require additional thought:

-   The user may have multiple accounts on the hardware wallet. The first account may not be the desired one
-   The user may want to set the a higher gas price so the transaction has a higher probability of being mined
-   The hardware device is limited to handling one request at a time
-   The hardware device is not capable of showing the message entirely on screen. An application [should confirm](https://github.com/ethfinex/0x-order-verify) what is displayed on the device

In our last example we will add redundancy to the application by making use of the RedundantRPCSubprovider. The RedundantRPCSubprovider helps your application stay up when underlying Ethereum nodes experience network issues. To use this subprovider, simply provide it with a list of Ethereum node RPC endpoints and it will attempt each one in sequence until a successful response is returned.

```typescript
import { RedundantSubprovider, RPCSubprovider, SignerSubprovider, Web3ProviderEngine } from '@0x/subproviders';
import { Web3Wrapper } from '@0x/web3-wrapper';

// Create a Web3 Provider Engine
const providerEngine = new Web3ProviderEngine();
// Compose our Providers, order matters
// Use the SignerSubprovider to wrap the browser extension wallet
const signerSubprovider = new SignerSubprovider(window.web3.currentProvider);
providerEngine.addProvider(signerSubprovider);
// Use the RedundantRPCSubprovider to route all other requests
const subproviders = [new RPCSubprovider('http://localhost:8545'), new RPCSubprovider('https://kovan.infura.io')];
providerEngine.addProvider(new RedundantSubprovider(subproviders));
providerEngine.start();

(async () => {
    // Get the latest Block Number
    const web3Wrapper = new Web3Wrapper(providerEngine);
    const blockNumber = await web3Wrapper.getBlockNumberAsync();
    console.log(blockNumber);
})();
```
