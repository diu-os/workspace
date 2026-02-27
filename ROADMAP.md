# DIU OS — Roadmap

**Last updated**: 27 February 2026

---

## Smart Contracts Phases

### Phase 1: Foundation (Feb 2026) — COMPLETE ✅
All 4 contracts deployed and redeployed to Arbitrum Sepolia (19 Feb 2026, `initialize()` pattern):
- DIURegistry (28 tests, 21.3KB), DIUReputation (36, 20.0KB), DIUAchievements (34, 23.2KB), DIUToken (49, 17.7KB) = **147 tests**

### Phase 2: Extension (Mar-May 2026) — IN PROGRESS 🔄
| Week | Task | Deliverable | Status |
|------|------|-------------|--------|
| 9-10 | DIUProgress contract | Simulation tracking, XP cross-contract, data export | ✅ deployed 27 Feb |
| 11-12 | Security review with Kirill | Audit-ready code, nonce patterns, proxy decision | 🔄 in progress |
| 13-14 | Backend REST API (Axum) + DB | SIWE auth, alloy integration, PostgreSQL | pending |
| 15-16 | Submit Stylus Sprint + Audit Subsidy | Grant applications | pending |

**Phase 2 contracts**:
- **DIUProgress** ✅ `0xb1c4edc73aae322f62cda57f84f303761ca3e347` — Simulation tracking, module completions, XP awards. 24 tests, 19.0KB. Cross-contract: DIUReputation.addXP. Data export hook (ADR D-019). Total: **171 tests**
- **DIUCrowdfunding** — Research funding, milestones, refund logic. Cross-contract: DIUToken.transfer

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
- Passport: passport.gitcoin.co — верифицировать GitHub (171 тест),
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
