In order to run 0x.js methods that interact with the Ethereum blockchain (i.e filling an order, checking a balance or setting an allowance) you need to point your Web3 Provider to an Ethereum node. Because of the ~12 second block times, during development it is best to use [TestRPC](https://github.com/ethereumjs/testrpc) a fast Ethereum RPC client made for testing and development.

Install TestRPC locally:

```
npm install -g ethereumjs-testrpc
```

In order to run TestRPC with all the latest 0x protocol smart contracts available, you must first download [this TestRPC snapshot](https://s3.amazonaws.com/testrpc-shapshots/07d00cc515e0f9825b81595386b358593b7a3d6f.zip) and save it. Next unzip it's contents with:

```bash
unzip ./07d00cc515e0f9825b81595386b358593b7a3d6f.zip -d ./0x_testrpc_snapshot
```

You can now start TestRPC as follows:

```bash
testrpc \
--networkId 50 \
-p 8545 \
--db ./0x_testrpc_snapshot \
-m "concert load couple harbor equip island argue ramp clarify fence smart topic"
```

**Note:** The `--db` flag expects the filepath to where the DB snapshot folder is located on your machine.

Since we started TestRPC on port 8545, we can pass ZeroEx the following provider during instantiation:

```
const provider = new Web3.providers.HttpProvider('http://localhost:8545');
```

0x.js will now communicate with TestRPC over HTTP!

### Contract addresses

- ZRXToken.sol: `0x1d7022f5b17d2f8b695918fb48fa1089c9f85401`
- EtherToken.sol: `0x871dd7c2b4b25e1aa18728e9d5f2af4c4e431f5c`
- Exchange.sol: `0x48bacb9266a570d521063ef5dd96e61686dbe788`
- TokenRegistry.sol: `0x0b1ba0af832d7c05fd64161e0db78e85978e8082`
- TokenTransferProxy.sol: `0x1dc4c1cefef38a777b15aa20260a54e584b16c48`

The entirety of the ZRX balance is in the `0x5409ed021d9299bf6814279a6a1411a7e866a631` user account setup by TestRPC.
