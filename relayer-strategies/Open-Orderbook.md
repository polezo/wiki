### Open Orderbook
This is the simplest and most obvious strategy a relayer can use. The relayer accepts and broadcasts orders where the `taker` field is equal to `0x0000000000000000000000000000000000000000`. Any trader with access to all of an order's parameters is able to fill the order by signing a transaction locally and sending it to an Ethereum node.

##### Example
Alice submits orderA to sell 1 WETH in exchange for 1000 ZRX to relayer Charlie. Bob sees the order on Charlie's orderbook and calls `fillOrder(orderA, 1000)` using his own local node. Bob's 1000 ZRX are exchanged for Alice's 1 WETH.

##### Rationale
This is arguably the cheapest and most trustless strategy to implement. In addition, orders on an open orderbook are easily accessible to third parties. This comes with numerous benefits, including:

* Orders are able to be shared across relayers.
* External dApps can use these orders for liquidity.
* External smart contracts can use these orders for liquidity.

##### Limitations
As trading throughput increases, it becomes increasingly likely that multiple traders will attempt to fill the same order (either intentionally or accidentally). This strategy benefits the most from the ability of relayers and traders to quickly and reliably monitor the pending transaction pool.