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
- **"Caveat Prompter"** — перед новым контрактом/фичей: сначала ADR с trade-offs, потом код
- "Last information wins" — newer docs override older ones

## Current Status (06 Mar 2026)
**Phase 2 IN PROGRESS** — 5 contracts on Arbitrum Sepolia, security ADRs resolved, QAT foundation complete:

| Contract | Address | Tests | WASM |
|----------|---------|-------|------|
| DIURegistry | `0x49e1b11e1037e74113a7c0ccc41e3042d4691018` | 28 | 21.3KB |
| DIUReputation | `0x8740f9d110133ff5efa0fb562e62ab92a466cdc5` | 36 | 20.0KB |
| DIUAchievements | `0x1a9783ba7966c0e7299af7ee2228e19028d8ea7e` | 34 | 23.2KB |
| DIUToken | `0xbbd9a558c049482f1be45399fec4a4c9dc1c810e` | 49 | 17.7KB |
| DIUProgress | `0xb1c4edc73aae322f62cda57f84f303761ca3e347` | 24 | 19.0KB |

Deployer: `0x67bB4D1895D9A736F9e6076529B468ba05aeD150` | Total: **171 tests**

**QAT**: Fitness functions ✅ + load-tests ✅ (03 Mar) | Security checklist ❌ Open
**ADRs**: D-025 (no proxy) + D-026 (nonces) + D-027 (multi-sig) ✅ resolved 04 Mar | D-028 VR/AR/MR ✅ 06 Mar
**AI**: Persona "Quantum" architected (ARCHITECTURE.md) — implementation Phase 3
**Docs**: 3-layer financial model + 90-day KPIs + Research Mode toolkit → ROADMAP.md (06 Mar)

## Tech Stack
| Component | Stack |
|-----------|-------|
| Contracts | Rust 1.93, stylus-sdk 0.10.0, alloy 0.7, wasm32-unknown-unknown |
| Frontend | React 18, Three.js 0.158, Vite 6.4, Tailwind 3.3, TypeScript 5.2 |
| Backend | Axum 0.7, SQLx, PostgreSQL, alloy 0.7, SIWE (EIP-4361) — B-1 in progress |
| Tooling | cargo-stylus, mini-alloc 1.0 |

## Project Structure
```
~/projects/diu-os/                    <- WORKSPACE ROOT
├── CLAUDE.md                         <- This file (concise context)
├── WORKFLOW.md                       <- How to work with Claude Code
├── ROADMAP.md                        <- Phases, grants, timeline
├── ARCHITECTURE.md                   <- ADRs, contract design, security
├── QAT.md                            <- Quality Architecture Testing (QARs, fitness functions, load tests)
├── .gitignore                        <- Ignores _workspace/, target/
├── _workspace/                       <- Working docs, archives (gitignored)
├── .claude/
│   ├── commands/                     <- /catchup, /status, /check-contract, /pr, /new-contract
│   ├── commands-archive/             <- /deploy-testnet, /sync-repos, etc.
│   └── rules/project-rules.md       <- Auto-applied rules
├── diu-contracts/                    <- ACTIVE: Stylus/Rust smart contracts (own git repo)
│   ├── src/{lib,registry,reputation,achievements,token,progress}.rs
│   ├── src/tests/fitness.rs          <- Architectural fitness functions ✅
│   └── load-tests/                   <- k6 load test scripts ✅
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
5 contracts, feature-gated (one `#[entrypoint]` at a time via Cargo features):
- **DIURegistry** — User identity, ORCID linking, researcher verification
- **DIUReputation** — XP, levels (1-5), daily streaks, leaderboard
- **DIUAchievements** — ERC-721 NFT badges and certificates
- **DIUToken** — ERC-20 platform token (rewards, governance)
- **DIUProgress** — Simulation tracking, experiment completion, cross-contract XP

**Access Control**: Registry(public+admin) | Reputation(public+backend) | Achievements(public+backend) | Token(public+backend+admin)

For detailed contract APIs, storage layouts, and security patterns, see `diu-contracts/README.md` and `diu-contracts/docs/`.
For architecture decisions and migration patterns, see `ARCHITECTURE.md`.

## Current Sprint (Phase 2) — as of 15 Mar 2026

