# DIU OS — Architecture & Decisions

**Last updated**: 02 March 2026 — D-024 S3-native event streaming ADR

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
Phase 1 (DEPLOYED, 19 Feb 2026):
  DIURegistry --> DIUReputation --> DIUAchievements    DIUToken
  (identity)     (XP/levels)      (NFT badges)        (ERC-20)
                       |                                  ^
                       +----------------------------------+
                         (DIUReputation -> DIUToken.mint)

Phase 2 (IN PROGRESS):
  DIUProgress --> DIUReputation    DIUCrowdfunding --> DIUToken
  (learning ✅)  (addXP)          (funding)           (transfer)

Phase 3 (2027+):
  DIUGovernance <--> DIUStaking
  (proposals)        (locking)
```

### Build System — Feature-Gated
Each contract is behind a Cargo feature flag. Only one `#[entrypoint]` active at a time:
```bash
cargo test                                    # All 171 tests
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

## Deployed Contracts (Arbitrum Sepolia)

| Contract | Address | Deployed | Tests | WASM |
|----------|---------|----------|-------|------|
| DIURegistry | `0x49e1b11e1037e74113a7c0ccc41e3042d4691018` | 19 Feb 2026 | 28 | 21.3KB |
| DIUReputation | `0x8740f9d110133ff5efa0fb562e62ab92a466cdc5` | 19 Feb 2026 | 36 | 20.0KB |
| DIUAchievements | `0x1a9783ba7966c0e7299af7ee2228e19028d8ea7e` | 19 Feb 2026 | 34 | 23.2KB |
| DIUToken | `0xbbd9a558c049482f1be45399fec4a4c9dc1c810e` | 19 Feb 2026 | 49 | 17.7KB |
| DIUProgress | `0xb1c4edc73aae322f62cda57f84f303761ca3e347` | 27 Feb 2026 | 24 | 19.0KB |

**Deployer**: `0x67bB4D1895D9A736F9e6076529B468ba05aeD150`
**Total tests**: 171
**DIUProgress init params**: `reputation=0x8740...cdc5`, `max_experiment_id=10`, `max_module_id=5`

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

### D-019: Simulation-First Research Loop (27 Feb 2026)
**Context**: Kevin Weil (OpenAI VP) описывает научную петлю будущего: model→simulation→real world→model.
  Наш Physics Server — первый уровень этой петли. Архитектурно это уже заложено в D-018, но не формализовано.
**Decision**: DIU OS строит двухуровневую Research Loop:
  - Tight loop: AI↔Simulation (<2s) — для интерактивного обучения
  - Extended loop: AI↔Simulation↔Data Export — для серьёзных исследований (Research Mode)
  Data export API включается в scope DIUProgress (Phase 2), не откладывается на Phase 3.
**Consequence**: DIUProgress должен поддерживать экспорт результатов симуляций (JSON/CSV).
  Это усиливает ценностное предложение для исследователей и дифференцирует от конкурентов.
**Source**: Kevin Weil, a16z speedrun, 26 Feb 2026

### D-020: Capability Window — Build Infrastructure Now (27 Feb 2026)
**Context**: AI capabilities идут по кривой 0%→10%→80% за 6–12 месяцев (Kevin Weil, a16z speedrun).
  MCP Mesh сейчас на 10% готовности. Через 6–12 месяцев AI будет отлично работать с нашей инфраструктурой
  — но только если инфраструктура уже построена.
**Decision**: MCP Server scaffolding поднять с B-5 на B-3 в backend roadmap.
  Строить MCP Physics Server stub параллельно с SIWE auth и PostgreSQL (не после).
  Ждать "готового UX" перед началом MCP = упустить capability window.
**Consequence**: Пересмотрен порядок backend задач (см. ROADMAP.md Backend Roadmap).
  Технический долг — минимальный; stub не требует полной реализации, только API-контракт.
**Source**: Kevin Weil, a16z speedrun, 26 Feb 2026

### D-021: Stateless WASM Simulations (27 Feb 2026)
**Context**: Physics calculations должны масштабироваться горизонтально (см. D-019 extended loop).
  Текущие симуляции реализованы на JS в браузере; это создаёт bottleneck для server-side вычислений.
**Decision**: CPU-intensive physics core переписать с JS на Rust→WASM.
  Начать с double-slit experiment (наиболее computationally intensive).
  Все симуляции должны быть stateless: входные параметры → выходные данные, без side effects.
  Stateless = горизонтальное масштабирование без shared state.
**Consequence**: Усиливает "Rust everywhere" нарратив для грантовых заявок (Stylus Sprint, a16z).
  Double-slit WASM rewrite — Phase 2 deliverable (UX-2/UX-3 period).
**Source**: Kevin Weil, a16z speedrun, 26 Feb 2026

### D-022: Ensemble Model Routing (27 Feb 2026)
**Context**: Kevin Weil прямо указывает — слишком мало стартапов используют ensemble моделей.
  "A lot of things today turn out best when you use an ensemble of models."
  D-018 уже описывает right-sizing, но не фиксирует официальный паттерн routing.
**Decision**: Официально фиксируем ensemble архитектуру:
  - Orchestrator LLM — маршрутизация и планирование (полный LLM)
  - IntentClassifier SLM — быстрая классификация намерений (<100ms, дешёво)
  - PhysicsValidator — специализированный инструмент проверки уравнений
  - KnowledgeRAG — Qdrant + embeddings для curriculum context
  Никогда не использовать один гигантский промпт для всего.
**Consequence**: Ensemble описывается в grant applications как технический differentiator.
  Ссылка: "Следует best practices, рекомендованным OpenAI VP (Kevin Weil, Feb 2026)."
**Source**: Kevin Weil, a16z speedrun, 26 Feb 2026

### D-023: Progressive Capability Release via Versioned MCP Interface (27 Feb 2026)
**Context**: Новые AI capabilities появляются каждый месяц (Weil: "And in another month, that happens again").
  Если Orchestrator жёстко связан с конкретными MCP Server реализациями — каждый апгрейд = breaking change.
**Decision**: Каждый MCP Server имеет versioned interface как contractual API:
  `mcp://physics.diu-os/v1/simulate`, `mcp://knowledge.diu-os/v2/query`.
  Новая capability = новая версия MCP Server (или новый сервер), без изменения Orchestrator.
  Orchestrator знает только о контрактах, не о реализациях.
