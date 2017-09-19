A relayer can provide its own liquidity by frequently posting large orders with short expiration times where the `taker` field is equal to `0x0000000000000000000000000000000000000000`. This can be used separately from or in conjunction with the [open orderbook](https://github.com/0xProject/wiki/blob/master/relayer-strategies/Open-Orderbook.md) strategy.

##### Example
Relayer Alice broadcasts orderA to sell 100 WETH in exchange for 100000 ZRX that expires in 45 seconds. Bob calls `fillOrder(orderA, 1000)`, exchanging his 1000 ZRX for Alice's 1 WETH. Charlie calls `fillOrder(orderA, 2000)`, exchanging his 2000 ZRX for Alice's 2 WETH.

##### Rationale
This strategy can be used by large market makers who are willing to reliably provide liquidity to traders. Note that orders can be broadcasted through any arbitrary communication channel, so this strategy can be implemented off-chain, through event logs, or on-chain if desired.

##### Limitations
This strategy suffers from the same limitations as the open orderbook model, although the likelihood of a race occuring becomes smaller as order sizes grow. In addition, it requires the relayer to hold large reserves of tokens in order to make markets. 