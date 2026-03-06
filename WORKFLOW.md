# DIU OS — Claude Code Workflow Guide

Guide for effective work with Claude Code in the DIU OS project.
Based on best practices from billion-token teams and security research.

**Last updated**: 06 March 2026

---

## Getting Started

```bash
cd ~/projects/diu-os
claude
```

Claude Code reads on startup:
- `CLAUDE.md` — project context (~150 lines)
- `.claude/rules/project-rules.md` — auto-applied guardrails
- `.claude/commands/` — available slash commands

Check context usage: `/context`

---

## Daily Commands (Core 5)

| Command | When to Use |
|---------|-------------|
| `/catchup` | Session start — read all changed files |
| `/status` | Overview of all repos and contracts |
| `/check-contract` | After contract changes — clippy + test + WASM |
| `/pr` | Ready to commit — verify + prepare commit |
| `/new-contract` | Create new contract from template |

Secondary commands (in `commands-archive/`):
- `/deploy-testnet` — deploy to Sepolia (rare)
- `/sync-repos` — sync all repos (rare)
- `/backend-init` — initialize backend (future)

---

## Deploy Commands (Arbitrum Sepolia)

```bash
# Check WASM compiles for a specific contract
cargo stylus check \
  --endpoint https://sepolia-rollup.arbitrum.io/rpc \
  --features registry

# Deploy a contract (use --max-fee-per-gas-gwei to avoid gas estimation failures)
cargo stylus deploy \
  --endpoint https://sepolia-rollup.arbitrum.io/rpc \
  --private-key-path ~/.keys/diu-deployer \
  --max-fee-per-gas-gwei 0.1

# Verify on explorer after deploy
# Check: https://sepolia.arbiscan.io/address/<deployed-address>
```

**Gas tip**: Default gas estimation is too tight on Sepolia. Always use `--max-fee-per-gas-gwei 0.1` to avoid `maxFeePerGas < baseFee` errors.

---

## Context Management (Critical)

Claude Code has 200K token context. Fresh DIU OS session uses ~15K. The remaining 185K fills up as you work.

### Three Reload Strategies

**1. `/clear` + `/catchup` (primary)**
For typical tasks. Clears context and loads current state:
```
/clear
/catchup
```

**2. "Document & Clear" (for complex tasks)**
When working on a large feature (new contract, refactoring):
```
> Write current plan and progress to progress.md
[Claude creates file]
/clear
> Read progress.md and continue
```

**3. `claude --continue` / `claude --resume` (continuation)**
```bash
claude --continue          # Continue last session
claude --resume            # Pick session from history
claude --resume <session>  # Continue specific session
```

### What to Avoid

- **`/compact`** — auto-compaction is unreliable, loses important context
- **Huge files in context** — give path + explain when to read, don't paste whole files
- **Very long sessions** — if `/context` shows >80% usage, time to reload

---

## CLAUDE.md Philosophy

### Principles (from teams running billions of tokens/month)

1. **Guardrails, not Manual** — document what Claude does wrong, not everything
2. **Don't embed docs** — give path + explain WHEN to read:
   - Bad: `See diu-contracts/src/registry.rs`
   - Good: `For Stylus storage patterns or sol_storage! errors, see diu-contracts/src/registry.rs`
3. **Never "Never" without alternative**:
   - Bad: `Never use Solidity`
   - Good: `Never use Solidity — all contracts must be Rust/Stylus (diu-contracts/src/)`
4. **Forcing function** — if the explanation is complex, simplify tooling instead
5. **Keep it short** — target CLAUDE.md size: ~150 lines

### CLAUDE.md Hierarchy
```
~/projects/diu-os/
├── CLAUDE.md                    <- MAIN (concise context)
├── diu-contracts/
│   └── CLAUDE.md                <- Stylus-specific (when needed)
├── physics-tutorial/
│   └── CLAUDE.md                <- Frontend-specific (when needed)
└── backend/
    └── CLAUDE.md                <- Backend-specific (future)
```

---

## Security Practices

### Real Threats (from published research)

1. **Invisible text in PDFs** — white-on-white text, invisible to humans but readable by agents.
   - **Defense**: Don't feed Claude untrusted PDFs. For papers — only from trusted sources (Optica, APS, Nature).

2. **Sycophancy Loop** — agent agrees even when you're wrong. Dangerous for:
   - Scientific simulations (simplifying formulas "for readability")
   - Smart contracts (skipping security checks "to be faster")
   - **Defense**: Always verify via `cargo test` and `cargo clippy` BEFORE accepting code.

3. **Fake plugins/commands** — social engineering via LinkedIn/Twitter. These DON'T EXIST:
   - `/plugin install`, plugin marketplace, `claude-plugins-official`
   - **Defense**: Only trust https://docs.anthropic.com and https://docs.claude.com

