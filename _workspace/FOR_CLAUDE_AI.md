# FOR_CLAUDE_AI.md — DIU OS Full Context
# Generated: 27 Feb 2026
# Contents: CLAUDE.md + ROADMAP.md + ARCHITECTURE.md (ADR section only) + WEIL_VALIDATION.md + diu-contracts git log

---

# PART 1: CLAUDE.md

# DIU OS — Project Context for Claude Code

## Project
DIU OS is a decentralized Scientific Operating System. Phase 0 MVP: quantum physics education with 3D simulations, AI tutoring, Web3.
- **Founder**: Bakhtiyor Ruzimatov (Barust) — 15+ years Rust, AI/ML, distributed systems
- **Advisor**: Kirill Taran — Web3 Security (joined 03 Feb 2026)
- **Live MVP**: https://physics.diu-os.org (v0.15.13) | **GitHub**: https://github.com/diu-os

## Critical Rules
- **NEVER Solidity/Foundry/Hardhat** — Rust/Stylus only. `contracts/` is DEPRECATED.
- **All Stylus contracts go in `diu-contracts/`** (NOT old `contracts/`)
- **FunDeSci is EXCLUDED** from grant strategies
- Scientific accuracy is paramount — peer-reviewed research only
- Domain-Driven Design (DDD) + Event-Driven Architecture (EDA)
- "Last information wins" — newer docs override older ones

## Current Status (22 Feb 2026)
**Phase 1 COMPLETE** — all 4 contracts redeployed to Arbitrum Sepolia (19 Feb 2026) with `initialize()` pattern:

| Contract | Address | Tests | WASM |
|----------|---------|-------|------|
| DIURegistry | `0x49e1b11e1037e74113a7c0ccc41e3042d4691018` | 28 | 21.3KB |
| DIUReputation | `0x8740f9d110133ff5efa0fb562e62ab92a466cdc5` | 36 | 20.0KB |
| DIUAchievements | `0x1a9783ba7966c0e7299af7ee2228e19028d8ea7e` | 34 | 23.2KB |
| DIUToken | `0xbbd9a558c049482f1be45399fec4a4c9dc1c810e` | 49 | 17.7KB |

Deployer: `0x67bB4D1895D9A736F9e6076529B468ba05aeD150` | Total: **147 tests**

**Next (Phase 2)**: DIUProgress contract, security review with Kirill, backend API, grant applications.

## Tech Stack
| Component | Stack |
|-----------|-------|
| Contracts | Rust 1.93, stylus-sdk 0.10.0, alloy 0.7, wasm32-unknown-unknown |
| Frontend | React 18, Three.js 0.158, Vite 6.4, Tailwind 3.3, TypeScript 5.2 |
| Backend | Axum 0.7 (stub in physics-tutorial/backend/, not prod) |
| Tooling | cargo-stylus, mini-alloc 1.0 |

## Project Structure
```
~/projects/diu-os/                    <- WORKSPACE ROOT
├── CLAUDE.md                         <- This file (concise context)
├── WORKFLOW.md                       <- How to work with Claude Code
├── ROADMAP.md                        <- Phases, grants, timeline
├── ARCHITECTURE.md                   <- ADRs, contract design, security
├── .gitignore                        <- Ignores _workspace/, target/
├── _workspace/                       <- Working docs, archives (gitignored)
├── .claude/
│   ├── commands/                     <- /catchup, /status, /check-contract, /pr, /new-contract
│   ├── commands-archive/             <- /deploy-testnet, /sync-repos, etc.
│   └── rules/project-rules.md       <- Auto-applied rules
├── diu-contracts/                    <- ACTIVE: Stylus/Rust smart contracts (own git repo)
│   └── src/{lib,registry,reputation,achievements,token}.rs
├── physics-tutorial/                 <- LIVE MVP (own git repo)
│   ├── frontend/                     # React + Three.js (27 components, 3 sims, 8 langs)
│   └── backend/                      # Axum stub (not in prod)
├── contracts/                        <- DEPRECATED Solidity (own git repo, reference only)
├── diu-docs/                         <- Documentation (minimal, own git repo)
├── diu-os.github.io/                 <- Main site (GitHub Pages, own git repo)
├── developer-portal/                 <- Dev docs (GitHub Pages, own git repo)
└── manifesto/                        <- IP protection v1.0.0 (own git repo)
```

