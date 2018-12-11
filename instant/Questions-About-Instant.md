## FAQs

### General

#### Q: What is a SignedOrder?

A SignedOrder is a basic primitive of the 0x protocol. You can think of the SignedOrder as a set of parameters that define the terms of a trade, plus a signature that proves that the creator (also known as the 'maker') of an order has approved the trade. For example, a SignedOrder can specify: address A wants to trade 5 REP tokens in exchange for 1 WETH token. The order also includes signature produced by address A's private key and a hash of the order parameters. For exact JavaScript / TypeScript definition of a SignedOrder, refer to our [docs](https://0xproject.com/docs/0x.js#types-SignedOrder).

#### Q: What is the Standard Relayer API

The Standard relayer API is an HTTP specification that facilitates discovering and publishing SignedOrders. As an integrator of 0x Instant you will want to grab an API url from the [Relayer Registry](https://github.com/0xProject/0x-relayer-registry/blob/master/relayers.json). Check out the exact specification of the API in the [documentation](http://sra-spec.s3-website-us-east-1.amazonaws.com/).

#### Q: What is a provider?

Check out this [article](https://0xproject.com/wiki#Web3-Provider-Explained) in the 0x wiki for explanation of web3 providers. A dApp developer typically grabs this object from window.ethereum or window.web3.currentProvider. For advanced usage of providers check out this [article](https://0xproject.com/wiki#Web3-Provider-Examples) for some examples of how to create your own providers

#### Q: What is assetData?

Check out the 0x v2 protocol specification for more information on [assetData](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#assetdata).

#### Q: What is assetMetaData?

In order to provide a good user experience, Instant requires data that is not available on-chain. This includes the `decimals`, `symbol` and `name` of the token along with optional information like `primaryColor`, and `iconUrl` that informs Instant how to theme itself when that token is selected. It also requires to know whether the token is ERC20 or some other standard via the assetProxyId field.

**Example assetMetaDataMap**
```js
{
    '0xf47261b0000000000000000000000000744d70fdbe2bc4cf95131626614a1764df805b9e': {
        assetProxyId: '0xf47261b0', // ERC20 Proxy Id
        decimals: 18,
        symbol: 'XXX',
        name: 'My Custom Token',
        primaryColor: '#F2F7FF', // Optional
        iconUrl: 'https://cdn.icons.com/my_icon.svg', // Optional
    },
}
```

| Field | Description |
|-----------|-------------|
| assetProxyId | Only ERC20 is supported and the value for assetProxyId should always be “0xf47261b0” |
| decimals | The number of decimal places this token requires |
| symbol | The token symbol (ex: `ZRX`, `BAT`, etc...) |
| name | The full name of the token (ex: `0x`, `Basic Attention Token`, etc...) |
| primaryColor (optional) | The color Instant will theme itself when this token is selected |
| iconUrl (optional) | The url to the icon to use for this token |

 The icon referenced by `iconUrl` will go on top of a 26x26 circle that has `primaryColor` as a background. If an `iconUrl` is not provided, the specified token `symbol` will be displayed over the circle in white.

#### Q: Do users need to have ZRX to pay for fees on orders?

Nope! The Asset Buyer will calculate the ZRX required to pay for fees on the desired orders, and automatically purchase ZRX from your order source to cover your fees as part of the order. Note that Instant cannot use a user's existing ZRX balance and the liquidity source must be able to provide ZRX / ETH orders.

#### Q: Does this work with order matching relayers (vs. open orderbook)?

No, 0x Instant currently needs globally accessible orders and is only compatible with orders from open orderbook relayers.

#### Q: What happens if there's not enough liquidity to purchase an asset?

To prevent massive price ranges, 0x Instant calculates a maximum amount of assets that a user can buy under a given amount of price slippage and restricts purchasing to that amount.

#### Q: Can I pay for assets through 0x Instant using an ERC-20 token like WETH or DAI?

0x Instant currently only supports purchasing ERC-20 and ERC-721 assets using ETH for now, but we're looking into adding ERC-20 to ERC-20 purchasing in the future.

#### Q: Does 0x Instant work with permissioned liquidity pools that require KYC?

0x Instant currently passes all orders through a forwarding contract for wrapping ETH and filling orders and is therefore incompatible with many on-chain KYC solutions. Check with the KYC solution you're using to verify.

### Mobile

#### Q: How can I make 0x Instant work in my mobile app?

For apps using React Native or apps that have a web view, the asset buyer engine will work out of the box with your application. For apps that are written in a native language like Java or Swift, you will need to wrap the asset-buyer logic in a JS interpreter.

### NFTs

#### Q: How does 0x Instant help people buy NFTs?

0x Instant can act as a familiar interface for people to easily purchase NFTs with a single tap — no need to purchase wrap ETH or set an allowance!

### Affiliates

#### Q: How do I make money as an affiliate?

If you host 0x Instant, you can designate an address that you own to receive a small % of ETH that users spend on assets. The fee percent maxes out at 5%. You can configure this in the AffiliateInfo setting.
