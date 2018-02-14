A relayer can provide its own liquidity by simply broadcasting an intent to trade (unsigned order). Traders can submit a signed order with the specified rate to the relayer (optionally, the `taker` field can be the relayer's address), who can then choose to fill the order using owned tokens or external liquidity.

##### Example

Relayer Charlie says that he is willing to buy ZRX at a rate of 1000 ZRX per WETH. Alice sends orderA to sell 10000 ZRX in exchange for 10 WETH to Charlie. Charlie calls `fillOrder(orderA, 10000)`, exchanging his 10 WETH for Alice's 10000 ZRX.

##### Rationale

Since this strategy does not require any commitments to be made until the trader agrees to a rate, any arbitrary off-chain negotiation process can occur before an order is signed. Once an order is signed, the relayer also has the option of hedging with external liquidity, so the relayer does not necessarily need as large reserves as with the [reserve manager](https://github.com/0xProject/wiki/blob/master/relayer-strategies/Reserve-Manager.md) strategy.

##### Limitations

After a trader signs an order, the relayer is in no way obligated to fill that order (although the relayer may suffer a loss of reputation). The relayer also still needs some token reserves to utilize this strategy until the addition of an [atomic matching function](https://github.com/0xProject/ZEIPs/issues/2) to the protocol.