## Contracts Summary
4 contracts, feature-gated (one `#[entrypoint]` at a time via Cargo features):
- **DIURegistry** — User identity, ORCID linking, researcher verification
- **DIUReputation** — XP, levels (1-5), daily streaks, leaderboard
- **DIUAchievements** — ERC-721 NFT badges and certificates
- **DIUToken** — ERC-20 platform token (rewards, governance)

**Access Control**: Registry(public+admin) | Reputation(public+backend) | Achievements(public+backend) | Token(public+backend+admin)

For detailed contract APIs, storage layouts, and security patterns, see `diu-contracts/README.md` and `diu-contracts/docs/`.
For architecture decisions and migration patterns, see `ARCHITECTURE.md`.

## Current Sprint (Phase 2 Prep)
- **P0**: Security review with Kirill (nonces, proxy, audit firms)
- **P0**: DIUProgress contract design
- **P0**: Добавить ADR D-019–D-023 в ARCHITECTURE.md ✅ (27 Feb 2026)
- **P0**: Создать _workspace/grants/WEIL_VALIDATION.md ✅ (27 Feb 2026)
- **P1**: Backend API (Axum + alloy + SIWE auth + PostgreSQL)
- **P1**: MCP Physics Server stub (параллельно с DIUProgress — ADR D-020)
- **P1**: Prepare Stylus Sprint grant application
- **P1**: Gitcoin GG25 application draft (AI For Public Goods positioning)
- **P1**: Создать профиль на builder.gitcoin.co (не ждать открытия GG25)
- **P1**: Оптимизировать Gitcoin Passport на passport.gitcoin.co
- **P2**: Double-slit physics core: JS → Rust/WASM rewrite (ADR D-021)
- **P2**: wagmi/Web3 frontend integration
- **P2**: Мониторить gov.gitcoin.co — анонс доменов GG25 (за 4–6 нед. до раунда)
- **P2**: Собрать impact metrics: MAU physics.diu-os.org, отзывы профессоров,
  GitHub stars (нужно для QF-кампании)

## Model Strategy
Default: Sonnet for all daily work.
- Haiku (claude-haiku-4-5): /pr commits, clippy fixes, doc comments, formatting
- Sonnet (claude-sonnet-4-6): feature development, tests, grant writing, frontend
- Opus (claude-opus-4-6): new contract architecture (DIUProgress, DIUCrowdfunding),
  security review with Kirill, complex Stylus debugging, ADR authoring
- opusplan: use when starting a new contract or major refactor
  (Opus plans → Sonnet implements automatically)

Check model anytime: /status
Switch mid-session: /model claude-haiku-4-5 | claude-sonnet-4-6 | claude-opus-4-6 | opusplan

## Key Commands
```bash
# Session
/daily-reset                          # Daily checkpoint: save log, load context, print plan
/catchup                              # Read changed files, get up to speed
/status                               # Overview of all repos and contracts
/check-contract                       # clippy + test + WASM check
/pr                                   # Prepare commit (test -> stage -> commit msg)
/new-contract                         # Create new contract from template

# Smart Contracts (from diu-contracts/)
cargo test                            # All 147 tests
cargo clippy -- -D warnings           # Strict lint (0 warnings required)
cargo stylus check --endpoint https://sepolia-rollup.arbitrum.io/rpc
cargo stylus deploy --endpoint https://sepolia-rollup.arbitrum.io/rpc \
  --private-key-path ~/.keys/diu-deployer --max-fee-per-gas-gwei 0.1
```

