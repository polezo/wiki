When instantiating a [new instance of the 0x.js library](https://0x.org/docs/0x.js#zeroEx), we require that you pass in a Web3 provider. This Web3 provider allows your application to communicate with an Ethereum Node. Since there doesn't seem to be great documentation on what exactly a Web3 Provider is, we thought we'd fill the gap.


A provider can be any module or class instance that implements the `sendAsync` method (simply `send` in web3 V1.0 Beta). That's it. What this `sendAsync` method does is take JSON RPC payload requests and handles them. [Web3.js](https://github.com/ethereum/web3.js/) is an example of a javascript module that contains a Web3 HTTP Provider.


Using a configured Web3 Provider, the application can request signatures, estimate gas and gas price, and submit transactions to an Ethereum node.


The simplest example of a provider is:

```ts
const provider = new web3.providers.HttpProvider('http://localhost:8545');
```

This provider simply takes each JSON RPC payload it receives and sends it on to the Ethereum node running on port 8545.

### More complex providers

Providers can have much more complex behavior however, and [ProviderEngine](https://github.com/MetaMask/provider-engine) is a great tool for building providers that do all kinds of cool stuff.

We at 0x have put together a package of subproviders that we've needed ourselves. Our package also re-exports the `Web3ProviderEngine` from [ProviderEngine](https://github.com/MetaMask/provider-engine) so you only need to install a single package to get started. You can install it as follows:

```
npm install @0x/subproviders --save
```

We describe several of our subproviders below.

#### RedundantSubprovider

In 0x Portal we use a custom built provider called [RedundantSubprovider](https://github.com/0xProject/0x-monorepo/blob/cd08a9c1218fa7c4819e31248e50da2a4f45ee36/packages/subproviders/src/subproviders/redundant_subprovider.ts) that sends an RPC payload to several Ethereum nodes sequentially until one successfully handles it. During our token sale, the huge amount of traffic caused some of our Ethereum nodes to fail. Using this subprovider we were able to have requests sent to backup nodes when this happened, greatly improving the application's resilience to outages.

#### LedgerSubprovider

A great use case for custom providers is to support additional ways for your users to sign requests and send transactions. We wanted to add hardware wallet signing support to 0x Portal so we wrote a [LedgerSubprovider](https://github.com/0xProject/0x-monorepo/blob/cd08a9c1218fa7c4819e31248e50da2a4f45ee36/packages/subproviders/src/subproviders/ledger.ts) that would send all message and transaction signing requests to a users Ledger Nano S. All other requests would be sent to a backing Ethereum node.

#### SignerSubprovider

Many people use browser extension wallets (e.g [Metamask](https://metamask.io/)). These services inject a web3 provider into the page so that your dApp can relay signing requests. We wrote [SignerSubprovider](https://github.com/0xProject/0x-monorepo/blob/cd08a9c1218fa7c4819e31248e50da2a4f45ee36/packages/subproviders/src/subproviders/signer.ts) so that we could route all signing requests to the injected provider, but have all other requests handled by another backing Ethereum node.

#### Updating the provider used by `@0x/contract-wrappers`

If at some point the provider used by your dApp changes (e.g if your user wants to use their Ledger Nano S to sign orders), it is important that you update the provider used by your `@0x/contract-wrappers`. You can do this by calling [setProvider](https://0x.org/docs/contract-wrappers#ContractWrappers-setProvider).

```ts
await contractWrappers.setProvider(newProvider, networkId);
```