**Consequence**: Plugin-архитектура для AI слоя. Соответствует "scientific LEGO blocks" нарративу.
  Новые модели (GPT-5, Claude 4, etc.) интегрируются заменой одного сервера, не рефакторингом системы.
**Source**: Kevin Weil, a16z speedrun, 26 Feb 2026

### D-024: S3-Native Event Streaming for Phase 3 Backend (02 Mar 2026)
**Context**: Phase 3 (500+ users, Arbitrum mainnet) потребует event-driven pipeline для:
  on-chain событий (XP, NFT mint, login streaks) → аналитика, ML-фичи для MCP серверов,
  CDC из PostgreSQL в реальном времени. Kafka — избыточен для стартапа: broker fleet,
  ZooKeeper/KRaft, replication, дорогой cross-AZ трафик. Исследован StreamHouse (Mar 2026)
  как S3-native Kafka-альтернатива на Rust (MIT, hobby-проект на ранней стадии).
**Decision**: Принять S3-native streaming как целевую архитектуру для Phase 3 backend.
  Конкретный инструмент не фиксируется до Phase 3 — оценить зрелость в Q3 2026:
  StreamHouse, WarpStream или AutoMQ. Требования к выбору: Rust-совместимость,
  PostgreSQL для метаданных (совместимо с B-3), Kafka-compatible API.
  Phase 1-2 (Axum + PostgreSQL) достаточен при <500 users — не усложнять раньше времени.
**Consequence**: Stateless агенты + S3 заменяют broker fleet → ~80% экономия на инфраструктуре.
  Архитектурно совместимо с D-018/D-020 (MCP серверы питаются из Progress/Physics event streams).
  Важно: event schema проектировать в Phase 2, чтобы избежать рефакторинга при миграции.
**Review**: Q3 2026, после выхода на mainnet.
**Source**: https://streamhouse.app, https://github.com/gbram1/streamhouse (MIT)

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
