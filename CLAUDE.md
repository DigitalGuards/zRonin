# zRonin - Privacy Layer for QRL Zond

## Project Overview

zRonin is a privacy-preserving smart contract system for QRL Zond, adapted from RAILGUN with key cryptographic upgrades for post-quantum security.

### Key Differences from RAILGUN
| Aspect | RAILGUN | zRonin |
|--------|---------|--------|
| Token | $RAIL | $ZRON |
| Proof System | zk-SNARKs (Groth16) | zk-STARKs |
| Signatures | EdDSA (Babyjubjub) | Dilithium (ML-DSA) |
| L1 Network | Ethereum/Polygon/etc | QRL Zond |
| Trusted Setup | Required | Not required |
| Smart Contracts | Solidity (.sol) | Hyperion (.hyp) |
| Token Standards | ERC20/ERC721/ERC1155 | ZRC20/ZRC721/ZRC1155 

---

## Hyperion Smart Contracts

Zond uses **Hyperion** instead of Solidity. Hyperion is syntactically nearly identical to Solidity but compiles to ZVM bytecode.

### Key Differences from Solidity
- File extension: `.hyp` (not `.sol`)
- Pragma: `pragma hyperion >=0.8.0` (not `pragma solidity`)
- Compiler: `hypc` or `hypcjs` (not `solc`)
- Target VM: ZVM (not EVM)

### Token Standards
- **ZRC20** - Fungible tokens (equivalent to ERC20)
- **ZRC721** - NFTs (equivalent to ERC721)
- **ZRC1155** - Multi-tokens (equivalent to ERC1155)

### Contract Files
- `src/contracts/src/interfaces/IStarkVerifier.hyp` - STARK proof verification interface
- `src/contracts/src/interfaces/ITokenVault.hyp` - Token vault interface
- `src/contracts/src/interfaces/IMerkleAccumulator.hyp` - Merkle tree interface
- `src/contracts/src/libraries/PoseidonT3.hyp` - Poseidon hash library
- `src/contracts/src/core/StarkVerifier.hyp` - STARK verifier implementation
- `src/contracts/src/core/MerkleAccumulator.hyp` - Merkle tree implementation
- `src/contracts/src/core/TokenVault.hyp` - Token vault implementation
- `src/contracts/src/ZronToken.hyp` - ZRON governance token

---

## Architecture Reference (from RAILGUN)

### Circuit Variants (reimplemented for STARKs):
- `Transfer1x2` - 1 input, 2 outputs
- `Transfer1x3` - 1 input, 3 outputs
- `Transfer2x2` - 2 inputs, 2 outputs
- `Transfer2x3` - 2 inputs, 3 outputs
- `Transfer8x2` - 8 inputs, 2 outputs
- POI Mini (3x3), POI Full (13x13)

### Merkle Trees
- **Depth**: 16 levels
- **Max Items**: 65,536 (2^16)
- **Hash**: Poseidon
- **Zero Value**: `keccak256('Railgun') % SNARK_PRIME`

### Key Cryptographic Components

**Poseidon Hash**
**Dilithium Signatures** (replaces EdDSA)

---

## QRL Zond Integration

### Dilithium Signatures
**Key Characteristics** (Dilithium5 / ML-DSA-87):
- Public key: 2592 bytes
- Secret key: 4864 bytes
- Signature: 4595 bytes
- Post-quantum secure (NIST FIPS 204)

### Hyperion Compiler

### Network Configuration

## Key Files Reference

### zRonin Implementation (src/)
- `crypto/dilithium/` - Dilithium5 wrapper using @theqrl/dilithium5
- `crypto/stark/` - STARK prover/verifier (field, AIR, FRI, prover, verifier)
- `engine/` - Transaction building, Merkle tree, wallet management
- `contracts/` - Hyperion smart contracts (.hyp files)
- `shared/` - Shared types and constants

### QRL Zond Reference (../../theQRL/)
- `go-zond/` - Zond client implementation
- `hyperion/` - Hyperion compiler (C++)
- `hypc-js/` - Hyperion JavaScript wrapper
- `go-qrllib/crypto/dilithium/` - Dilithium library

### STARK Reference (predicates/)
- `winterfell/` - Rust STARK framework (audited) - **PRIMARY REFERENCE**
- `ethSTARK/` - C++ reference - **NOT SUITABLE** (wrong target, 80-bit security, non-ZK)

---

## Security Notes

### STARK vs SNARK Tradeoffs
- **Pros**: No trusted setup, post-quantum secure, transparent
- **Cons**: Larger proofs (~100KB vs ~200B), higher verification cost
- **Mitigation**: Recursive proofs, circuit optimization

### Security Strategy
1. Use **Winterfell** (Rust, audited by Polygon) as primary reference
2. Build custom STARK implementation for Zond's ZVM
3. Plan formal security audit before production
4. **No unaudited research-grade code in production**

---

## Related Documentation
- RAILGUN Docs: https://docs.railgun.org/
- QRL Zond: https://docs.theqrl.org/
- Winterfell: https://github.com/novifinancial/winterfell
- Hyperion: https://github.com/theQRL/hyperion
