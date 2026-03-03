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

## Current Status (27 Feb 2026)
**Phase 1 COMPLETE** — all 5 contracts deployed to Arbitrum Sepolia with `initialize()` pattern:

| Contract | Address | Tests | WASM |
|----------|---------|-------|------|
| DIURegistry | `0x49e1b11e1037e74113a7c0ccc41e3042d4691018` | 28 | 21.3KB |
| DIUReputation | `0x8740f9d110133ff5efa0fb562e62ab92a466cdc5` | 36 | 20.0KB |
| DIUAchievements | `0x1a9783ba7966c0e7299af7ee2228e19028d8ea7e` | 34 | 23.2KB |
| DIUToken | `0xbbd9a558c049482f1be45399fec4a4c9dc1c810e` | 49 | 17.7KB |
| DIUProgress | `0xb1c4edc73aae322f62cda57f84f303761ca3e347` | 24 | 19.0KB |

Deployer: `0x67bB4D1895D9A736F9e6076529B468ba05aeD150` | Total: **171 tests**

**Next (Phase 2)**: Security review with Kirill, backend API, grant applications. DIUProgress ✅ deployed 27 Feb 2026.
**QAT Status**: QAT.md создан 03 Mar 2026. Fitness functions ✅ + load-tests ✅ завершены 03 Mar 2026.

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

## Current Sprint (Phase 2)
- **P0**: Fitness functions ✅ — `diu-contracts/src/tests/fitness.rs` + `load-tests/` (03 Mar 2026)
- **P0**: Security review with Kirill (nonces, proxy, audit firms)
- **P0**: DIUProgress contract design ✅ (27 Feb 2026)
- **P0**: DIUProgress implement + deploy ✅ (27 Feb 2026) — `0xb1c4edc73aae322f62cda57f84f303761ca3e347`
- **P0**: Добавить ADR D-019–D-023 в ARCHITECTURE.md ✅ (27 Feb 2026)
- **P0**: Создать _workspace/grants/WEIL_VALIDATION.md ✅ (27 Feb 2026)
- **P1**: Backend API (Axum + alloy + SIWE auth + PostgreSQL)
- **P1**: MCP Physics Server stub (ADR D-020)
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
