# DIU OS — Architecture & Decisions

**Last updated**: 15 March 2026 — D-029 PauseController ADR (Gap #2)

---

## Architecture Overview

### Smart Contracts — Arbitrum Stylus (Rust -> WASM -> Native)
- **Framework**: stylus-sdk 0.10.0
- **CLI**: cargo-stylus
- **Network**: Arbitrum One (L2 Ethereum, EVM-compatible)
- **Testnet**: Arbitrum Sepolia
- **Target**: wasm32-unknown-unknown

### Architectural Principle: Empirical Validation First
> Source: Pureur & Bittner "You've Generated Your MVP Using AI" (InfoQ, Feb 2026)

Архитектурные решения без эмпирической валидации — гипотезы, не решения.
**"Caveat Prompter"**: перед каждым новым контрактом — написать ADR с явными trade-offs,
использовать как контекст промпта. Затем реализовывать.
Полная спецификация QARs, fitness functions и тестов: **`QAT.md`**

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

#### AI Persona: "Quantum"
Branded AI tutor — единая точка взаимодействия для всех трёх UX-режимов:
- **Роль**: Physics explainer, simulation guide, research co-pilot
- **Принципы**: Точность прежде доступности; цитирует peer-reviewed источники;
  флагирует неуверенность ("This is an approximation: ..."); никогда не упрощает законы физики
- **Адаптация по режиму**:
  - *Explorer*: Аналогии, доступный язык, пробуждение любопытства
  - *Laboratory*: Структурированные шаги, Сократовские вопросы, подсказки
  - *Research*: Технический уровень, co-author framing, DOI-ссылки
- **Реализация**: Orchestrator LLM с Quantum-branded system prompt + Physics Server validation
- **Safety tier**: Explorer (мягкие) → Laboratory (средние) → Research (строгие guardrails)
- **Идентичность**: "Quantum" — не ChatGPT-wrapper. Personality prompt версионируется как код (ADR D-023)

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

### D-028: VR/AR/MR Architecture — WebXR + Rust VR Stack (06 Mar 2026)
**Context**: 3D квантовые симуляции — идеальный контент для иммерсивного XR.
  "Observe quantum interference inside the double-slit experiment" — compelling нарратив для грантов и медиа.
  Текущий стек (Three.js + WebGL) поддерживает WebXR API без полного переписывания.
  Rust/bevy — зрелый игровой движок (MIT) с нативной поддержкой WASM и WebXR.
**Decision**:
  - **Интерфейс-стандарт**: WebXR API (W3C) — vendor-agnostic, браузерный, охватывает VR + AR + MR
  - **Phase 3 (Q3-Q4 2026)**: WebXR wrapper поверх Three.js симуляций (минимальные изменения)
  - **Phase 4+ (2027)**: bevy 0.16+/WASM для нативного рендеринга (замена Three.js CPU-intensive симуляций)
  - **GPU pipeline**: wgpu (уже в bevy) — cross-platform Metal/Vulkan/WebGPU
  - **Целевые устройства**: Meta Quest 3 (72Hz min), Apple Vision Pro (100Hz), WebXR-браузеры
  - **Rust VR toolkit**: `bevy` (engine), `bevy_oxr` (OpenXR bindings), `wgpu` (GPU)
  - **Stateless invariant сохраняется** (ADR D-021): симуляция params → rendered frame, без side effects

  **VR Performance Targets** (добавлены в QAT.md как Q-21..Q-23):
  - Frame time < 11ms (90 FPS / 72Hz headset requirement)
  - WebXR session enter < 2s (cold start, immersive-vr mode)
  - Physics calculation < 8ms/frame (оставляет 3ms буфер на рендеринг)

  **AR/MR overlay**: Three.js + WebXR AR session для наложения уравнений на реальный мир.
  Особенно эффектно для Laboratory Mode (видеть формулы поверх реального эксперимента).
**Consequence**: Иммерсивный XR — сильный differentiator против конкурентов (Phylo, другие DeSci).
  "First VR quantum physics lab on blockchain" — grant-narrative для Arbitrum Trailblazer + EF.
  Технический риск: bevy/WebXR интеграция требует device-specific testing.
  Phase 3 entry point (WebXR wrapper) — низкий риск, проверяет спрос до полной миграции.
**ADR Decision Matrix**:
  | Option | Risk | Grant Appeal | Timeline |
  |--------|------|-------------|----------|
  | WebXR wrapper (Phase 3) | Low | High | UX-4 |
  | bevy/WASM native (Phase 4) | Medium | Very High | 2027 |
  | Native apps (iOS/Android) | High | Low | Rejected |
**Review**: Phase 3 WebXR POC → decision on bevy migration (Q4 2026).
**Source**: WebXR Device API (W3C, 2025), bevy 0.16 release notes, ADR D-021

### D-029: PauseController — Universal Emergency Stop (15 Mar 2026)
**Context:**
Сейчас только DIUToken имеет механизм `pause()`/`unpause()` (admin-only, `paused: StorageBool`). Остальные 4 контракта (DIURegistry, DIUReputation, DIUAchievements, DIUProgress) не имеют pause вообще. Gap #2: при компрометации backend-ключа или обнаружении критической уязвимости нет способа остановить всю систему одним действием — нужно вручную отправить 5 отдельных транзакций. Ограничения: Stylus SDK 0.10.0 не поддерживает `delegatecall` между WASM контрактами (D-025); один `#[entrypoint]` на WASM binary; `pause()`/`unpause()` — без timelock, должны исполняться мгновенно (D-027).

**Options Considered:**
| Option | Описание | Pros | Cons |
|--------|----------|------|------|
| A | Отдельный PauseController контракт (registry-паттерн): каждый контракт при каждой мутации делает static call `pc.is_globally_paused()` | Единый `pause_all()` = вся система остановлена | +2000–5000 gas на каждую операцию; PauseController сам требует защиты; новый вектор отказа |
| B | Pause в каждом контракте + backend orchestration: 5 последовательных `pause()` вызовов | Нет cross-contract overhead; изолированность | Не атомарно — окно между 1-м и 5-м вызовом; 5 отдельных подписей при ручном управлении |
| C | Shared Rust модуль `pause.rs` + Gnosis Safe batch (D-027): одна Safe 2-of-3 подпись запускает 5 `pause()` в одной batch-транзакции | DRY-код; нет gas overhead; атомарность на уровне Safe batch; leverages D-027 | Единая точка только после настройки Safe (Phase 3) |

**Decision:** Option C — shared Rust module `pause.rs` + Gnosis Safe batch.
- Option A добавляет cross-contract call к каждой операции — неоправданный overhead для образовательной платформы; нарушает принцип "кратчайший путь отказа"
- Option B уже фактически реализован в DIUToken, но без DRY — тот же код дублируется в 5 контрактах
- Option C достигает "единой точки управления" на уровне Safe без runtime overhead; атомарность Safe batch достаточна для threat model (DeSci платформа, не DeFi с миллиардным TVL)

Каждый контракт получает роль **`pause_guardian`** (отдельно от `admin`, Compound-style) — только этот адрес может вызывать `pause()` без timelock. Guardian = адрес Gnosis Safe на mainnet.

**Consequence:**
1. Новый модуль `diu-contracts/src/pause.rs` — `PauseStorage` + `require_not_paused()` + `require_pause_guardian()` helpers
2. DIUToken: рефактор существующего pause → использовать `pause.rs`; добавить роль `pause_guardian`
3. DIURegistry, DIUReputation, DIUAchievements, DIUProgress: добавить `pause.rs` storage + `require_not_paused()` в мутирующие функции + `pause_guardian` роль
4. Storage discipline (D-025): поле `pause_guardian: StorageAddress` добавляется в конец каждого storage struct; `_reserved` слоты не трогаются
5. `pause()` не инкрементирует nonce (D-026) — экстренная операция, не бизнес-логика
6. `pause_guardian` может быть EOA до Phase 3 (testnet — нет реальных средств); Gnosis Safe 2-of-3 обязателен перед mainnet (синхронизировано с D-027)

**Known limitation (testnet период):** До Phase 3 (Gnosis Safe не настроен): экстренная остановка = 5 отдельных backend транзакций. Known limitation testnet периода, не блокирует Phase 2.

**Phase:** Phase 2 — `pause.rs` модуль + рефактор DIUToken. Остальные контракты — перед Phase 3 mainnet.

---

### D-025: No Proxy Through Phase 3; Mainnet Decision Deferred to May 2026 (04 Mar 2026)
**Context**: P-005 analyzed across multiple sources (DeepSeek, ChatGPT, Gemini).
  Stylus SDK 0.10.0 does NOT support `delegatecall` between WASM contracts natively.
  No battle-tested proxy libraries exist for Stylus. No production proxy upgrades in ecosystem.
  UUPS rejected: if `upgradeTo()` has a bug — contract permanently locked (1-person team, unacceptable risk).
  Diamond rejected: extreme complexity, zero Stylus examples.
**Decision**: Deploy without proxy through Phase 2 and Phase 3 testnet.
  Mainnet proxy decision deferred to **May 2026** — evaluate SDK state at that point.
  - If `delegatecall` WASM→WASM appears in SDK: Transparent Proxy (Rust-native).
  - If not: deploy mainnet as immutable contracts; upgrades via new deployment + state migration.
**Strict Storage Discipline (mandatory from Phase 2)**:
  - Single `storage.rs` per contract — no `#[storage]` spread across files
  - Field order FROZEN — never reorder or delete fields; add new fields to END only
  - Reserve slots: `_reserved0..4: StorageU256` in every contract struct
  - Storage version as first field in every contract
**Consequence**: Maximum auditability and simplicity. Immutable mainnet is honest for DeSci users.
**Sources**: DeepSeek (primary — confirmed delegatecall unavailable), ChatGPT, Gemini (Mar 2026)

### D-026: Per-User On-Chain Nonce for Replay Prevention (04 Mar 2026)
**Context**: P-006 analyzed. Two distinct replay surfaces:
  (1) Network-level replay — prevented by Arbitrum's native EOA nonce.
  (2) Logical replay — `addXP(user, 50)` called twice by buggy/compromised backend;
  different tx hash, same business effect. Native Arbitrum nonce does NOT protect against this.
**Decision**: Per-user on-chain nonce in DIUProgress and all future mutating contracts:
  ```rust
  pub user_nonces: StorageMap<Address, StorageU64>,
  // Every mutating call: require(nonce == expected); nonce += 1;
  ```
  Per-action nonce rejected for Phase 2: ~30-40% higher gas, overhead unjustified while backend
  is single-threaded. Revisit in Phase 3 when parallel backend workers appear.
  Phase 3 meta-transactions (EIP-712): per-user nonce already in place, forward-compatible.
**Consequence**: Protection against both network-level and logical replay from Phase 2 onward.
**Sources**: DeepSeek (identified logical replay gap), Gemini/ChatGPT confirmed (Mar 2026)

### D-027: Gnosis Safe + Differentiated Timelock for Admin Functions (04 Mar 2026)
**Context**: P-007 analyzed. Current: single EOA `0x67bB4D...` controls pause/verify/mint
  across all 5 contracts. Pre-audit: severity critical. Timelock alone insufficient —
  does not protect if key is stolen. Adding Timelock at mainnet launch is dangerous:
  first weeks require ability to hotfix critical bugs immediately.
**Decision**:
  | Phase | Solution | Rationale |
  |-------|----------|-----------|
  | Phase 2 testnet | No change | No real funds at risk |
  | Phase 3 pre-mainnet | Gnosis Safe 2-of-3 | Mandatory before mainnet |
  | Phase 3 mainnet stable | Safe 2-of-3 + Timelock | After system proves stable (4-6 weeks) |

  Differentiated Timelock by function:
  | Function | Timelock | Reason |
  |----------|----------|--------|
  | `pause()` / `unpause()` / `blacklist_address()` | ❌ None | Emergency — must be instant |
  | `verify_user()` | ❌ None | Operational, low risk |
  | `add_authorized_address()` | ✅ 48h | New admin = high risk |
  | `mint_tokens()` | ✅ 48h | Token emission |
  | `withdraw_funds()` | ✅ 48h | Asset movement |
  | `upgrade_implementation()` | ✅ 7 days | Critical — maximum delay |

  Safe configuration: 2-of-3 signers (founder hardware wallet + 2 trusted keys).
  Key rotation procedure must be documented before mainnet.
**Consequence**: Eliminates single-point-of-failure on admin keys before mainnet.
**Sources**: DeepSeek (function table), all three LLMs agree on Safe 2-of-3 (Mar 2026)

---

## Solidity -> Stylus Migration Patterns

| # | Solidity Pattern | Stylus Solution |
|---|------------------|-----------------|
| 1 | UUPS Proxy | No proxy Phase 1-2; Transparent Proxy if SDK ready May 2026 (see D-025) |
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
| Upgradability | No proxy Phase 1-2; decision May 2026 (see D-025) |
| Sybil Resistance | ORCID verification in DIURegistry |
| Replay Prevention | Per-user on-chain nonce StorageMap<Address, StorageU64> (see D-026) |

### Pre-Audit Summary (10 Feb 2026)
**Rating**: B+ (internal assessment)
**Интерпретация**: код качественный, логика верная, 171 тест.
Есть известные structural gaps — не критичны на testnet, опасны на mainnet.
Целевой рейтинг перед mainnet: **A** (внешний аудит пройден, все gaps закрыты).

**Critical gaps — трекер закрытия** (полная спецификация в `QAT.md`):

| Gap | Severity | Решение | Target | Статус |
|-----|----------|---------|--------|--------|
| #1 Нет multi-sig для admin | Critical | Gnosis Safe 2-of-3 | Phase 3 mainnet | ❌ Open |
| #2 Нет universal pause | High | PauseController (ADR D-029) | Phase 2 | 📋 ADR ready |
| #3 ORCID centralized | Medium | Verification queue + fallback | Phase 2 | ❌ Open |
| #4 Нет XP rate limiting | Medium | Per-user daily cap в DIUReputation | Phase 2 | ❌ Open |

**Target**: External audit by May 2026 (see P-009).

For detailed security analysis, see `diu-contracts/docs/SECURITY_AUDIT.md`.

---

## Pending Decisions

### ~~P-005: Proxy Pattern for Upgradability~~ → RESOLVED: ADR D-025 (04 Mar 2026)
No proxy through Phase 2-3 testnet. Mainnet decision deferred to May 2026 pending Stylus SDK state.

### ~~P-006: Nonce-Based Replay Prevention~~ → RESOLVED: ADR D-026 (04 Mar 2026)
Per-user on-chain nonce (`StorageMap<Address, StorageU64>`) in DIUProgress and all future contracts.

### ~~P-007: Multi-Sig for Admin Functions~~ → RESOLVED: ADR D-027 (04 Mar 2026)
Gnosis Safe 2-of-3 before mainnet. Differentiated Timelock added after system stabilizes (not at launch).

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
- **QARs, fitness functions, load tests, security gaps tracker**: `QAT.md` ← NEW
- Old Solidity reference: `contracts/` (DEPRECATED, do not modify)
