# Blockchain Supply Chain Traceability System — English Lab Manual

This English lab manual documents the project under:
`Blockchain-based_supply_chain_traceability_system/`

It covers the project objective, on-chain data model design, contract/module overview, traceability workflow (fund flow + parts/procedure flow), and how to deploy and test it with Truffle/Ganache.

---

## 1. Project Overview

The project implements a **blockchain-based supply chain finance and traceability system**.
It provides:

1. **Fund flow traceability**
   - A *cash flow* tracks a lender-to-borrower budget and maintains its current balance.
   - A *transaction* represents a concrete transfer event within a cash flow.
   - On-chain query functions allow users to retrieve the cash flow details and the list of linked transactions.

2. **Parts (component) traceability**
   - A *parts flow* represents a component/batch created by a maker, including its metadata and links to its former (sub-)parts.
   - A *procedure* records a processing step performed on a specific parts flow.
   - On-chain query functions allow users to retrieve a parts flow’s procedures list and former parts list.

The system is implemented as a Truffle project (see `TraceabilitySystem/`).

---

## 2. Development Environment

The repository README lists the intended environment:

- Ubuntu 18.04.6 LTS
- Go 1.14.4
- Solc 0.8.11
- Node 14.18.3
- Npm 6.14.15
- Ganache 6.12.2
- Truffle 5.4.29
- VSCode 1.67

---

## 3. Lab Objectives

After completing the lab and using the project, you should be able to:

1. Understand the on-chain data structures for:
   - `CashFlow` and `Transaction`
   - `PartsFlow` and `Procedure`
2. Deploy the contracts to a local chain (Ganache/Truffle development network).
3. Create and query:
   - Lenders, borrowers, and managers
   - Cash flows and their transactions
   - Parts flows and their procedures
4. Implement a traceability workflow conceptually by:
   - From a cash-flow ID, retrieving linked transactions and details
   - From a parts-flow ID, retrieving linked procedure IDs and former parts IDs
5. Run the provided Truffle tests (if the environment matches the project expectations).

---

## 4. Contract and Module Overview

All contracts are located in:
`TraceabilitySystem/contracts/`

### 4.1 Top-level system contract

`TraceabilitySystem.sol`

This contract maintains arrays of instantiated objects:
- `Lender[] lenders`
- `Borrower[] borrowers`
- `Manager[] managers`
- `CashFlow[] cashFlows`
- `Transaction[] transactions`
- `PartsFlow[] partsflows`
- `Procedure[] procedures`

It provides the primary public APIs to:
- create roles (`createLender`, `createBorrower`, `createManager`)
- create and link flows and objects:
  - `createCashFlow`
  - `createTransaction`
  - `createPartsFlow`
  - `createProcedure`
- query details:
  - `checkCashFlow`
  - `checkTransaction`
  - `checkPartsFlow`
  - `checkProcedure`

It also emits events:
- `CreateCashFlow(from, to, amount)`
- `CreateTransaction(from, to, cashFlowID, amount)`

### 4.2 Fund-flow contracts

`CashFlow.sol`
- `id` (uint256)
- `initBalance`, `currBalance`
- `Transactions` (uint256[] linked transaction IDs)

Key methods:
- `withdraw(cashNum)` returns `bool` and updates `currBalance`
- `attachTransaction(transactionID)` appends to `Transactions`
- `checkCashFlow()` returns `(initBalance, currBalance, Transactions)`

`Transaction.sol`
- `id` (uint256)
- `cashFlowID`
- `senderID` and `receiverID` (address)
- `beforeTrans`, `afterTrans`, `cashNum`
- `note` and `timestamp`

Key method:
- `checkTransaction()` returns transaction details tuple

### 4.3 Parts-flow contracts

`PartsFlow.sol`
- `id`, `createrID`
- `name`, `partsType`, `batch`, `note`
- `procedures` (uint256[] procedure IDs)
- `formerparts` (uint256[] IDs of former/sub-parts)

Key methods:
- `attachProcedure(procedureID)` appends to `procedures`
- `checkPartsFlow()` returns parts metadata + procedure IDs + former parts IDs

`Procedure.sol`
- `id`
- `partsID` (the processed parts-flow ID)
- `processorID` (address)
- `procedureType` (enum `technical` or `assemble`)
- `operation` and `timestamp`

Key method:
- `checkProcedure()` returns procedure details tuple

### 4.4 Role contracts

`Lender.sol`, `Borrower.sol`, `Manager.sol`

Roles store an `id` (the creator address) and metadata.
They support deletion by setting `valid = false` (but they are not removed from arrays).

`Lender` also keeps:
- `cashFlows` (uint256[] of linked cash flow IDs)

`Borrower` also keeps:
- `cashFlows` (uint256[] of linked cash flow IDs)

### 4.5 Helper/library

`SysFunc.sol`

This is a Solidity `library` used by `TraceabilitySystem`:
- creates `Lender`, `Borrower`, `Manager`
- creates `CashFlow` and `Transaction` bindings (via the system contract logic)
- creates `PartsFlow` and `Procedure`

---

## 5. Traceability Workflow (Conceptual + On-Chain Calls)

This section explains how the system supports traceability using the stored IDs and query APIs.

### 5.1 Fund flow traceability

Recommended workflow:

