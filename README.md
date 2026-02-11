# BridgeX - Lightweight DAO-Governed Cross-Chain Bridge

## Overview

**BridgeX** is a Clarity v2 reference implementation of a lightweight, DAO-governed cross-chain bridge protocol on Stacks. It enables users to lock STX assets and mint wrapped tokens on target chains, with validator staking, slashing, fee-sharing, and quorum-based confirmation mechanisms.

** WARNING:** This is a reference implementation. It requires comprehensive security audits, Clarinet unit tests, and production hardening before mainnet deployment.

---

## Features

###  Core Bridge Functionality
- **Lock/Mint Mechanism:** Users lock STX on Stacks to mint wrapped tokens on target chains
- **Burn/Release Mechanism:** Users burn wrapped tokens on target chains to release locked STX
- **Cross-Chain Request Tracking:** Maintains pending lock and release requests with execution status

###  Validator Infrastructure
- **Validator Staking:** Validators register with STX and build stake over time
- **Stake-Weighted Quorum:** Confirmation requirements based on configurable percentage of total validator stake
- **Jailing & Slashing:** Configurable slashing penalties for misbehavior (default 20%)
- **Confirmation Tracking:** Per-validator confirmation status for each cross-chain request

###  Fee & Treasury Management
- **Configurable Fees:** Basis points (bps) fee on bridge operations (default 0.5%)
- **Fee Sharing:** Infrastructure for distributing fees to validators and treasury
- **Treasury Account:** Designated account for protocol revenue

###  Admin Controls
- **Multi-Admin System:** Owner + configurable admin set
- **Parameter Management:**
  - Minimum validator stake
  - Fee basis points
  - Validator quorum threshold
  - Slashing percentage

###  Wrapped Token Ledger
- **Simple Ledger:** Non-SIP-010 implementation tracking wrapped token balances
- **Total Supply Tracking:** Maintains total wrapped tokens in circulation

---

## Contract API

### Admin Functions

```clarity
(set-admin-status (who principal) (active bool)) -> response<bool, uint>
(set-min-validator-stake (v uint)) -> response<bool, uint>
(set-fee-bps (v uint)) -> response<bool, uint>
(set-validator-quorum-bps (v uint)) -> response<bool, uint>
(set-slash-bps (v uint)) -> response<bool, uint>
```

### Validator Functions

```clarity
(register-validator) -> response<bool, uint>
(unregister-validator (amount uint)) -> response<bool, uint>
```

### User Bridge Functions

```clarity
(lock-assets (target-chain (string-ascii 32)) (target-address (string-ascii 128)) (amount uint)) -> response<bool, uint>
(confirm-lock (request-id uint)) -> response<bool, uint>
(finalize-lock (request-id uint)) -> response<bool, uint>
```

### Read-Only Functions

```clarity
(is-admin (p principal)) -> bool
```

---

## Data Structures

### Maps

| Map | Purpose |
|-----|---------|
| `admins` | Admin set tracking |
| `validators` | Validator stakes, active status, jailing |
| `wrapped-balances` | User wrapped token balances |
| `lock-requests` | Pending lock/mint requests |
| `release-requests` | Pending burn/release requests |
| `confirmations` | Per-validator confirmations per request |

### Constants

| Constant | Default | Purpose |
|----------|---------|---------|
| `min-validator-stake` | 1,000,000 µSTX (0.01 STX) | Minimum stake to register |
| `fee-bps` | 50 (0.5%) | Bridge operation fee |
| `validator-quorum-bps` | 5000 (50%) | Required stake % for finalization |
| `slash-bps` | 2000 (20%) | Slashing penalty percentage |

---

## Error Codes

| Code | Value | Meaning |
|------|-------|---------|
| `ERR-NOT-AUTH` | 100 | Unauthorized caller |
| `ERR-NOT-FOUND` | 101 | Request/validator not found |
| `ERR-BAD-STATE` | 102 | Invalid contract state |
| `ERR-BAD-INPUT` | 103 | Invalid input parameters |
| `ERR-INSUFFICIENT` | 104 | Insufficient balance |
| `ERR-ALREADY` | 105 | Request already processed |
| `ERR-NO-PENDING` | 106 | No pending request |
| `ERR-NO-VOTE` | 107 | Insufficient confirmations |
| `ERR-NO-BALANCE` | 108 | No balance to withdraw |
| `ERR-LOW-STAKE` | 109 | Stake below minimum |
| `ERR-NO-EVIDENCE` | 110 | Missing proof/evidence |

---

## Workflow Example

### Lock & Mint Flow

1. **User initiates lock:** Calls `lock-assets` with target chain, address, and amount
2. **Validators confirm:** Validators call `confirm-lock` to validate the request
3. **Finalization:** Once quorum is reached, `finalize-lock` executes and triggers cross-chain mint
4. **Result:** STX locked on Stacks, wrapped tokens minted on target chain

### Burn & Release Flow

1. **User initiates burn:** Calls contract on target chain to burn wrapped tokens
2. **Proof submitted:** Release request created with source chain proof
3. **Validators confirm:** Validators call confirm function to validate
4. **Finalization:** Once quorum met, locked STX released to recipient

---

## Development & Testing

### Prerequisites
- Clarinet CLI
- Node.js 18+
- Stacks wallet (for testnet)

### Setup

```bash
cd c:\Users\USER\Desktop\STACKS\USED\SEPTEMBER\bitbridge
clarinet new
clarinet contract add contracts/bitbridge.clar
```

### Running Tests

```bash
clarinet test
```

### Local Deployment

```bash
clarinet console
# Deploy contract
clarinet deploy
```

### Testnet Deployment

```bash
clarinet deploy --network testnet
```

---

## Security Considerations

### Audit Requirements
-  Validator stake correctness and slashing logic
-  Quorum calculation and finalization gates
-  Wrapped token ledger consistency
-  Admin control access patterns
-  Cross-chain request atomicity

### Known Limitations
- No SIP-010 token standard (custom ledger)
- Simple linear quorum (not weighted by reputation)
- No timelock on admin actions
- No emergency pause mechanism
- Requires off-chain validator infrastructure

### Recommended Enhancements
- Implement SIP-010 for wrapped token standard
- Add timelock delay for critical admin functions
- Implement emergency pause/recovery mechanisms
- Add reputation/slashing history tracking
- Integrate with external oracle for source-chain verification

---

## Configuration

Edit constants at the top of bitbridge.clar:

```clarity
(define-constant ERR-NOT-AUTH        (err u100))
;; ... error codes ...

(define-data-var min-validator-stake uint u1000000)    ;; Adjust minimum stake
(define-data-var fee-bps uint u50)                     ;; Adjust fee percentage
(define-data-var validator-quorum-bps uint u5000)      ;; Adjust quorum threshold
(define-data-var slash-bps uint u2000)                 ;; Adjust slashing penalty
```

---

---

**Version:** 1.0.0 (Reference)  
**Last Updated:** February 11, 2026
