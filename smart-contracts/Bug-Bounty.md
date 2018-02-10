The Bug Bounty is live! Details regarding compensation for reported bugs can be found [here](https://blog.0xproject.com/announcing-the-0x-protocol-bug-bounty-b0559d2738c). Submissions should be based off of commit [ebf8ccfb012e2533094f00d6e813e6a086548619](https://github.com/0xProject/contracts/tree/ebf8ccfb012e2533094f00d6e813e6a086548619). Please e-mail all submissions to team@0xProject.com with the subject "BUG BOUNTY". Note that submissions already reported in our [security audits](https://github.com/ConsenSys/0x_review) or GitHub issues will not be eligible.

The following contracts are within the scope of the bug bounty:

#### Core Contracts

* [Exchange.sol](https://github.com/0xProject/contracts/blob/ebf8ccfb012e2533094f00d6e813e6a086548619/contracts/Exchange.sol)
* [TokenTransferProxy.sol](https://github.com/0xProject/contracts/blob/ebf8ccfb012e2533094f00d6e813e6a086548619/contracts/TokenTransferProxy.sol)
* [TokenRegistry.sol](https://github.com/0xProject/contracts/blob/ebf8ccfb012e2533094f00d6e813e6a086548619/contracts/TokenRegistry.sol)
* [ZRXToken.sol](https://github.com/0xProject/contracts/blob/ebf8ccfb012e2533094f00d6e813e6a086548619/contracts/tokens/ZRXToken.sol)
* [MultiSigWalletWithTimeLockExceptRemoveAuthorizedAddress.sol](https://github.com/0xProject/contracts/blob/ebf8ccfb012e2533094f00d6e813e6a086548619/contracts/MultiSigWalletWithTimeLockExceptRemoveAuthorizedAddress.sol)
* [TokenSale.sol](https://github.com/0xProject/contracts/blob/ebf8ccfb012e2533094f00d6e813e6a086548619/contracts/TokenSale.sol)

#### Wallets

* [MultiSigWallet.sol](https://github.com/0xProject/contracts/blob/ebf8ccfb012e2533094f00d6e813e6a086548619/contracts/base/MultiSigWallet.sol) (There is an additional growing bug bounty for this contract [here](https://www.reddit.com/r/ethdev/comments/6qaxc1/bug_bounty_on_the_consensys_and_whitelisted/))
* [MultiSigWalletWithTimeLock.sol](https://github.com/0xProject/contracts/blob/ebf8ccfb012e2533094f00d6e813e6a086548619/contracts/MultiSigWalletWithTimeLock.sol)
* [VestingWallet.sol](https://github.com/0xProject/vesting-wallet/blob/5b546aaa28cca843d52c01bc02800d702f6f6135/contracts/VestingWallet.sol) (This is in a different repo, found [here](https://github.com/0xProject/vesting-wallet))

#### Tokens

* [EtherToken.sol](https://github.com/0xProject/contracts/blob/ebf8ccfb012e2533094f00d6e813e6a086548619/contracts/tokens/EtherToken.sol)
* [StandardTokenWithOverflowProtection.sol](https://github.com/0xProject/contracts/blob/ebf8ccfb012e2533094f00d6e813e6a086548619/contracts/base/StandardTokenWithOverflowProtection.sol)
* [StandardToken.sol](https://github.com/0xProject/contracts/blob/ebf8ccfb012e2533094f00d6e813e6a086548619/contracts/base/StandardToken.sol)

#### Base Contracts

* [Ownable.sol](https://github.com/0xProject/contracts/blob/ebf8ccfb012e2533094f00d6e813e6a086548619/contracts/base/Ownable.sol)
* [SafeMath.sol](https://github.com/0xProject/contracts/blob/ebf8ccfb012e2533094f00d6e813e6a086548619/contracts/base/SafeMath.sol)