1. **Create a cash flow**
   - Call `createCashFlow(borrowerID, cashNum)` from the lender address.
   - The system:
     - instantiates `CashFlow` with `id = cashFlowCount`
     - links it to:
       - the caller lender’s `cashFlows` list
       - the given borrower’s `cashFlows` list
     - returns the created cash-flow ID

2. **Create a transaction within that cash flow**
   - Call:
     `createTransaction(cashFlowID, receiverID, cashNum, note)`
   - Internally it:
     - checks that the cash flow has enough balance via `cashFlows[cashFlowID].withdraw(cashNum)`
     - instantiates `Transaction` if success
     - links the transaction back to the cash flow via `attachTransaction`

3. **Trace back / audit**
   - Call `checkCashFlow(cashFlowID)` to retrieve:
     - init balance
     - current balance
     - `Transactions[]` list
   - For each transaction ID in `Transactions[]`, call `checkTransaction(transactionID)` to retrieve:
     - sender/receiver addresses
     - before/after balances
     - transferred amount and note/timestamp

This forms a traceability chain:
`cashFlowID -> transactionIDs -> transaction details`

### 5.2 Parts/component traceability

Recommended workflow:

1. **Create a parts flow**
   - Call `createPartsFlow(name, partsType, batch, note, formerType)`
   - The last parameter is a list of IDs of former/sub-parts.
   - The system returns the new `partsFlowID`.

2. **Create procedures for that parts flow**
   - Call:
     `createProcedure(partsID, ProcedureType procedureType, operation)`
   - It instantiates `Procedure` and attaches the procedure ID to the parts flow’s `procedures[]` list.

3. **Trace / audit**
   - Call `checkPartsFlow(partsID)` to retrieve:
     - creator address, name/type/batch/note
     - `procedures[]` IDs
     - `formerparts[]` IDs
   - For each `procedureID` in `procedures[]`, call `checkProcedure(procedureID)` to retrieve:
     - processing type and operation description
     - processor address and timestamp

This forms a traceability graph:
`partsFlowID -> procedureIDs (+ formerpartsIDs)`

> Note: The contract stores the links as IDs; it does not automatically compute multi-level transitive closure. A frontend or off-chain script can recursively query former parts and procedures if needed.

---

## 6. Deployment and Testing with Truffle

All Truffle configuration is under:
`TraceabilitySystem/`

### 6.1 Build/Compile

From `TraceabilitySystem/`:
```bash
truffle compile
```

### 6.2 Deploy to local development network

The `truffle-config.js` uses:
- network `development` with `host: 127.0.0.1`, `port: 7545` (Ganache default)
- network `develop` with port `8545` (Truffle develop-style)

To deploy (example: Ganache):
```bash
truffle migrate --network development
```

### 6.3 Run tests

From `TraceabilitySystem/`:
```bash
truffle test
```

The repository includes:
- `test/TestTraceabilitySystem.sol`

The test focuses on role creation, cash flow creation, transaction creation, and deletion operations.

---

## 7. API Reference (Main Public Functions)

### 7.1 Role creation and deletion

- `createLender(string account, string passwd, string name)`
- `createBorrower(string account, string passwd, string name)`
- `createManager(string passwd)`
- `deleteLender()`, `deleteBorrower()`, `deleteManager()`

Query:
- `getLenderInfo()` → returns `(address id, string account, string name, uint256[] cashFlows)`
- `getBorrowerInfo()` → returns `(address id, string account, string name, uint256[] cashFlows)`

### 7.2 Fund flow

- `createCashFlow(address borrowerID, uint256 cashNum) returns (uint256 cashFlowID)`
- `createTransaction(uint256 cashFlowID, address receiverID, uint256 cashNum, string note) returns (uint256 transactionID)`
- `checkCashFlow(uint256 cashflowID) view returns (uint256 initBalance, uint256 currBalance, uint256[] transactionIDs)`
- `checkTransaction(uint256 transactionID) view returns (...)` (full tuple defined in `Transaction.sol`)

### 7.3 Parts flow

- `createPartsFlow(string name, string partsType, string batch, string note, uint256[] formerType) returns (uint256 partsFlowID)`
- `createProcedure(uint256 partsID, ProcedureType procedureType, string operation)`
- `checkPartsFlow(uint256 partsID) view returns (address createrID, string name, string partsType, string batch, string note, uint256[] procedureIDs, uint256[] formerparts)`
- `checkProcedure(uint256 procedureID) view returns (uint256 partsID, address processorID, ProcedureType procedureType, string operation, uint256 timestamp)`

---

## 8. Known Limitations / Design Considerations

This is useful for writing a lab report and for future improvement discussion:

1. Passwords (`passwd`) are stored on-chain in plain text and are not used for authentication/authorization in contract logic.
2. Deletion only flips a `valid` flag; role objects remain stored in arrays.
3. The “traceability” linking is implemented via stored ID arrays; full recursive trace graphs are not computed on-chain.
4. Access control (e.g., only manager can create parts/procedures) is not enforced in the shown code; anyone can call creation functions.

---

## 9. What to Submit (Suggested)

If you need to turn this into an assignment submission, consider including:

- A short description of each contract’s data model
- Screenshots or console outputs for:
  - cash flow creation + querying
  - transaction creation + querying
  - parts flow + procedure creation + querying
- A diagram showing:
  - fund-flow link: `CashFlow -> Transaction[]`
  - parts-flow link: `PartsFlow -> procedures[]` and `PartsFlow -> formerparts[]`

---

End of document.

