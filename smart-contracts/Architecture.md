0x uses a modular system of Ethereum smart contracts, allowing each component of the system to be upgraded without effecting the other components or causing active markets to be disrupted. The business logic that is responsible for executing trades is separated from governance logic and trading balance access controls. Not only can the trade execution module be swapped out, so can the governance module.

<div style="text-align: center;">
<img src="https://s3.eu-west-2.amazonaws.com/0x-wiki-images/architecture_diagram.png" style="padding-bottom: 20px; padding-top: 20px;" height="450" width="80%" />
</div>

### Contract Descriptions

#### [Exchange.sol](https://github.com/0xProject/contracts/tree/master/contracts/Exchange.sol)
Exchange contains all business logic associated with executing trades and cancelling orders. Exchange accepts orders that conform to 0x message format, allowing for off-chain order relay with on-chain settlement. Exchange is designed to be replaced as protocol improvements are adopted over time. It follows that Exchange does not have direct access to ERC20 token allowances; instead, all transfers are carried out by TokenTransferProxy on behalf of Exchange.

#### [TokenTransferProxy.sol](https://github.com/0xProject/contracts/tree/master/contracts/TokenTransferProxy.sol)
TokenTransferProxy is analogous to a valve that may be opened or shut by MultiSigWalletWithTimeLock, either allowing or preventing Exchange from executing trades. TokenTransferProxy plays a key role in 0x protocol's update mechanism: old versions of the Exchange contract may be deprecated, preventing them from executing further trades. New and improved versions of the Exchange contract are given permission to execute trades through decentralized governance implemented within a DAO (for now we use MultiSigWallet as a placeholder for DAO).

#### [MultiSigWalletWithTimeLockExceptRemoveAuthorizedAddress.sol](https://github.com/0xProject/contracts/tree/master/contracts/MultiSigWalletWithTimeLockExceptRemoveAuthorizedAddress.sol)
MultiSigWalletWithTimeLockExceptRemoveAuthorizedAddress is a temporary placeholder contract that will be replaced by a thoroughly researched, tested and audited DAO. MultiSigWalletWithTimeLock is based upon the [MultiSigWallet](https://github.com/ConsenSys/MultiSigWallet) contract developed by the Gnosis team, but with an added mandatory time delay between when a proposal is approved and when that proposal may be executed. This speed bump ensures that the multi sig owners cannot conspire to push through a change to 0x protocol without end users having sufficient time to react. An exception to the speed bump is made for revoking access to trading balances in the unexpected case of a bug being found within Exchange. MultiSigWalletWithTimeLock is the only entity with permission to grant or revoke access to the TokenTransferProxy and, by extension, ERC20 token allowances. MultiSigWalletWithTimeLock is assigned as the `owner` of TokenTransferProxy and, once a suitable DAO is developed, MultiSigWalletWithTimeLock will call `TokenTransferProxy.transferOwnership(DAO)` to transfer permissions to the DAO.

#### [TokenRegistry.sol](https://github.com/0xProject/contracts/tree/master/contracts/TokenRegistry.sol)
TokenRegistry stores metadata associated with ERC20 tokens. TokenRegistry entries may only be created/modified/removed by MultiSigWallet (until it is replaced by a suitable DAO), meaning that information contained in the registry will generally be trustworthy. 0x message format is not human-readable making it difficult to visually verify order parameters (token addresses and exchange rates); the TokenRegistry can be used to quickly verify order parameters against audited metadata.