## Coding Conventions
- `Result<T, E>` over panics — never `unwrap()` in production
- All public functions: doc comments (`///`)
- Strict clippy: `cargo clippy -- -D warnings` must pass
- Stylus macros: `#[entrypoint]`, `#[public]`, `sol_storage!`, `sol!`
- Patterns: https://stylus-by-example.org/
- Conventional commits: `feat:`, `fix:`, `docs:`, `test:`, `refactor:`
- TypeScript: functional components, hooks, `< 200` lines per component

## Links
| Resource | URL |
|----------|-----|
| Live MVP | https://physics.diu-os.org |
| GitHub Org | https://github.com/diu-os |
| Stylus Docs | https://docs.arbitrum.io/stylus |
| Stylus by Example | https://stylus-by-example.org/ |
| Stylus SDK | https://docs.rs/stylus-sdk/0.10.0/stylus_sdk/ |
| Arbitrum Grants | https://arbitrum.foundation/grants |
| Sepolia Explorer | https://sepolia.arbiscan.io/ |

## See Also
- `ROADMAP.md` — phases, grants pipeline, UX roadmap, international expansion
- `ARCHITECTURE.md` — ADRs, contract design, security patterns, pending decisions
- `WORKFLOW.md` — Claude Code workflow, context management, security practices
- `.claude/rules/project-rules.md` — auto-applied guardrails
- `diu-contracts/docs/` — detailed security audit, business logic analysis

---

# PART 2: ROADMAP.md

# DIU OS — Roadmap

**Last updated**: 22 February 2026

---

## Smart Contracts Phases

### Phase 1: Foundation (Feb 2026) — COMPLETE ✅
All 4 contracts deployed and redeployed to Arbitrum Sepolia (19 Feb 2026, `initialize()` pattern):
- DIURegistry (28 tests, 21.3KB), DIUReputation (36, 20.0KB), DIUAchievements (34, 23.2KB), DIUToken (49, 17.7KB) = **147 tests total**

### Phase 2: Extension (Apr-May 2026)
| Week | Task | Deliverable |
|------|------|-------------|
| 9-10 | DIUProgress contract | Learning state, module completion, quiz results |
| 11-12 | Security review with Kirill | Audit-ready code, nonce patterns, proxy decision |
| 13-14 | Backend REST API (Axum) + DB | SIWE auth, alloy integration, PostgreSQL |
| 15-16 | Submit Stylus Sprint + Audit Subsidy | Grant applications |

**Phase 2 contracts**:
- **DIUProgress** — Learning state tracking, module completions, quiz results. Cross-contract: calls DIUReputation.addXP
- **DIUCrowdfunding** — Research funding, milestones, refund logic. Cross-contract: calls DIUToken.transfer

### Phase 3: Mainnet + Traction (Jun-Aug 2026)
| Week | Task | Deliverable |
|------|------|-------------|
| 17-20 | External audit (subsidized via Arbitrum) | Audit report |
| 21-22 | Deploy to Arbitrum One mainnet | Production deployment |
| 23-26 | User acquisition (target: 500+ users) | Traction metrics |
| 27-30 | AI integration (Trailblazer grant) | MCP-based AI features |

### Phase 4: DAO (2027+)
- **DIUGovernance** — Proposals, voting, execution
- **DIUStaking** — Token locking, voting power, staking rewards

---

## Grants Pipeline

| Program | Amount | Priority | Timeline | Status |
|---------|--------|----------|----------|--------|
| **Arbitrum Stylus Sprint** | 5M ARB pool | P0 | Apr 2026 | Preparing |
| **Arbitrum Audit Subsidy** | $10M ARB pool | P0 | After Phase 2 | After security review |
| **Trailblazer AI Grant** | $1M pool | P1 | After Phase 2 | Research |
| **Gitcoin GG25** | QF matching | P1 | Q2 2026 | Preparing — см. ниже |
| **Ethereum Foundation** | $2M academic | P2 | Q3 2026 | Research |

**Excluded**: FunDeSci (trust issues — permanent exclusion).

### Gitcoin GG25 — Детальная стратегия (Q2 2026)

