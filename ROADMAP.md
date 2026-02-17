# DIU OS — Roadmap

**Last updated**: 17 February 2026

---

## Smart Contracts Phases

### Phase 1: Foundation (Feb-Mar 2026) — COMPLETE
All 4 foundation contracts deployed to Arbitrum Sepolia on 15 Feb 2026:
- DIURegistry (26 tests), DIUReputation (34), DIUAchievements (32), DIUToken (47) = 139 tests total

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
| **Gitcoin Grants** | QF matching | P1 | Q2 2026 | Preparing |
| **Ethereum Foundation** | $2M academic | P2 | Q3 2026 | Research |

**Excluded**: FunDeSci (trust issues — permanent exclusion).

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
| B-3 | PostgreSQL + Supabase CDC for real-time |
| B-4 | alloy integration for blockchain interaction |
| B-5 | MCP servers (Physics simulation, Knowledge base, Progress tracking) |

---

## International Expansion

| Phase | Jurisdiction | Timeline |
|-------|-------------|----------|
| 1 | Georgia LLC | Q2 2026 |
| 2 | UAE DIFC / Singapore | 2027+ |

**Business Model**: Foundation (.org) open source + Enterprise (.com) B2B
**Beachhead Market**: Quantum physics researchers, expanding to other disciplines