### Rule: "Verify Output, Not Input"
You can't prevent all malicious inputs. Instead — verify the RESULT:
- `cargo test` — do tests pass?
- `cargo clippy` — any warnings?
- `cargo stylus check` — does WASM compile?
- Review diff before commit — what exactly changed?

### Безопасность локальной среды — Claude Code + Deployer Key (Linux)
> Source: _workspace/references/how-to-secure-autonomous-tools.pdf (Machupalli, Feb 2026)
> Принцип: treat Claude Code as potentially hostile from day one.

#### Статус deployer ключа
- Ключ НЕ существовал на момент проверки (18 Feb 2026)
- Создан в ~/.keys/diu-deployer с правами 600
- ~/.keys/ директория с правами 700
- Никогда не коммитить в git (защищено через .gitignore)

#### Создание ключа (Linux, единоразово)
```bash
mkdir -p ~/.keys && chmod 700 ~/.keys
echo "0xTWOY_PRIVATE_KEY" > ~/.keys/diu-deployer
chmod 600 ~/.keys/diu-deployer
# Проверка:
ls -la ~/.keys/diu-deployer  # должно быть -rw------- (600)
```

#### Альтернатива: Linux Keyring через secret-tool
```bash
sudo apt install libsecret-tools
secret-tool store --label="DIU Deployer Key" service diu-os account deployer
# Использование в деплое:
secret-tool lookup service diu-os account deployer > /tmp/key && \
  cargo stylus deploy --private-key-path /tmp/key ... && \
  rm /tmp/key
```

#### Мониторинг исходящих соединений (Linux)
- **OpenSnitch** — аналог Little Snitch для Linux: github.com/evilsocket/opensnitch
- **iptables logging**: `sudo iptables -A OUTPUT -j LOG --log-prefix "OUTBOUND: "`
- Просматривать логи еженедельно: `sudo journalctl -k | grep OUTBOUND`

#### Приоритеты по срокам

| Срок | Действие | Статус |
|------|----------|--------|
| ~~Сейчас (P0)~~ | ~/.keys/diu-deployer создан с правами 600 | ✅ (Phase 1 deployed) |
| Phase 2 | Установить OpenSnitch для мониторинга | ⬜ |
| Phase 2 | Dedicated claude_runner user для изоляции | ⬜ |
| Phase 3 mainnet | Hardware wallet (Ledger) + multi-sig (P-007) | ⬜ |
| Phase 3 mainnet | HashiCorp Vault для audit log secret access | ⬜ |

---

## Subagents

### Don't Create Custom Subagents
Custom subagents hide context from the main agent and enforce rigid workflows. Instead:
1. Put all context in CLAUDE.md
2. Let Claude decide when to delegate via `Task(...)`
3. For parallel tasks use SDK:
```bash
claude -p "in diu-contracts/ run cargo test" &
claude -p "in physics-tutorial/frontend/ npm run build" &
wait
```

---

## Hooks (Future — Phase 2+)

When the project grows, add hooks:

### Block-at-Submit (recommended)
Block `git commit` if tests haven't passed:
```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash(git commit*)",
      "hook": "test -f /tmp/diu-tests-passed || exit 1"
    }]
  }
}
```

### Don't Block-at-Write
Don't block the agent while writing files — let it finish, verify on commit.

---

## Typical Workflows

### New Contract ("Caveat Prompter" rule)
```
> Write ADR for DIUCrowdfunding: context, decision, trade-offs.
  Save to ARCHITECTURE.md as D-XXX.
[Review ADR]
> Use plan mode. Show implementation plan before coding.
[Approve plan]
> Implement
/check-contract
/pr
```
Правило: **ADR сначала, код потом**. Без ADR — не начинать.

### Bug Fix
```
/catchup
> In registry.rs function X does Y, but should do Z. Fix it.
/check-contract
```

### QAT Check (перед новым деплоем)
```bash
cargo test --test fitness --features all-contracts  # инварианты
cargo clippy -- -D warnings                         # 0 warnings
cargo audit                                         # 0 critical CVE
k6 run load-tests/basic.js                          # p95 < 3s @ 50 users
```

### New ADR
```
> Add ADR D-0XX to ARCHITECTURE.md: [topic]
  Use format: Context / Decision / Consequence / Source
  Number sequentially after last ADR (current: D-028).
```

### Post-Session Update
```
> Update "Current Status" section in CLAUDE.md:
> [what was completed today], date 06 Mar 2026
```

---

## Sources
- "How I Use Every Claude Code Feature" — Shrivu Shankar (blog.sshh.io), Nov 2025
- "How We Hijacked a Claude Skill" — Josh Devon (securetrajectories.substack.com), Oct 2025
- "The Sycophantic Agent" — Josh Devon, Aug 2025
- Claude Code docs: https://docs.anthropic.com/en/docs/claude-code