**Ключевой факт**: DeSci больше НЕ является отдельным доменом в Gitcoin Grants (с GG24, Oct 2025).
Позиционирование под DeSci = автоматический отказ.

**Целевые домены (в порядке приоритета):**

1. **Public Goods R&D / AI For Public Goods** (P0)
   - Позиционирование: open-source симуляции квантовой физики = public good,
     AI-тьютор = AI for public goods (образование), SDG 4 Quality Education
   - Заявление в заявке: "DIU OS — open-source scientific education platform:
     3D quantum physics simulations + AI tutor + decentralized on-chain credentials"
   - НЕ упоминать DeSci в первом предложении — сначала "public good" + "education"

2. **Dev Tooling & Infrastructure** (P1)
   - Позиционирование: Rust/Stylus паттерны из diu-contracts/ как tooling
     для Arbitrum-разработчиков (SECURITY_AUDIT.md, BUSINESS_LOGIC_ANALYSIS.md —
     это реальный вклад в экосистему)
   - Один профиль на builder.gitcoin.co покрывает оба домена одновременно

**Механика QF (критично для стратегии):**
- Число доноров важнее суммы: 50 доноров × $2 > 1 донор × $100 по matching
- 1 DAI от 10 людей может дать до 400 DAI matching
- Нужна широкая community-мобилизация: Twitter, Farcaster, профессора, студенты

**Validation:** Positioning "AI For Public Goods" подтверждена Kevin Weil (OpenAI VP, a16z speedrun,
Feb 2026) — наш MCP Ensemble Mesh соответствует best practices OpenAI. Использовать как credibility
signal в заявке: "Архитектура следует паттернам, рекомендованным VP of Science OpenAI."

**Технический процесс:**
- Профиль: builder.gitcoin.co (один профиль → несколько раундов)
- Платформа: Giveth.io (партнёр Gitcoin с GG24, QF-провайдер)
- Passport: passport.gitcoin.co — верифицировать GitHub (139 тестов),
  Twitter, контракты на Arbiscan

**Мониторинг:**
- Домены GG25 объявляются через голосование GTC-холдеров на gov.gitcoin.co
- Следить за анонсами за 4–6 недель до открытия раунда
- Matching-партнёры прошлых раундов: VitaDAO (DeSci!), EF, Protocol Labs, a16z

**Исключено**: FunDeSci (permanent exclusion, см. выше в ROADMAP.md)

---

## UX/Frontend Roadmap

| Phase | Period | Focus |
|-------|--------|-------|
| UX-1 | Feb-Mar 2026 | Explorer Mode improvements |
| UX-2 | Apr-May 2026 | Laboratory Mode v1 (Modules 1-3, quizzes) |
| UX-3 | Jun-Aug 2026 | Research Mode v1 (custom params, data export, AI) |
| UX-4 | Q3-Q4 2026 | Web3 integration (NFTs, on-chain progress, wagmi) |
| UX-5 | 2027 | Multi-discipline expansion |

### Three UX Modes
| Mode | Audience | Features |
|------|----------|----------|
| **Explorer** | Casual visitors | Free 3D simulations, quick info popups, donate button |
| **Laboratory** | Students | Guided modules, step-by-step, quizzes, NFT achievements |
| **Research** | Scientists | Custom params, data export, AI assistant, on-chain attribution |

---

## Backend Roadmap
| Phase | Task |
|-------|------|
| B-1 | Production Axum backend with DDD structure |
| B-2 | SIWE (Sign-In with Ethereum) authentication |
| B-3 | PostgreSQL + Supabase CDC for real-time + **MCP Server stub (Physics + Knowledge) — параллельно** |
| B-4 | alloy integration for blockchain interaction |
| B-5 | ~~MCP servers~~ **MOVED UP → B-3** — full MCP implementation (Progress tracking, versioned interfaces) |

> **ADR D-020**: MCP scaffolding поднято с B-5 на B-3 (capability window — строить инфраструктуру
> до готовности финального UX). Source: Kevin Weil, a16z speedrun, 26 Feb 2026.

---

## International Expansion

