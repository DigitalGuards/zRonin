# zRonin Security Audit Preparation

This document outlines the security architecture, threat model, and areas of focus for security auditors reviewing zRonin.

## Overview

zRonin is a privacy-preserving smart contract system for QRL Zond, using:
- **STARK proofs** for zero-knowledge privacy
- **Dilithium5 signatures** for post-quantum authentication
- **Poseidon hash** for efficient in-circuit hashing
- **Merkle trees** for commitment tracking

## Architecture

### Contract Structure

```
TokenVault (Main Entry Point)
├── StarkVerifier (Proof Verification)
├── MerkleAccumulator (Commitment Tracking)
├── RelayAdapt (Relay Integration)
└── ZronToken (Governance Token)
```

### Data Flow

1. **Shield (Deposit)**:
   - User deposits tokens to TokenVault
   - TokenVault creates commitment (hash of note)
   - MerkleAccumulator inserts commitment
   - User receives encrypted note

2. **Transfer (Private)**:
   - User creates STARK proof of note ownership
   - Proof includes nullifiers (to prevent double-spend)
   - New commitments added for outputs
   - Old nullifiers marked as spent

3. **Unshield (Withdraw)**:
   - User creates STARK proof with withdrawal request
   - TokenVault verifies proof
   - Tokens released to recipient

## Cryptographic Assumptions

### STARK System
- **Field**: Goldilocks prime (2^64 - 2^32 + 1)
- **Hash**: SHA3-256 for Merkle trees, Poseidon for circuits
- **FRI**: Folding factor 2, 128-bit security target
- **Fiat-Shamir**: Transcript-based challenge derivation

### Dilithium5 (ML-DSA-87)
- **Public Key**: 2592 bytes
- **Secret Key**: 4864 bytes
- **Signature**: 4595 bytes
- **Security**: NIST Level 5 (256-bit classical, 128-bit quantum)

### Poseidon Hash
- **Parameters**: t=3, rounds based on security level
- **Field**: SNARK-friendly prime
- **Security**: 128-bit collision resistance

## Threat Model

### In-Scope Threats

1. **Double-Spending**
   - Threat: Spending same note twice
   - Mitigation: Nullifier tracking on-chain

2. **Privacy Leakage**
   - Threat: Transaction linkability
   - Mitigation: STARK zero-knowledge proofs

3. **Proof Forgery**
   - Threat: Creating valid proofs without valid inputs
   - Mitigation: Soundness of STARK system

4. **Front-Running**
   - Threat: MEV extraction from privacy transactions
   - Mitigation: RelayAdapt integration

5. **Key Extraction**
   - Threat: Recovering spending keys from public data
   - Mitigation: Dilithium5 post-quantum security

### Out-of-Scope

1. Physical side-channel attacks
2. Social engineering
3. Zond network consensus attacks
4. Key management by end users

## Areas for Audit Focus

### Smart Contracts (.hyp files)

| File | Priority | Focus Areas |
|------|----------|-------------|
| TokenVault.hyp | Critical | Deposit/withdrawal logic, proof verification calls |
| StarkVerifier.hyp | Critical | Proof validation logic |
| MerkleAccumulator.hyp | High | Tree insertion, root computation |
| RelayAdapt.hyp | High | Relay integration, swap logic |
| ZronToken.hyp | Medium | Token economics, vesting |

### TypeScript Cryptography

| Module | Priority | Focus Areas |
|--------|----------|-------------|
| src/crypto/stark/ | Critical | Proof generation, FRI protocol |
| src/crypto/dilithium/ | Critical | Key generation, signing |
| src/engine/ | High | Transaction building, note encryption |

### Key Audit Questions

1. **Nullifier Uniqueness**: Can two different notes produce the same nullifier?
2. **Commitment Binding**: Is the commitment scheme binding?
3. **Proof Soundness**: What is the concrete soundness error of the STARK system?
4. **Merkle Security**: Are Merkle proofs validated correctly?
5. **Gas Griefing**: Can users cause excessive gas consumption?
6. **Reentrancy**: Are there reentrancy vulnerabilities in deposit/withdraw?
7. **Integer Overflow**: Are arithmetic operations safe?

## Test Coverage

### Unit Tests
- Dilithium: 23 tests (key gen, sign/verify, address derivation)
- STARK: 24 tests (field arithmetic, FRI, prover/verifier)
- Merkle: 26 tests (tree operations, proofs)
- Poseidon: 17+ tests (hash operations)
- Notes: 27 tests (encryption, selection)
- Circuits: 23 tests (AIR constraints)

### Integration Tests
- Contract deployment verification
- Token operations
- Shield/unshield flow simulation

## Gas Analysis

| Operation | Gas Cost |
|-----------|----------|
| ZRC20 Transfer | ~52,000 |
| ZRC20 Approve | ~46,000 |
| Deploy TokenVault | ~1,486,000 |
| Deploy MerkleAccumulator | ~3,485,000 |
| Deploy StarkVerifier | ~957,000 |

## Known Limitations

1. **Proof Size**: STARK proofs are larger than SNARKs (~100KB vs ~200B)
2. **Verification Cost**: On-chain verification is more expensive
3. **Testnet**: Not yet deployed to testnet

## Recommendations for Auditors

1. **Focus on cryptographic correctness** - particularly FRI and Fiat-Shamir
2. **Review nullifier derivation** - critical for double-spend prevention
3. **Analyze gas consumption** - look for DoS vectors
4. **Check access controls** - admin functions, pause mechanisms
5. **Verify Poseidon parameters** - round numbers, constants

## Contact

For security-related inquiries during the audit, contact the zRonin team through secure channels.

---

Document Version: 1.0
Last Updated: 2025-12-14
