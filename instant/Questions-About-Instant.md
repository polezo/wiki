## FAQs

### General

#### Q: What is a SignedOrder?

A SignedOrder is a basic primitive of the 0x protocol. You can think of the SignedOrder as a set of parameters that define the terms of a trade, plus a signature that proves that the creator (also known as the 'maker') of an order has approved the trade. For example, a SignedOrder can specify: address A wants to trade 5 REP tokens in exchange for 1 WETH token. The order also includes signature produced by address A's private key and a hash of the order parameters. For exact JavaScript / TypeScript definition of a SignedOrder, refer to our [docs](https://0x.org/docs/0x.js#types-SignedOrder).

#### Q: What is the Standard Relayer API

The Standard relayer API is an HTTP specification that facilitates discovering and publishing SignedOrders. As an integrator of 0x Instant you will want to grab an API url from the [Relayer Registry](https://github.com/0xProject/0x-relayer-registry/blob/master/relayers.json). Check out the exact specification of the API in the [documentation](http://sra-spec.s3-website-us-east-1.amazonaws.com/).

#### Q: What is a provider?

Check out this [article](https://0x.org/wiki#Web3-Provider-Explained) in the 0x wiki for explanation of web3 providers. A dApp developer typically grabs this object from window.ethereum or window.web3.currentProvider. For advanced usage of providers check out this [article](https://0x.org/wiki#Web3-Provider-Examples) for some examples of how to create your own providers

#### Q: How can I check liquidity of an asset prior to rendering Instant?

There is a helper method available at `zeroExInstant.hasLiquidityForAssetDataAsync` which takes in an assetData string and order source (Standard Relayer API url or an array of signed orders) and returns a Promise which resolves a boolean value indicating if there is an liquidity available for a given assetData.

See below for an example of how to check liquidity on Radar Relay for a given ERC20 token address:

```
<div id="liquidityContainer">Loading...</div>
<script>
    const erc20TokenAddress = '0xd26114cd6EE289AccF82350c8d8487fedB8A0C07';
    const assetData = zeroExInstant.assetDataForERC20TokenAddress(erc20TokenAddress);

    zeroExInstant.hasLiquidityForAssetDataAsync(assetData, 'https://api.relayer.com/sra/v2/').then(l => {
        document.getElementById('liquidityContainer').innerHTML = `Relayer has liquidity: ${l ? 'Yes' : 'No'}`;
    });
</script>
```

Codepen [example](https://codepen.io/steveklebanoff/pen/dwExJz)

#### Q: What is assetData?

As we now support multiple [token transfer proxies](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#assetproxy), the identifier of which proxy to use for the token transfer must be encoded, along with the token information. Each proxy in 0x v2 has a unique identifier. If you're using 0x.js there will be helper methods for this [encoding](https://0x.org/docs/0x.js#assetDataUtils-encodeERC20AssetData) and [decoding](https://0x.org/docs/0x.js#assetDataUtils-decodeAssetProxyId).

The identifier for the Proxy uses a similar scheme to [ABI function selectors](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI#function-selector).

```
// ERC20 Proxy ID  0xf47261b0
bytes4(keccak256("ERC20Token(address)"))
// ERC721 Proxy ID 0x02571792
bytes4(keccak256("ERC721Token(address,uint256)"))
```

Asset data is encoded using [ABI encoding](https://solidity.readthedocs.io/en/develop/abi-spec.html).

For example, encoding the ERC20 token contract (address: 0x1dc4c1cefef38a777b15aa20260a54e584b16c48) using the ERC20 Transfer Proxy (id: 0xf47261b0) would be:

```
0xf47261b00000000000000000000000001dc4c1cefef38a777b15aa20260a54e584b16c48
```

Encoding the ERC721 token contract (address: `0x371b13d97f4bf77d724e78c16b7dc74099f40e84`), token id (id: `99`, which hex encoded is `0x63`) and the ERC721 Transfer Proxy (id: `0x02571792`) would be:

```
0x02571792000000000000000000000000371b13d97f4bf77d724e78c16b7dc74099f40e840000000000000000000000000000000000000000000000000000000000000063
```

For convenience, zeroExInstant exposes a `zeroExInstant.assetDataForERC20TokenAddress` method, which returns the `assetData` for a given ERC20 address.

For more information see [the Asset Proxy](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#erc20proxy) section of the v2 spec and the [Ethereum ABI Spec](https://solidity.readthedocs.io/en/develop/abi-spec.html).

#### Q: What is assetMetaData?

In order to provide a good user experience, Instant requires data that is not available on-chain. This includes the `decimals`, `symbol` and `name` of the token along with optional information like `primaryColor`, and `iconUrl` that informs Instant how to theme itself when that token is selected. It also requires to know whether the token is ERC20 or some other standard via the assetProxyId field.

The keys in this mapping are the `assetData` (see "What is assetData?" above) for the corresponding token.

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

| Field                   | Description                                                                          |
| ----------------------- | ------------------------------------------------------------------------------------ |
| assetProxyId            | Only ERC20 is supported and the value for assetProxyId should always be “0xf47261b0” |
| decimals                | The number of decimal places this token requires                                     |
| symbol                  | The token symbol (ex: `ZRX`, `BAT`, etc...)                                          |
| name                    | The full name of the token (ex: `0x`, `Basic Attention Token`, etc...)               |
| primaryColor (optional) | The color Instant will theme itself when this token is selected                      |
| iconUrl (optional)      | The url to the icon to use for this token                                            |

The icon referenced by `iconUrl` will go on top of a 26x26 circle that has `primaryColor` as a background. If an `iconUrl` is not provided, the specified token `symbol` will be displayed over the circle in white.

We provide assetMetaData for a subset of ERC20 tokens. You can call the helper function `zeroExInstant.hasMetaDataForAssetData` which returns a boolean indiciating whether or not we have assetMetaData for a given assetData string.

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
