One interesting feature enabled by the 0x protocol is the ability to remove the necessity for a user of a certain dApp or protocol to own it's native token in order to use it. The native token is still required, but it can be acquired seamlessly within the same Ethereum transaction as the action the user wishes to perform.

### Abstracting ZRX fees from 0x trades

Let's start with describing how a relayer can allow order **takers** to pay trading fees in WETH instead of ZRX. The way this is done is by enabling users to batch fill two orders in a single Ethereum transaction. The first will convert a sufficient amount of WETH to ZRX to cover the `takerFee` for the order they want to fill. The second order is the one the user is actually interested in filling.

Once the user has selected an order they want to fill, the relayer will supply the user with a second order exchanging WETH for ZRX at the current market price, without fees. The maker then submits both orders to the blockchain. Since the [batchFillOrders](https://0xproject.com/docs/contracts#batchFillOrders) 0x smart contract method fills orders synchronously, by the time the second order is filled, the user has acquired enough ZRX to pay the transaction fee of the second order.

##### Limitations

- Token abstraction cannot be used to pay the `makerFee` if one exists.
- Executing a batch fill costs more gas then a regular fill. As such, frequent traders will still prefer to hold a ZRX balance directly.

### Abstracting native tokens from a dApp/protocol

Token abstraction applied to other dApps and protocols have the same end-effect but work slightly differently.

Anyone can write and deploy a smart contract that implements token abstraction for an existing smart contract with a native token. The wrapper contract would accept a valid 0x order that converts another token to the native token along with any arguments required by the primary smart contract.

Let's say someone writes a token abstraction wrapper for the Golem network so that people can pay computation providers in ETH instead of GNT. They would do the following steps:

1. Fetch market orders exchanging WETH to GNT from several relayers using the [Standard Relayer API](https://blog.0xproject.com/introducing-the-0x-standard-relayer-api-8a37bd90a3e).
2. Call the Golem wrapper contract, passing in an 0x order exchanging sufficient WETH for GNT to perform a computation on the Golem network.

The smart contract would then do the following:

1. Make sure the user sent sufficient ETH when calling the wrapper smart contract.
2. Wrap the ETH into WETH ERC20-compliant tokens.
3. Call `fillOrder` on the 0x smart contract passing in the supplied order.
4. Call the Golem smart contract on behalf of the user, spending the newly acquired GNT.

In this way, the user owns GNT just in time for the transaction, and in the exact amount required for payment.

##### Limitations

- This again is more expensive in terms of gas then calling the Golem network directly. As such, it is great for on-boarding new users but frequent users would still prefer to hold a balance of GNT.
- In order for someone other then the authors of the dApp/protocol to add token abstraction, we need to use the `DELEGATECALL` opcode, and users must trust the wrapper smart contract to act honestly on their behalf.
