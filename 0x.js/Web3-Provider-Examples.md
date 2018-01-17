As described in the Web3 Provider Explained section on the wiki, we at 0x have created a number of useful subproviders, but they aren't only useful for 0x.js. They can be added to any dApp to provide resiliency, usability and to offer hardware support.

You can install the 0x providers package as follows:
```
npm install @0xproject/subproviders --save
```

The subproviders work best when they are composed together using the [Web3 Provider Engine](https://github.com/MetaMask/provider-engine). This can be installed as follows:

```
npm install web3 web3-provider-engine --save
```

In the first example, we will make use of a browser extension wallet (e.g [Metamask](https://metamask.io/)) composed with a Ethereum node we control. This set up allows all of the account based activity (signing of messages and transactions) to route to the browser extension wallet, while allowing the actual submission of the transaction to flow through to a different Ethereum node. 


```typescript
import * as Web3 from 'web3';
import Web3ProviderEngine = require('web3-provider-engine');
import {promisify} from '@0xproject/utils';
import {
    InjectedWeb3Subprovider,
    RedundantRPCSubprovider
} from '@0xproject/subproviders';

// Create a Web3 Provider Engine
const engine = new Web3ProviderEngine();
// Compose our Providers, order matters
// Use the InjectedWeb3Subprovider to wrap the browser extension wallet
engine.addProvider(new InjectedWeb3Subprovider((window as any).web3));
// Use an RPC provider to route all other requests
engine.addProvider(new RedundantRPCSubprovider(['http://localhost:8545']));
engine.start();

const web3 = new Web3(engine);
// Optional, use within 0x.js
const zeroEx = new ZeroEx(web3.currentProvider, { networkId: 42 });
const accounts = await promisify<string>(web3.eth.getAccounts)();
console.log(accounts);
```

In the above example, all account based requests, such as signing a transaction, will go through the browser extension wallet. All other requests, such as submitting a transaction, will go through the RPC Subprovider. This is great if Metamask or Infura are under heavy load and you can use your Ethereum node to submit transactions to the network.

Within the 0x Subprovider package, we also have added a Ledger Nano S subprovider. By adding this subprovider into the Provider Engine, we are able to route all account based requests (signing etc) to the Ledger Nano S. The subprovider can then be used either directly, or through the Web3 Provider API.

```typescript
import * as Web3 from 'web3';
import Web3ProviderEngine = require('web3-provider-engine');
import {promisify} from '@0xproject/utils';
import {
    ledgerEthereumBrowserClientFactoryAsync as ledgerEthereumClientFactoryAsync,
    LedgerSubprovider,
    RedundantRPCSubprovider
} from '@0xproject/subproviders';

// Create a Web3 Provider Engine
const engine = new Web3ProviderEngine();
// Compose our Providers, order matters
// Use the Ledger Subprovider to intercept all account based requests
const ledgerSubprovider = new LedgerSubprovider(
    42,
    ledgerEthereumClientFactoryAsync,
);
engine.addProvider(ledgerSubprovider);
// Use an RPC provider to route all other requests
engine.addProvider(new RedundantRPCSubprovider(['http://localhost:8545']));
engine.start();

const web3 = new Web3(engine);
// Optional, use within 0x.js
const zeroEx = new ZeroEx(web3.currentProvider, { networkId: 42 });
const accounts = await promisify<string>(web3.eth.getAccounts)();
console.log(accounts);
```

This above example works for Browser based applications, if you want to use the Ledger Nano S directly in a Node.js application, you must use a different `ledgerEthereumClientFactoryAsync`, for example:


```typescript
// Import the NodeJS Client Factory, rather than the Browser Client Factory
import {
    ledgerEthereumNodeJsClientFactoryAsync as ledgerEthereumClientFactoryAsync,
    LedgerSubprovider,
    RedundantRPCSubprovider
} from '@0xproject/subproviders';
```

The Ledger Subprovider has a public API to set various options. For example, the derivation path and the account index can be changed.

```typescript
public getPath(): string
public setPath(derivationPath: string)
public setPathIndex(pathIndex: number)
public async getAccountsAsync(): Promise<string[]>
public async signTransactionAsync(txParams: PartialTxParams): Promise<string>
public async signPersonalMessageAsync(data: string): Promise<string>
```

These methods can be used outside of a Web3 Provider if the application prefers to call them directly, rather than through the Web3 Provider Engine.


### Notes on Ledger Subprovider
It is important to remember that UI/UX components need to be considered when using the Ledger provider. A user may have multiple accounts on the ledger and assuming the first account may not be correct. The user may want to set the gas price, to either a high number or low number. These UI compnents must be considered.

Unlike a Browser wallet, the hardware displays are limited and can only show a subset of a transaction or signed message. To provide a good experience a dApp should consider displaying a confirmation window in the dApp at the same time as the ledger request is made. Only send one request at a time to a hardware device, unlike a Browser wallet they may not queue up and the experience could be confusing.