| Phase | Jurisdiction | Timeline | Purpose |
|-------|-------------|----------|---------|
| 1 | Georgia LLC | Q2-Q3 2026 | Bank accounts, Stripe integration, crypto compliance |
| 2 | UAE DIFC / Singapore | 2027+ | International scaling, institutional partnerships |

**Geographic Constraints**:
- Uzbekistan lacks Stripe/PayPal support — crypto is the only monetization path pre-relocation
- Georgia LLC required for fiat payment rails and mainstream business infrastructure
- Relocation path: Georgia → UAE → Singapore (aligns with Web3 + traditional finance needs)

**Business Model**: Foundation (.org) open source + Enterprise (.com) B2B
**Beachhead Market**: Quantum physics researchers, expanding to other disciplines

---

## Fundraising Strategy

### Pre-Seed Round (Target: Q2-Q3 2026)
- **Amount**: $150K (9-month runway)
- **Use of Funds**:
  - Team: 70% (2-3 core developers)
  - Infrastructure: 15% (hosting, RPC, analytics)
  - Security: 10% (external audit, bug bounties)
  - Legal: 5% (Georgia LLC, compliance)

### a16z CSX Speedrun SR007 (Target: Q3-Q4 2026)
- **Program**: Season 7 (acceptance rate <0.4%)
- **Requirement KPIs**:
  - 500+ active users (current: ~50)
  - 100+ GitHub stars (current: ~10)
  - $1K+ MRR or equivalent traction
- **Timeline**: Application after Phase 3 launch on Arbitrum mainnet
- **Strategy**: Demonstrate technical depth (Rust/Stylus) + scientific rigor + early traction

### Grant Diversification
Avoid single-point dependency: combine Arbitrum grants (Stylus Sprint, Audit Subsidy, Trailblazer), Gitcoin QF, and Ethereum Foundation academic funding.

---

## Market Context

### Competitive Landscape
**Key Competitor: Phylo** (closest analog)
- **Funding**: $13.5M Series A (led by a16z crypto, January 2024)
- **Focus**: Biology / genomics research platform
- **Traction**: 7,000+ research labs, peer-reviewed papers published
- **Validation**: Proves "Scientific OS" category viability and investor appetite

**DIU Positioning**: "First DeSci Platform for Physics Education" — different discipline, different pedagogy (interactive 3D simulations vs. static datasets).

### DeSci Market Data (2024-2025)
- Market capitalization surged **2,640%** in 2024 to **$1B+**
- **180+ active projects** competing for attention
- Academic adoption: **3/10** maturity (still early)
- Funding concentrates in top 3-4 projects; mid-tier struggles
- Mainstream media largely ignores DeSci (niche community focus)

### Market Timing Warning
> "Window to differentiate is narrowing."

- 180+ projects in survival mode post-hype cycle
- Technical differentiation (Rust/Stylus, scientific accuracy) is critical
- First-mover advantage in physics education vertical must be captured in 2026
- Grant funding environment competitive but accessible with strong execution

---

# PART 3: ARCHITECTURE.md — ADR Section Only

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
  - IntentClassifier SLM — быстрая классификация намерений (<100ms, дёшево)
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

---

# PART 4: _workspace/grants/WEIL_VALIDATION.md

# Архитектурная валидация DIU OS — Kevin Weil, OpenAI VP

Дата: 26 February 2026, a16z speedrun fireside chat
Source: speedrun.substack.com

---

## Что валидировано

### 1. Ensemble Model Architecture (ADR D-022)
**Weil**: "A lot of things today turn out best when you use an ensemble of models.
An orchestrating model puts a plan together. Then cheaper models trained to do one thing
really well. I don't see people doing that enough."

**DIU OS**: Orchestrator LLM + SLM IntentClassifier + PhysicsValidator + KnowledgeRAG (Qdrant)

**Статус**: ✅ Совпадает с нашим ADR D-018/D-022

---

