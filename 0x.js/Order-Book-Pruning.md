The most important function of any relayer is the effective maintenance of their orderbook. Over time there are many reasons why orders might become unfillable, including:

- The order becoming expired/cancelled/filled
- The order maker changing their allowance/balance associated with the order.

In order to avoid having unfillable orders on their orderbooks, relayers must continually monitor their orderbooks and remove orders if they become unfillable.

0x.js provides the method [validateOrderFillableOrThrowAsync](https://0xproject.com/docs/0xjs#validateOrderFillableOrThrowAsync) for validating if an order is still fillable.

```ts
await zeroEx.exchange.validateOrderFillableOrThrowAsync(signedOrder);
```

This method throws if the entirety of the remaining fillAmount of the order is not fillable. There are scenarios however where an order might still be partially fillable.

##### Example of partially fillable order

Alice create a valid 0x order to sell 10 WETH tokens. Later in the day, she uses the same account specified as the order maker to send a friend 2 WETH. She now only have 8 WETH remaining in her account. If the relayer validates her order, it will throw as it is no longer possible for someone to buy 10 WETH from her order. Someone could still fill it for up to 8 WETH however.

If a relayer wanted to keep Alice's order on their order book, they would need to pass an optional param to the above call:

```ts
const remainingAliceBalance = ZeroEx.toBaseUnitAmount(new BigNumber(8), 18);
const opts = {
    expectedFillTakerTokenAmount: remainingAliceBalance,
};
await zeroEx.exchange.validateOrderFillableOrThrowAsync(signedOrder, opts);
```

The relayer could fetch the makers balance & allowance and pass the amount still available to the validation method. This way, as long as the order could be filled up to 8 WETH, it will still be considered valid.

Handling these types of edge-cases however requires additional UI/UX work for the relayer since they will need to show traders the fillable amount of an order and should only let them attempt to fill up to the remaining fillable amount. The easiest solution is to simply remove orders that are not fillable in their entirety from the orderbook. It is up to each relayer to decide how they wish to prune their orderbook.
