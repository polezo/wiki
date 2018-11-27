## FAQs

### General

_Q: What is a SignedOrder?_

A SignedOrder is a basic primitive of the 0x protocol. You can think of the SignedOrder as a set of parameters that define the terms of a trade, plus a signature that proves that the creator (also known as the 'maker') of an order has approved the trade. For example, a SignedOrder can specify: address A wants to trade 5 REP tokens in exchange for 1 WETH token. The order also includes signature produced by address A's private key and a hash of the order parameters. For exact JavaScript / TypeScript definition of a SignedOrder, refer to our [docs](https://0xproject.com/docs/0x.js#types-SignedOrder).

_Q: What is the Standard Relayer API_

The Standard relayer API is an HTTP specification that facilitates discovering and publishing SignedOrders. As an integrator of 0x Instant you will want to grab an API url from the [Relayer Registry](https://github.com/0xProject/0x-relayer-registry/blob/master/relayers.json). Check out the exact specification of the API in the [documentation](http://sra-spec.s3-website-us-east-1.amazonaws.com/).

_Q: What is a provider?_

Check out this [article](https://0xproject.com/wiki#Web3-Provider-Explained) in the 0x wiki for explanation of web3 providers. A dApp developer typically grabs this object from window.ethereum or window.web3.currentProvider. For advanced usage of providers check out this [article](https://0xproject.com/wiki#Web3-Provider-Examples) for some examples of how to create your own providers

_Q: What is assetData?_

Check out the 0x v2 protocol specification for more information on [assetData](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#assetdata).

_Q: Do users need to have ZRX to pay for fees on orders?_

Nope! The Asset Buyer will calculate the ZRX required to pay for fees on the desired orders, and automatically purchase ZRX from your order source to cover your fees as part of the order. Note that Instant cannot use a user's existing ZRX balance and the liquidity source must be able to provide ZRX / ETH orders.

_Q: Does this work with order matching relayers (vs. open orderbook)?_

No, 0x Instant currently needs globally accessible orders and is only compatible with orders from open orderbook relayers.

_Q: What happens if there's not enough liquidity to purchase an asset?_

To prevent massive price ranges, 0x Instant calculates a maximum amount of assets that a user can buy under a given amount of price slippage and restricts purchasing to that amount.

_Q: Can I pay for assets through 0x Instant using an ERC-20 token like WETH or DAI?_

0x Instant currently only supports purchasing ERC-20 and ERC-721 assets using ETH for now, but we're looking into adding ERC-20 to ERC-20 purchasing in the future.

_Q: Does 0x Instant work with permissioned liquidity pools that require KYC?_

0x Instant currently passes all orders through a forwarding contract for wrapping ETH and filling orders and is therefore incompatible with many on-chain KYC solutions. Check with the KYC solution you're using to verify.

_Q: What is assetMetaData?_

In order to provide a good user experience, Instant requires data that is not available on-chain. This includes the decimals, symbol and name of the token along with optional information like primaryColor that informs Instant how to theme itself when that token is selected. It also requires to know whether the token is ERC20 or some other standard via the assetProxyId field. At the moment, only ERC20 is supported and the value for assetProxyId should always be “0xf47261b0”.

### Mobile

_Q: How can I make 0x Instant work in my mobile app?_

For apps using React Native or apps that have a web view, the asset buyer engine will work out of the box with your application. For apps that are written in a native language like Java or Swift, you will need to wrap the asset-buyer logic in a JS interpreter.

### NFTs

_Q: How does 0x Instant help people buy NFTs?_

0x Instant can act as a familiar interface for people to easily purchase NFTs with a single tap — no need to purchase wrap ETH or set an allowance!

### Affiliates

_Q: How do I make money as an affiliate?_

If you host 0x Instant, you can designate an address that you own to receive a small % of ETH that users spend on assets. The fee percent maxes out at 5%. You can configure this in the AffiliateInfo setting.