### Завершено ✅
- Fitness functions `diu-contracts/src/tests/fitness.rs` + `load-tests/` (03 Mar)
- ADR D-019–D-027 в ARCHITECTURE.md (все security решения приняты)
- ADR D-028 VR/AR/MR + AI persona "Quantum" (06 Mar)
- QAT.md: Q-21..Q-23 VR performance targets (06 Mar)
- ROADMAP.md: 3-layer financial model + 90-day KPIs + Research Mode toolkit (06 Mar)
- DIUProgress deploy ✅ `0xb1c4edc73aae322f62cda57f84f303761ca3e347`
- _workspace/grants/WEIL_VALIDATION.md ✅
- **Security Audit Level 1 (15 Mar 2026)**:
  - R-1 ORCID global uniqueness — keccak256 reverse mapping, `OrcidAlreadyRegistered` error (commit `7e22b26`)
  - P-3 `get_export_snapshot` ACL — `require_authorized()` gate (commit `7e22b26`)
  - E-1 arithmetic overflow — `checked_add` + `ArithmeticOverflow` error (commit `7e22b26`)
  - A-2/P-1 Accepted Risk documented — backend enforces, Phase 2 on-chain guard (commit `8c717aa`)
  - Tests: 171 → 174, total **189** (174 unit + 15 fitness)
- **ADR D-029 PauseController** + `pause.rs` shared module, DIUToken refactored (commit `0319de0`)
- **Gap #4 (E-4)** per-user daily XP cap MAX_DAILY_XP=500 в DIUReputation (commit `75fd333`)
- **E-3 nonces** per-user on-chain nonce в DIUReputation (commit `a340b0b`, ADR D-026)
- **Q-15 Phase 1** — 4 компонента извлечены из App.tsx (commit `47d9488`):
  ExperimentSelector, ModeSelectorDropdown, TunnelingControls, HydrogenControls
- **Q-15 Phase 2** — 3 компонента извлечены (commit `dfad267`):
  ExperimentStatsPanels, ControlsSidebar, StatsSidebar
  App.tsx: 1357 → 483 строк
- **Q-15 Phase 3+4 ✅ ЗАКРЫТО** — хуки + SimulationCanvas (commit `ed9bb49`):
  `useSimulationState` (17 useState + handlers), `SimulationCanvas`, `CanvasOverlayInfo`
  App.tsx: 483 → **170 строк** (итого: 1357 → 170). Q-15 инвариант закрыт: все компоненты < 200 строк.
- **ADR D-030 ✅** — ORCID Verification Queue + Fallback (Gap #3, 15 Mar 2026)

### Открыто — P0 (блокирует Phase 3)
- **Security review с Кириллом** — code audit nonces/pause/ORCID (P-008/P-009 ещё открыты)
- **Gap #3 (реализация)**: `validate_orcid_format` в registry.rs + backend orcid_verifier worker ← **следующий приоритет**

### Открыто — P1
- Backend API: Axum + alloy + SIWE auth + PostgreSQL (B-1..B-3)
- MCP Physics Server stub параллельно с B-3 (ADR D-020)
- Stylus Sprint grant application (Apr 2026 deadline)
- Gitcoin GG25 draft (AI For Public Goods) + builder.gitcoin.co профиль
- Оптимизировать Gitcoin Passport (passport.gitcoin.co)

### Открыто — P2
- DIUCrowdfunding контракт (ADR перед кодом — "Caveat Prompter")
- Double-slit: JS → Rust/WASM rewrite (ADR D-021)
- wagmi/Web3 frontend integration (UX-4)
- Собрать impact metrics: MAU, отзывы профессоров, GitHub stars
- Мониторить gov.gitcoin.co — анонс доменов GG25 (за 4–6 нед. до раунда)

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
cargo test                            # All 171 tests
cargo test --test fitness --features all-contracts  # Only architectural fitness functions
cargo clippy -- -D warnings           # Strict lint (0 warnings required)
cargo audit                           # CVE check — run before every deploy
cargo stylus check --endpoint https://sepolia-rollup.arbitrum.io/rpc
cargo stylus deploy --endpoint https://sepolia-rollup.arbitrum.io/rpc \
  --private-key-path ~/.keys/diu-deployer --max-fee-per-gas-gwei 0.1

# Load Testing
k6 run load-tests/basic.js            # Baseline: 50 users, p95 < 3s
k6 run load-tests/spike.js            # Spike: 100 users (university lecture scenario)
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
- `QAT.md` — QARs registry, fitness functions, load tests, security checklist, phase checkpoints
- `WORKFLOW.md` — Claude Code workflow, context management, security practices
- `.claude/rules/project-rules.md` — auto-applied guardrails
- `diu-contracts/docs/` — detailed security audit, business logic analysis
