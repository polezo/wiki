Each order is a data packet containing order parameters and an associated signature. Order parameters are concatenated and hashed to 32 bytes via the Keccak SHA3 function. The order originator signs the order hash with their private key to produce an ECDSA signature.

<table>
    <thead>
        <th>Name</th>
        <th>Data Type</th>
        <th>Description</th>
    </thead>
    <tbody>
        <tr>
            <td>exchangeContractAddress</td>
            <td>address</td>
            <td>Address of the Exchange contract. This address will change each time the protocol is updated.</td>
        </tr>
        <tr>
            <td>maker</td>
            <td>address</td>
            <td>Address originating the order.</td>
        </tr>
        <tr>
            <td>taker</td>
            <td>address</td>
            <td>Address permitted to fill the order (optional).</td>
        </tr>
        <tr>
            <td>makerToken</td>
            <td>address</td>
            <td>Address of an ERC20 Token contract.</td>
        </tr>
        <tr>
            <td>takerToken</td>
            <td>address</td>
            <td>Address of an ERC20 Token contract.</td>
        </tr>
        <tr>
            <td>makerTokenAmount</td>
            <td>uint256</td>
            <td>Total units of makerToken offered by maker.</td>
        </tr>
        <tr>
            <td>takerTokenAmount</td>
            <td>uint256</td>
            <td>Total units of takerToken requested by maker.</td>
        </tr>
        <tr>
            <td>expirationTimestampInSec</td>
            <td>uint256</td>
            <td>Time at which the order expires (seconds since unix epoch).</td>
        </tr>
        <tr>
            <td>salt</td>
            <td>uint256</td>
            <td>Arbitrary number that allows for uniqueness of the order's Keccak SHA3 hash.</td>
        </tr>
        <tr>
            <td>feeRecipient</td>
            <td>address</td>
            <td>Address that recieves transaction fees (optional).</td>
        </tr>
        <tr>
            <td>makerFee</td>
            <td>uint256</td>
            <td>Total units of ZRX paid to feeRecipient by maker.</td>
        </tr>
        <tr>
            <td>takerFee</td>
            <td>uint256</td>
            <td>Total units of ZRX paid to feeRecipient by taker.</td>
        </tr>
        <tr>
            <td>v</td>
            <td>uint8</td>
            <td>ECDSA signature of the above arguments.</td>
        </tr>
        <tr>
            <td>r</td>
            <td>bytes32</td>
            <td>ECDSA signature of the above arguments.</td>
        </tr>
        <tr>
            <td>s</td>
            <td>bytes32</td>
            <td>ECDSA signature of the above arguments.</td>
        </tr>
    </tbody>
</table>
