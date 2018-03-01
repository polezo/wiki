The 0x smart contracts can be called from other smart contracts in order to build more complex systems on top of the 0x protocol. In this tutorial we will show you how to write an atomic arbitrage smart contract between EtherDelta and 0x, as an example of what can be built using 0x at the smart contract level. Our arbitrage smart contract will either succeed at executing two trades (one on 0x, the other on EtherDelta) and make a guaranteed profit or fail both trades and leave our balances untouched. This is useful since there is no strong guarantee that either trade succeeds, and were we to simply submit both trades independently to the blockchain, we run the risk of only a single trade going through and having unwanted tokens in our possession.

### Interfaces of both contracts

#### 0x Exchange contract

In order to be able to trade via the [0x Exchange contract](https://github.com/0xProject/0x.js/blob/development/packages/contracts/src/current/protocol/Exchange/Exchange.sol) we need to set an allowance for [0x Proxy contract](https://github.com/0xProject/0x.js/blob/development/packages/contracts/src/current/protocol/TokenTransferProxy/TokenTransferProxy.sol).

#### EtherDelta contract

In order to be able to trade via an EtherDelta we need to:

* Set an allowance for EtherDelta contract
* Call `etherDelta.depositToken(token, amount)`
* Withdraw tokens after the trade `etherDelta.withdrawToken(token, amount)`

### Arbitrage smart contract

Here is the example code for the Arbitrage smart contract. It's not intended to be gas efficient or safe. Please don't use this in production. It is solely for educational purposes.

```solidity
pragma solidity ^0.4.19;

import { Exchange } from "../../protocol/Exchange/Exchange.sol";
import { EtherDelta } from "../EtherDelta/EtherDelta.sol";
import { Ownable } from "../../utils/Ownable/Ownable.sol";
import { Token } from "../../tokens/Token/Token.sol";

/// @title Arbitrage - Facilitates atomic arbitrage of ERC20 tokens between EtherDelta and 0x Exchange contract.
/// @author Leonid Logvinov - <leo@0xProject.com>
contract Arbitrage is Ownable {

    Exchange exchange;
    EtherDelta etherDelta;
    address proxyAddress;

    uint256 constant MAX_UINT = 2**256 - 1;

    function Arbitrage(address _exchangeAddress, address _etherDeltaAddress, address _proxyAddress) {
        exchange = Exchange(_exchangeAddress);
        etherDelta = EtherDelta(_etherDeltaAddress);
        proxyAddress = _proxyAddress;
    }

    /*
     * Makes token tradeable by setting an allowance for etherDelta and 0x proxy contract.
     * Also sets an allowance for the owner of the contracts therefore allowing to withdraw tokens.
     */
    function setAllowances(address tokenAddress) external onlyOwner {
        Token token = Token(tokenAddress);
        token.approve(address(etherDelta), MAX_UINT);
        token.approve(proxyAddress, MAX_UINT);
        token.approve(owner, MAX_UINT);
    }

    /*
     * Because of the limits on the number of local variables in Solidity we need to compress parameters while loosing
     * readability. Scheme of the parameter layout:
     *
     * addresses
     * 0..4 orderAddresses
     * 5 user
     *
     * values
     * 0..5 orderValues
     * 6 fillTakerTokenAmount
     * 7 amountGet
     * 8 amountGive
     * 9 expires
     * 10 nonce
     * 11 amount

     * signature
     * exchange then etherDelta
     */
    function makeAtomicTrade(
        address[6] addresses, uint[12] values,
        uint8[2] v, bytes32[2] r, bytes32[2] s
    ) external onlyOwner {
        makeExchangeTrade(addresses, values, v, r, s);
        makeEtherDeltaTrade(addresses, values, v, r, s);
    }

    function makeEtherDeltaTrade(
        address[6] addresses, uint[12] values,
        uint8[2] v, bytes32[2] r, bytes32[2] s
    ) internal {
        uint amount = values[11];
        etherDelta.depositToken(
            addresses[2], // tokenGet === makerToken
            values[7] // amountGet
        );
        etherDelta.trade(
            addresses[2], // tokenGet === makerToken
            values[7], // amountGet
            addresses[3], // tokenGive === takerToken
            values[8], // amountGive
            values[9], // expires
            values[10], // nonce
            addresses[5], // user
            v[1],
            r[1],
            s[1],
            amount
        );
        etherDelta.withdrawToken(
            addresses[3], // tokenGive === tokenToken
            values[8] // amountGive
        );
    }

    function makeExchangeTrade(
        address[6] addresses, uint[12] values,
        uint8[2] v, bytes32[2] r, bytes32[2] s
    ) internal {
        address[5] memory orderAddresses = [
            addresses[0], // maker
            addresses[1], // taker
            addresses[2], // makerToken
            addresses[3], // takerToken
            addresses[4] // feeRecepient
        ];
        uint[6] memory orderValues = [
            values[0], // makerTokenAmount
            values[1], // takerTokenAmount
            values[2], // makerFee
            values[3], // takerFee
            values[4], // expirationTimestampInSec
            values[5]  // salt
        ];
        uint fillTakerTokenAmount = values[6]; // fillTakerTokenAmount
        // Execute Exchange trade. It either succeeds in full or fails and reverts all the changes.
        exchange.fillOrKillOrder(orderAddresses, orderValues, fillTakerTokenAmount, v[0], r[0], s[0]);
    }
}
```

The usage example as well as the tests can be found [here](https://github.com/0xProject/0x.js/blob/development/packages/contracts/test/tutorials/arbitrage.ts).
