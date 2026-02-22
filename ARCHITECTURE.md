# DIU OS — Architecture & Decisions

**Last updated**: 22 February 2026 — Phase 1 COMPLETE (redeployed 19 Feb with `initialize()` pattern)

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
cargo test                                    # All 147 tests
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

## Platform Architecture

### 3-Mode UX (see ADR D-015)
```
Explorer Mode (sandbox) → Laboratory Mode (guided) → Research Mode (AI + export)
```
- **Explorer**: Free-form simulation sandbox, no login required
- **Laboratory**: Structured experiments with progress tracking (requires auth)
- **Research**: AI assistant + paper export + collaboration (requires token)

### AI Assistant Architecture
MCP-based multi-server design:
- **Physics Server** — simulation control, equation validation
- **Knowledge Server** — RAG pipeline (Qdrant vector DB), curriculum context
- **Progress Server** — learning analytics, adaptive recommendations
- **Targets**: <2s response latency, >90% physics accuracy

#### Принципы проектирования MCP-серверов (Phase 2-3)
> Source: _workspace/references/from-prompts-to-platforms-part1.pdf (Katarya, Feb 2026)

**Messaging — потоковая передача, не request-response:**
- Progressive feedback обязателен: пока Physics Server считает симуляцию, агент
  выдаёт interim-ответы ("Вычисляю траекторию..."). Критично для <2s perceived latency.
- Все взаимодействия User↔Agent и Agent↔Agent персистируются в mailbox-структурах
  (granular access control, совместимо с D-016).

**Memory — rolling window + episodic persistence:**
- Учебные сессии растягиваются на недели. Progress Server обязан персистировать
  историю без полной ре-контекстуализации при следующем входе.
- Rolling window: ~10 последних turns в активном контексте. Старый контекст
  вносит шум без улучшения качества.
- RAG grounding: Knowledge Server (Qdrant) дополняет знания модели актуальным
  прогрессом пользователя в реальном времени.

**Right-sizing models — не один LLM для всего:**
- Intent classification → SLM (быстро, детерминировано, дёшево)
- Equation validation → Physics Server (специализированный инструмент)
- Генерация объяснений → полный LLM (только здесь нужна generative breadth)
- Embedding generation, feature precomputation → асинхронные offline pipelines

**Prompt management — промпты как versioned code:**
- Каждый промпт привязан к версии модели. Апгрейд модели ≠ лучший результат автоматически.
- Динамическая инъекция: прогресс пользователя + текущий модуль + результаты
  симуляции подставляются в шаблон в runtime.
- Промпты хранятся отдельно от кода — быстрая итерация без redeploy.

**Agent Mesh:**
- Orchestrator маршрутизирует задачи к специализированным sub-агентам.
- Стандартизированный A2A протокол между Physics/Knowledge/Progress серверами.
- Никаких прямых связей между серверами — только через Orchestrator.

**Safety Guardrails (multi-stage):**
- Input: блокировать prompt injection (особенно Research Mode — открытый ввод)
- Planning: валидировать план агента ДО выполнения действий
- Output: фильтровать физически некорректные утверждения (>90% accuracy target)
- Trust tiers: Explorer (мягкие) → Laboratory (средние) → Research (строгие)

### Design System
- **Colors**: quantum-blue (`#0066FF`), wave-purple (`#6B46C1`)
- **Typography**: Inter (UI), JetBrains Mono (code/equations), KaTeX (math rendering)
- **Personas**: Student, Researcher, Educator, Curious Explorer
- **Source**: `_workspace/Local-Working-Docs/DeSci/DIU Phase 0 MVP/UI-UX Requirements/`

### User Flow References
88+ user cases across 10 categories documented:
- Auth flow, Experiment flow, AI interaction, Progress tracking, NFT mint
- **Source**: `_workspace/Local-Working-Docs/DeSci/Dialog/07-02-26/Завершённая документация/`

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

### D-010: Non-Upgradable Phase 1 Deployment (19 Feb 2026)
**Context**: Proxy patterns for Stylus are immature; Kirill reviewing options (P-005).
**Decision**: Deploy without proxy for Phase 1 testnet. Redeployed 19 Feb 2026 with `initialize()` pattern (constructor bug fix — `msg::sender()` returns `0x00` in Stylus WASM).
**Consequence**: Simpler deployment, but redeploy required for contract changes. Proxy decision deferred to Phase 2.

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

### D-015: 3-Mode UX Architecture (5 Feb 2026)
**Context**: Platform needs to serve both casual visitors and serious researchers.
**Decision**: Three progressive access modes — Explorer (sandbox), Laboratory (guided), Research (AI + export).
**Consequence**: Clear upgrade path for users; each mode maps to a different auth/token requirement.

### D-016: Web3 as Infrastructure, not Product (31 Jan 2026)
**Context**: DeSci projects risk alienating mainstream users with "blockchain-first" positioning.
**Decision**: Core product = simulations + AI (80%), Web3 = infrastructure layer (20%). Never position as "blockchain platform."
**Consequence**: Lower barrier to entry; Web3 benefits (credentials, tokens) without Web3 friction.

### D-017: Tech Stack Evolution (historical)
**Context**: Project migrated through three blockchain stacks.
**Decision**: Casper Network (2024) → Solidity/Foundry (Jan 2026) → Rust/Stylus (Feb 2026).
**Consequence**: Explains deprecated `contracts/` directory. Current stack is final — Rust/Stylus only.

### D-018: MCP Agent Mesh Architecture (18 Feb 2026)
**Context**: AI Assistant нужен для Research Mode. Требования: <2s latency, >90% physics accuracy.
**Decision**: Orchestrator-based mesh: один Orchestrator + три специализированных MCP Server.
  Нет прямых A2A связей между серверами. Right-sizing: SLM для classification, LLM для generation.
  Rolling window ~10 turns. Промпты — versioned assets, не строки в коде.
**Consequence**: Масштабируется и изолировано, но требует стандартизированного A2A протокола.
**Source**: _workspace/references/from-prompts-to-platforms-part1.pdf

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

### Pre-Audit Summary (10 Feb 2026)
**Rating**: B+ (internal assessment)
**Critical gaps before mainnet**:
1. No multi-sig for admin functions (severity: critical)
2. No universal pause mechanism across all contracts
3. Centralized ORCID verification (single point of failure)
4. No rate limiting on XP/reputation operations

**Target**: External audit by May 2026 (see P-009).

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

### P-010: Dual-Token Economy ($DIU + $SCI)
**Context**: Current DIUToken is interim single ERC-20. Long-term design envisions two tokens.
**Proposed**: $DIU (governance, 1B fixed supply) + $SCI (utility, 500M + 2%/yr inflation).
**Timeline**: Phase 3+ (2027). Current DIUToken may become $DIU or be migrated.
**Impact**: Tokenomics design, migration strategy, regulatory implications.

---

## Detailed Documentation Pointers
- Contract APIs, storage layouts: `diu-contracts/README.md`
- Security audit notes: `diu-contracts/docs/SECURITY_AUDIT.md`
- Business logic analysis: `diu-contracts/docs/BUSINESS_LOGIC_ANALYSIS.md`
- Frontend architecture: `physics-tutorial/docs/ARCHITECTURE.md`
- Old Solidity reference: `contracts/` (DEPRECATED, do not modify)
