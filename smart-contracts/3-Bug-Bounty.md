A bug bounty is live for 0x protocol 2.0.0. Submissions should be based off of the contracts as of commit [965d6098294beb22292090c461151274ee6f9a26](https://github.com/0xProject/0x-monorepo/tree/965d6098294beb22292090c461151274ee6f9a26/packages/contracts/src/2.0.0).

### Rewards

The severity of reported vulnerabilities will be graded according to the [CVSS](https://www.first.org/cvss/) (Common Vulnerability Scoring Standard). The following table will serve as a guideline for reward decisions:

| Critical (CVSS 9.0 - 10.0) | High (CVSS 7.0 - 8.9) | Medium (CVSS 4.0 - 6.9) | Low (CVSS 0.0 - 3.9) |
| -------------------------- | --------------------- | ----------------------- | -------------------- |
| $10,000 - $100,000         | $2,500 - $10,000      | $1,000 - $2,500         | $0 - $1,000          |

Please note that any rewards will ultimately be awarded at the discretion of ZeroEx Intl. All rewards will be paid out in ZRX.

### Areas of interest

The following are examples of types of vulnerabilities that are of interest:

-   Loss of assets
    -   A user loses assets in a way that they did not explicitly authorize (e.g an account is able to gain access to an `AssetProxy` and drain user funds).
    -   A user authorized a transaction or trade but spends more assets than normally expected (e.g an order is allowed to be over-filled).
-   Unintended contract state
    -   A user is able to update the state of a contract such that it is no longer useable (e.g permanently lock a mutex).
    -   Any assets get unexpectedly "stuck" in a contract with regular use of the contract's public methods.
-   Bypassing time locks
    -   The `AssetProxyOwner` is allowed to bypass the timelock for transactions where it is not explicitly allowed to do so.
    -   A user is allowed to bypass the `AssetProxyOwner`.

### Scope

The contracts found in the following directories are considered within scope of this bug bounty:

-   `src/2.0.0/protocol`
-   `src/2.0.0/utils`
-   `src/2.0.0/multisig/MultiSigWalletWithTimeLock`
-   `src/2.0.0/extensions/Forwarder`

Please note that any bugs already reported are considered out of scope. Security audits of these contracts can be found [here](https://docs.google.com/document/d/1jYv6V21MfCSwCS5fxD6ZyaLWGzkpRSUO0lZpST94XsA/edit) and [here](https://github.com/ConsenSys/0x_audit_report_2018-07-23).

### Disclosures

Please e-mail all submissions to team@0x.org with the subject "BUG BOUNTY". Your submission should include any steps required to reproduce or exploit the vulnerability. Please allow time for the vulnerability to be fixed before discussing any findings publicly. After receiving a submission, we will contact you with expected timelines for a fix to be implemented.
