The diagrams provided below demonstrate the interactions that occur between the various 0x smart contracts when executing trades or when upgrading exchange or governance logic. Arrows represent external function calls between Ethereum accounts (circles) or smart contracts (rectangles): arrows are directed from the caller to the callee.

#### Trade Execution (excl. fees)

<img src="https://s3.eu-west-2.amazonaws.com/0x-wiki-images/trade_execution.png" style="padding-bottom: 20px; padding-top: 20px" height="350" />

Transaction #1
1. `Exchange.fillOrder(order, value)`
2. `TokenTransferProxy.transferViaTokenTransferProxy(token, from, to, value)`
3. `Token(token).transferFrom(from, to, value)`
4. Token: (bool response)
5. TokenTransferProxy: (bool response)
6. `TokenTransferProxy.transferViaTokenTransferProxy(token, from, to, value)`
7. `Token(token).transferFrom(from, to, value)`
8. Token: (bool response)
9. TokenTransferProxy: (bool response)
10. Exchange: (bool response)

#### Upgrading the Exchange Contract

<img src="https://s3.eu-west-2.amazonaws.com/0x-wiki-images/upgrade_exchange.png" height="350" style="padding-bottom: 20px; padding-top: 20px" />

Transaction #1

1. `TokenTransferProxy.transferFrom(token, from, to, value)` 🚫

Transaction #2

2. `DAO.submitTransaction(destination, bytes)`

Transaction #3 (one tx per stakeholder)

3. `DAO.confirmTransaction(transactionId)`

Transaction #4

4. `DAO.executeTransaction(transactionId)`
5. `TokenTransferProxy.addAuthorizedAddress(Exchangev2)`

Transaction #5

6. `TokenTransferProxy.transferFrom(token, from, to, value)` ✅

#### Upgrading the Governance Contract

<img src="https://s3.eu-west-2.amazonaws.com/0x-wiki-images/upgrade_governance.png" height="350" style="padding-bottom: 20px; padding-top: 20px;" />
</div>

Transaction #1

1. `TokenTransferProxy.doSomething(...)` 🚫

Transaction #2

2. `DAOv1.submitTransaction(destination, bytes)`

Transaction #3 (one tx per stakeholder)

3. `DAOv1.confirmTransaction(transactionId)`

Transaction #4

4. `DAOv1.executeTransaction(transactionId)`
5. `TokenTransferProxy.transferOwnership(DAOv2)`

Transaction #5

6. `TokenTransferProxy.doSomething(...)`  ✅
