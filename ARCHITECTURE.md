# DIU OS — Architecture & Decisions

**Last updated**: 17 February 2026

---

## Architecture Overview

### Smart Contracts — Arbitrum Stylus (Rust -> WASM -> Native)
- **Framework**: stylus-sdk 0.10.0
- **CLI**: cargo-stylus
- **Network**: Arbitrum One (L2 Ethereum, EVM-compatible)
- **Testnet**: Arbitrum Sepolia
- **Target**: wasm32-unknown-unknown

### 8-Contract Ecosystem
```
Phase 1 (DEPLOYED):
  DIURegistry --> DIUReputation --> DIUAchievements    DIUToken
  (identity)     (XP/levels)      (NFT badges)        (ERC-20)
                       |                                  ^
                       +----------------------------------+
                         (DIUReputation -> DIUToken.mint)

Phase 2 (planned):
  DIUProgress --> DIUReputation    DIUCrowdfunding --> DIUToken
  (learning)     (addXP)          (funding)           (transfer)

Phase 3 (2027+):
  DIUGovernance <--> DIUStaking
  (proposals)        (locking)
```

### Build System — Feature-Gated
Each contract is behind a Cargo feature flag. Only one `#[entrypoint]` active at a time:
```bash
cargo test                                    # All 139 tests
cargo stylus check --features registry        # Check single contract WASM
```

### Access Control Matrix

| Contract | Public | Backend Only | Admin Only | Cross-Contract |
|----------|--------|-------------|------------|----------------|
| DIURegistry | register, view | -- | verify | -- |
| DIUReputation | view | addXP, recordLogin | -- | DIUToken.mint |
| DIUAchievements | view | mint | -- | -- |
| DIUToken | transfer, approve | mint | pause | -- |
| DIUProgress | view | record | -- | DIUReputation.addXP |
| DIUCrowdfunding | contribute, refund | -- | -- | DIUToken.transfer |

---

## Architecture Decision Records

### D-007: Stylus/Rust over Solidity (3 Feb 2026)
**Context**: Founder has 15+ years Rust, Kirill recommended Stylus, Arbitrum offers 5M ARB grants.
**Decision**: All smart contracts in Rust/Stylus. Old Solidity code (`contracts/`) deprecated.
**Consequence**: Low ecosystem competition, native Rust tooling, but smaller community.

### D-008: Feature-Gated Contracts (10 Feb 2026)
**Context**: Stylus requires single `#[entrypoint]` per WASM binary.
**Decision**: Cargo feature flags per contract, single crate with multiple modules.
**Consequence**: Shared types/errors, unified test suite, single deploy per contract.

### D-009: Manual Access Control via StorageMap (10 Feb 2026)
**Context**: No OpenZeppelin equivalent for Stylus.
**Decision**: `StorageMap<Address, StorageBool>` for roles (owner, admin, authorized).
**Consequence**: Simple, auditable, but must be reimplemented per contract.

### D-010: Non-Upgradable Phase 1 Deployment (15 Feb 2026)
**Context**: Proxy patterns for Stylus are immature; Kirill reviewing options.
**Decision**: Deploy without proxy for Phase 1 testnet.
**Consequence**: Simpler deployment, but redeploy required for contract changes.

### D-011: ERC-721 for Achievements (12 Feb 2026)
**Context**: Badges could be soulbound (non-transferable) or standard NFTs.
**Decision**: Standard ERC-721 for Phase 1 (transferable).
**Consequence**: Users can trade/showcase badges; soulbound option deferred to Phase 2.

### D-012: ERC-20 for Platform Token (13 Feb 2026)
**Context**: Platform needs a fungible token for rewards and future governance.
**Decision**: Standard ERC-20 with restricted minting (backend-only).
**Consequence**: Compatible with DEXes and governance frameworks.

### D-013: Backend-Only Minting (10 Feb 2026)
**Context**: Phase 1 has no on-chain oracle or decentralized verification.
**Decision**: Minting (XP, achievements, tokens) restricted to authorized backend addresses.
**Consequence**: Centralized trust in Phase 1; decentralize in Phase 2+ with oracles/ZK proofs.

### D-014: IPFS for Metadata URIs (12 Feb 2026)
**Context**: NFT metadata needs off-chain storage.
**Decision**: IPFS for Phase 1 (standard, free pinning via Pinata/nft.storage).
**Consequence**: Content-addressed, but pinning required. Arweave under review with Kirill.

---

## Solidity -> Stylus Migration Patterns

| # | Solidity Pattern | Stylus Solution |
|---|------------------|-----------------|
| 1 | UUPS Proxy | Non-upgradable Phase 1; proxy TBD with Kirill |
| 2 | OZ AccessControl | Manual `StorageMap<Address, StorageBool>` per role |
| 3 | Initializable | `StorageBool initialized` + check in `init()` |
| 4 | `bytes(name).length == 0` | `name.is_empty()` |
| 5 | `address(0)` check | `addr == Address::ZERO` |
| 6 | `keccak256(bytes(name))` | `stylus_sdk::crypto::keccak()` |
| 7 | `msg.sender` | `msg::sender()` |
| 8 | Custom errors with params | Rust `Result<T, E>` with enum |
| 9 | calldata optimization | Rust ownership handles automatically |
| 10 | Event indexed params (max 3) | Same limit in Stylus `sol!` events |

---

## Security Patterns

| Pattern | Implementation |
|---------|----------------|
| Access Control | onlyOwner, onlyAuthorized modifiers (manual) |
| Reentrancy | Disabled by default in Stylus |
| Overflow | Rust checked arithmetic (built-in) |
| Timestamp | Block timestamp + tolerance |
| Upgradability | TBD with Kirill (see P-005) |
| Sybil Resistance | ORCID verification in DIURegistry |
| Replay Prevention | Nonce-based (see P-006) |

For detailed security analysis, see `diu-contracts/docs/SECURITY_AUDIT.md`.

---

## Pending Decisions (Awaiting Kirill)

### P-005: Proxy Pattern for Upgradability
**Question**: Should Phase 2 contracts use a proxy pattern? Which one (UUPS, transparent, diamond)?
**Impact**: Deployment strategy, storage layout constraints, gas costs.

### P-006: Nonce-Based Replay Prevention
**Question**: Best practices for nonces in Stylus? Per-user vs per-action nonces?
**Impact**: XP system security, daily login integrity.

### P-007: Multi-Sig for Admin Functions
**Question**: Should admin functions require multi-sig in Phase 2?
**Impact**: Centralization risk, operational complexity.

### P-008: IPFS vs Arweave for NFT Metadata
**Question**: Permanent storage (Arweave) vs pinned storage (IPFS)?
**Impact**: Cost, permanence, tooling availability.

### P-009: Audit Firm Selection
**Question**: Which firms have Rust/WASM smart contract experience?
**Impact**: Audit quality, cost, timeline for mainnet.

---

## Detailed Documentation Pointers
- Contract APIs, storage layouts: `diu-contracts/README.md`
- Security audit notes: `diu-contracts/docs/SECURITY_AUDIT.md`
- Business logic analysis: `diu-contracts/docs/BUSINESS_LOGIC_ANALYSIS.md`
- Frontend architecture: `physics-tutorial/docs/ARCHITECTURE.md`
- Old Solidity reference: `contracts/` (DEPRECATED, do not modify)