### 2. Simulation-First Research Loop (ADR D-019)
**Weil**: "The model thinks, maybe running a simulation, then sending that to robotic labs
which you can scale horizontally. Results come back. The model thinks more. You have tight
loops with simulation, and longer loops through the real world."

**DIU OS**: AI↔Quantum Simulation tight loop (<2s) уже в MVP. Extended loop (data export)
в scope DIUProgress (Phase 2).

**Статус**: ✅ Мы строим именно это

---

### 3. Infrastructure Ahead of Product (ADR D-020)
**Weil**: "You go very quickly from 'models could never do this thing' to 'models can just
barely do this thing' — maybe 5-10%. And then six to twelve months later, models are great
at this thing — 60 or 80%."

**DIU OS**: строим MCP Mesh stub сейчас (B-3), до готовности Research Mode UX.
MCP сейчас на ~10% — через 6-12 месяцев будет production-ready вместе с инфраструктурой.

**Статус**: ✅ Правильная стратегия подтверждена

---

### 4. Timing — "Most Fertile Ground for Startups"
**Weil**: "Models can do something computers have never done before. And in another month,
that happens again. OpenAI doesn't always know what's possible. It is just the most fertile
ground for startups that there has ever been."
Сказано перед a16z speedrun аудиторией (Feb 2026).

**DIU OS**: a16z CSX SR007 в целях (Q3-Q4 2026). Weil выступал именно перед a16z speedrun
аудиторией — direct validation нашего target investor.

**Статус**: ✅ Timing совпадает

---

### 5. Versioned Plugin Architecture (ADR D-023)
**Weil**: "In another month, that happens again." — новые capabilities ежемесячно.

**DIU OS**: Versioned MCP interfaces (`mcp://physics.diu-os/v1/`). Новая модель = новый
MCP Server, без рефакторинга Orchestrator.

**Статус**: ✅ Архитектура готова к ежемесячным AI обновлениям

---

## Использование в grant applications

### Шаблон для Gitcoin GG25 (AI For Public Goods):
> "Наша MCP Ensemble архитектура следует паттернам, которые OpenAI считает industry
> best practice: orchestrating model + specialized smaller models (Kevin Weil, VP of
> Science OpenAI, a16z speedrun, February 2026)."

### Шаблон для Arbitrum Trailblazer:
> "DIU OS строит двухуровневую Research Loop (AI↔Simulation↔Data Export), архитектурно
> совпадающую с видением будущего science AI, описанным Kevin Weil (OpenAI) на a16z
> speedrun (Feb 2026): tight loops with simulation, longer loops through the real world."

### Шаблон для a16z CSX:
> "Weil outlined the most fertile ground for startups in history at a16z speedrun (Feb 2026).
> DIU OS is positioned at the exact intersection he described: AI + simulation + scientific
> infrastructure, built with ensemble model architecture as recommended by OpenAI."

---

## Связанные ADR

| ADR | Тема | Weil Тезис |
|-----|------|------------|
| D-019 | Simulation-First Research Loop | Тезис 2 |
| D-020 | Capability Window — Build Now | Тезис 1 |
| D-021 | Stateless WASM Simulations | Тезис 2 (горизонтальное масштабирование) |
| D-022 | Ensemble Model Routing | Тезис 5 |
| D-023 | Versioned MCP Interface | Тезис 4 (monthly capability releases) |

---

# PART 5: diu-contracts — git log --oneline -10

```
d89c7e2 docs(business-logic): bump to v1.1, reflect testnet redeployment status
d258f94 docs(security): bump to v1.1, reflect testnet redeployment status
96a8b52 docs: update contract test counts and WASM sizes to post-initialize() values
5863050 docs: add Arbitrum Sepolia testing guide with deployed addresses
f2bdb06 refactor(contracts): migrate from constructor() to initialize() pattern
300382a docs: remove AI-assisted development attribution from README
8e4c81c feat: complete Arbitrum Sepolia deployment (4/4 contracts)
685df86 docs: add security audit and business logic analysis
0bb93fc chore: harden .gitignore for deploy phase
fda2810 docs: add README with contract documentation and architecture
```
