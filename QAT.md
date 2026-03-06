# DIU OS — Quality Architecture Testing (QAT)

**Last updated**: 06 March 2026
**Source**: Pureur & Bittner "You've Generated Your MVP Using AI" (InfoQ, Feb 2026)
**Owner**: Barust | **Reviewer**: Kirill Taran (security layer)

---

## Зачем QAT

Юнит-тесты проверяют **правильность** кода — делает ли функция то, что должна.
Архитектурные тесты проверяют **качественные характеристики** (QARs) — выдержит ли
система нагрузку, не сломается ли при сбое, нет ли дыр в безопасности.

**Принцип (Pureur & Bittner, 2026)**:
> "The only way to evaluate the behavior of a black box is through experimentation.
> Software architecture needs to become a primarily empirical approach."

171 юнит-тест — отличная база. Но они не отвечают на вопросы:
- Что произойдёт с симуляцией при 100 одновременных пользователях?
- Можно ли вызвать addXP без авторизации через прямой вызов контракта?
- Что случится с frontend если Arbitrum RPC упадёт на 5 минут?

QAT закрывает этот пробел.

**"Caveat Prompter" принцип**: перед каждым новым контрактом или фичей —
написать ADR с явными trade-offs, использовать как контекст промпта.

---

## QARs — Реестр требований

**Шаблон**: `[Stimulus] → [System] → [Response] measured by [Metric]`

### Performance

| ID | Stimulus | Response | Метрика | Статус |
|----|----------|----------|---------|--------|
| Q-01 | Открытие симуляции | Первый кадр на экране | p95 < 3s | ❌ Не тестировалось |
| Q-02 | Frontend вызывает view() контракта | RPC ответ получен | p95 < 2s | ❌ Не тестировалось |
| Q-03 | Запрос к AI в Research Mode | Первый токен получен | p95 < 2s | ❌ Phase 3 |
| Q-04 | 100 одновременных пользователей | Симуляция доступна | Error < 5% | ❌ Не тестировалось |

### Security

| ID | Stimulus | Response | Метрика | Статус |
|----|----------|----------|---------|--------|
| Q-05 | Unauthorized вызов addXP() | Контракт → Err | 100% rejection | ✅ Unit тест |
| Q-06 | Unauthorized вызов mint() | Контракт → Err | 100% rejection | ✅ Unit тест |
| Q-07 | Admin функция без multi-sig | Невозможно на mainnet | 0 single-key ops | ❌ Phase 3 |
| Q-08 | Re-initialization попытка | Контракт → Err | 100% rejection | ⚠️ Проверить |
| Q-09 | Replay attack на addXP | Дублирующая tx отклонена | 100% rejection | ❌ P-006 pending |
| Q-10 | XP rate limit превышен | Tx → RateLimitError | 100% rejection | ❌ Gap #4 |

### Reliability

| ID | Stimulus | Response | Метрика | Статус |
|----|----------|----------|---------|--------|
| Q-11 | Arbitrum RPC недоступен | Симуляция работает, данные degrade | Graceful UI | ❌ Не тестировалось |
| Q-12 | IPFS пин для NFT упал | UI показывает fallback | Graceful UI | ❌ Не тестировалось |
| Q-13 | ORCID API недоступен | Верификация в очереди | Queue mode | ❌ Gap #3 |
| Q-14 | Один контракт ревертит | Остальные не затронуты | Isolation | ⚠️ Проверить |

### Maintainability

| ID | Requirement | Метрика | Статус |
|----|-------------|---------|--------|
| Q-15 | Размер компонента React | < 200 строк | ⚠️ Не автоматизировано |
| Q-16 | Test coverage контрактов | ≥ 80% | ✅ 171 тест |
| Q-17 | Cross-contract calls задокументированы в ADR | 100% coverage | ✅ D-019 (Simulation-First Research Loop) |
| Q-18 | Все state mutations → события | 100% | ⚠️ Проверить |

### VR/AR/MR Performance (WebXR — Phase 3+, ADR D-028)

| ID | Stimulus | Response | Метрика | Статус |
|----|----------|----------|---------|--------|
| Q-21 | Рендеринг кадра в VR-гарнитуре | Frame доставлен до deadline | frame time < 11ms (90 FPS) | ❌ Phase 3 |
| Q-22 | Вход в иммерсивный WebXR сеанс | `immersive-vr` режим активен | cold start < 2s | ❌ Phase 3 |
| Q-23 | Physics расчёт за кадр (stateless) | Результат получен до рендеринга | < 8ms/frame (3ms буфер) | ❌ Phase 3 |

> **Целевые устройства**: Meta Quest 3 (72Hz), Apple Vision Pro (100Hz), WebXR-браузеры (Chrome 125+).
> Q-21..Q-23 становятся обязательными перед UX-4 (ADR D-028 WebXR wrapper).

### Scientific Accuracy (специфично для DIU OS)

| ID | Requirement | Метрика | Статус |
|----|-------------|---------|--------|
| Q-19 | Симуляции на peer-reviewed источниках | 100% sourced | ✅ Policy |
| Q-20 | AI ответы по физике ("Quantum" persona) | > 90% accuracy | ❌ Phase 3 |

**Легенда**: ✅ Выполнено | ⚠️ Частично | ❌ Не выполнено

---

## Architectural Fitness Functions

Fitness functions — автоматизированные тесты архитектурных инвариантов.
Запускаются в составе `cargo test --test fitness` и `npm test`.
Zero overhead в CI. Ловят архитектурные регрессии автоматически.

**Принцип**: если инвариант важен — он должен быть тестом, не комментарием.

### Layer 1: Smart Contracts

Файл: `diu-contracts/src/tests/fitness.rs`

```rust
/// ═══════════════════════════════════════════════════════════
/// FITNESS FUNCTIONS — DIU OS Smart Contracts
/// Тестируют архитектурные инварианты, не бизнес-логику
/// Запуск: cargo test --test fitness
/// ═══════════════════════════════════════════════════════════

/// ИНВАРИАНТ Q-08: Контракты не могут быть инициализированы дважды
#[test]
fn ff_contracts_cannot_be_reinitialized() {
    let mut registry = DIURegistry::default();
    registry.initialize(OWNER_ADDR).expect("First init must succeed");
    let second = registry.initialize(OWNER_ADDR);
    assert!(second.is_err(),
        "INVARIANT VIOLATED: Re-initialization must be rejected");
}

/// ИНВАРИАНТ Q-05: Только authorized backend вызывает addXP
#[test]
fn ff_add_xp_rejects_unauthorized() {
    let mut rep = DIUReputation::default();
    let attacker = Address::from([0x42u8; 20]);
    let result = rep.add_xp(attacker, 100);
    assert!(result.is_err(),
        "INVARIANT VIOLATED: Unauthorized addXP must be rejected");
}

/// ИНВАРИАНТ Q-06: Только authorized backend минтит токены
#[test]
fn ff_mint_rejects_unauthorized() {
    let mut token = DIUToken::default();
    let attacker = Address::from([0xAAu8; 20]);
    let result = token.mint(attacker, U256::from(1000));
    assert!(result.is_err(),
        "INVARIANT VIOLATED: Unauthorized mint must be rejected");
}

/// ИНВАРИАНТ Q-16: XP значения в пределах безопасных границ
#[test]
fn ff_xp_within_safe_bounds() {
    for xp in [0u64, 1, 1_000, u64::MAX / 2] {
        let level = calculate_level(xp);
        assert!(level <= 5u8,
            "INVARIANT VIOLATED: Level must be 1-5, got {} for xp={}", level, xp);
    }
}

/// ИНВАРИАНТ Q-18: DIUProgress record() — только authorized
#[test]
fn ff_progress_record_rejects_unauthorized() {
    let mut progress = DIUProgress::default();
    let attacker = Address::from([0xBBu8; 20]);
    let result = progress.record_simulation(attacker, 1u64, 1u64, true);
    assert!(result.is_err(),
        "INVARIANT VIOLATED: Unauthorized record must be rejected");
}
```

**Proptest — fuzzing** (добавить в `Cargo.toml: [dev-dependencies] proptest = "1.0"`):

```rust
use proptest::prelude::*;

proptest! {
    /// calculate_level детерминирован и не паникует на любом u64
    #[test]
    fn ff_level_calculation_never_panics(xp in 0u64..u64::MAX) {
        let level = calculate_level(xp);
        prop_assert!(level >= 1 && level <= 5);
    }

    /// Streak не уменьшается без gap между днями
    #[test]
    fn ff_streak_monotonic_within_day(initial in 0u32..1000u32) {
        let result = simulate_streak_update(initial, SAME_DAY);
        prop_assert!(result >= initial || result == 0);
    }
}
```

### Layer 2: Frontend (TypeScript)

Файл: `physics-tutorial/frontend/src/__tests__/architecture.fitness.test.ts`

```typescript
import * as fs from 'fs';
import * as glob from 'glob';

describe('Architectural Fitness Functions — Frontend', () => {

  // ИНВАРИАНТ Q-15: Компоненты < 200 строк (из CLAUDE.md)
  it('ff_all_components_under_size_limit', () => {
    const files = glob.sync('src/components/**/*.tsx');
    const violations: string[] = [];
    files.forEach(file => {
      const lines = fs.readFileSync(file, 'utf8').split('\n').length;
      if (lines >= 200) violations.push(`${file}: ${lines} lines`);
    });
    expect(violations).toHaveLength(0,
      'INVARIANT VIOLATED:\n' + violations.join('\n'));
  });

  // ИНВАРИАНТ: Blockchain вызовы только через hooks/ слой (DDD)
  it('ff_no_direct_blockchain_calls_in_components', () => {
    const files = glob.sync('src/components/**/*.tsx');
    const violations: string[] = [];
    const FORBIDDEN = [/window\.ethereum/, /new ethers\./, /useContractWrite\(/];
    files.forEach(file => {
      const content = fs.readFileSync(file, 'utf8');
      FORBIDDEN.forEach(p => {
        if (p.test(content)) violations.push(`${file}: ${p}`);
      });
    });
    expect(violations).toHaveLength(0,
      'INVARIANT VIOLATED — use hooks/ layer:\n' + violations.join('\n'));
  });

  // ИНВАРИАНТ Q-11: Симуляции работают без wallet (Explorer Mode)
  it('ff_simulations_have_no_wallet_dependency', () => {
    const simFiles = glob.sync('src/simulations/**/*.tsx');
    simFiles.forEach(file => {
      const content = fs.readFileSync(file, 'utf8');
      expect(content).not.toMatch(/useAccount|useConnect|wagmi/,
        `INVARIANT VIOLATED: ${file} — simulation must work without wallet`);
    });
  });
});
```

### Layer 3: Resilience Tests

Файл: `physics-tutorial/frontend/src/__tests__/resilience.fitness.test.ts`

```typescript
describe('Resilience Fitness Functions', () => {

  // ИНВАРИАНТ Q-11: Деградация при недоступном RPC — не белый экран
  it('ff_graceful_degradation_when_rpc_unavailable', async () => {
    jest.spyOn(rpcProvider, 'call').mockRejectedValue(
      new Error('Connection refused')
    );
    render(<App />);
    expect(screen.getByTestId('simulation-canvas')).toBeInTheDocument();
    await waitFor(() => {
      expect(screen.queryByText('Application Error')).not.toBeInTheDocument();
    });
  });

  // ИНВАРИАНТ Q-12: NFT display при недоступном IPFS
  it('ff_nft_display_fallback_when_ipfs_unavailable', async () => {
    jest.spyOn(global, 'fetch').mockRejectedValue(new Error('IPFS unavailable'));
    render(<AchievementBadge tokenId={1} />);
    await waitFor(() => {
      expect(screen.getByTestId('badge-placeholder')).toBeInTheDocument();
    });
  });
});
```

---

## Security Testing Plan

### Ручной Security Checklist (перед каждым деплоем нового контракта)

Результаты фиксировать в `diu-contracts/docs/SECURITY_AUDIT.md`.

```
DIURegistry
□ Повторная регистрация одного ORCID с разных адресов — отклоняется?
□ verify() — только owner?
□ Что при ORCID API timeout — fallback или revert?

DIUReputation
□ addXP(u64::MAX) — нет overflow, возвращает Err?
□ recordLogin — повторный вызов в тот же день — idempotent?
□ Replay attack: та же tx дважды — отклоняется (P-006)?
□ Нет rate limiting → per-user daily XP cap (Gap #4, открыт)

DIUAchievements
□ Двойной mint одного badge — отклоняется?
□ mint без registration в DIURegistry — отклоняется?
□ tokenURI при отсутствующем IPFS пине — fallback URI?

DIUToken
□ pause() — что именно блокируется?
□ transfer при paused — отклоняется?
□ Нет multi-sig (Gap #1) — задокументировано для Phase 3

DIUProgress
□ record_simulation() без регистрации пользователя — отклоняется?
□ XP cross-contract call — что если DIUReputation недоступен?
□ get_export_snapshot() — только authorized?
□ Address::ZERO как reputation addr = test mode (задокументировано)
```

### Static Analysis (в CI, перед каждым деплоем)

```bash
cargo audit                          # CVE в зависимостях
cargo geiger                         # unsafe код в deps
cargo deny check                     # лицензии и policy
cargo clippy -- -D warnings          # уже есть, 0 warnings required

cd physics-tutorial/frontend
npm audit --audit-level=high
```

### Тестирование access control на Sepolia

```bash
# diu-contracts/scripts/security-check.sh
REPUTATION="0x8740f9d110133ff5efa0fb562e62ab92a466cdc5"
RPC="https://sepolia-rollup.arbitrum.io/rpc"
# ATTACKER_KEY — тестовый key, НЕ deployer

echo "=== Test: addXP без авторизации должен revert ==="
cast send $REPUTATION \
  "addXP(address,uint64)" \
  0x1234567890123456789012345678901234567890 100 \
  --private-key $ATTACKER_KEY \
  --rpc-url $RPC 2>&1 | grep -E "revert|error|success"
echo "=== ОЖИДАЕТСЯ: revert. Если success — КРИТИЧЕСКАЯ ДЫРА ==="
```

---

## Load Testing Plan

### Инструмент: k6

```bash
npm install -g k6   # или: snap install k6
```

### basic.js — Baseline (перед academic outreach)

Файл: `load-tests/basic.js`

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '1m', target: 20 },
    { duration: '3m', target: 50 },
    { duration: '1m', target: 0 },
  ],
  thresholds: {
    http_req_duration: ['p(95)<3000'],  // Q-01
    http_req_failed: ['rate<0.05'],
  },
};

export default function () {
  const res = http.get('https://physics.diu-os.org');
  check(res, {
    'status 200': (r) => r.status === 200,
    'loads in 3s': (r) => r.timings.duration < 3000,
  });
  sleep(1);
}
```

```bash
k6 run load-tests/basic.js
# Результат → load-tests/results/YYYY-MM-DD-basic.json
# Grant evidence: "Platform sustains 50 concurrent users, p95 < 3s"
```

### spike.js — University Lecture Scenario (100 users)

```javascript
export const options = {
  stages: [
    { duration: '30s', target: 100 },
    { duration: '5m',  target: 100 },
    { duration: '30s', target: 0 },
  ],
  thresholds: {
    http_req_duration: ['p(95)<5000'],
    http_req_failed: ['rate<0.10'],
  },
};
```

### Ручная матрица Three.js симуляций

```
Симуляция       | Chrome Desktop | Chrome Mobile | Firefox | Safari iOS
────────────────┼────────────────┼───────────────┼─────────┼───────────
Double-Slit     |   FPS / Time   |   FPS / Time  |         |
[другие]        |                |               |         |

Целевые: FPS > 30 | Первый кадр < 3s | Память < 500MB
Проводить перед каждым крупным релизом и перед university outreach.
```

---

## Chaos Testing Plan (Phase 3, перед mainnet)

```
Chaos Test 1: Arbitrum RPC недоступен
  Действие: неправильный RPC URL в .env
  Ожидание: симуляции работают, Web3 блок → fallback UI

Chaos Test 2: IPFS недоступен
  Действие: 127.0.0.1 ipfs.io в /etc/hosts
  Ожидание: NFT badges → placeholder, не broken icon

Chaos Test 3: Один контракт недоступен
  Действие: неправильный адрес DIUReputation в конфиге
  Ожидание: XP недоступен, симуляции работают

Chaos Test 4: Backend API недоступен (Phase 2+)
  Действие: остановить Axum backend
  Ожидание: Explorer Mode работает полностью (client-side)
```

---

## Security Gaps — Трекер

| Gap | Severity | Описание | Решение | Target | Статус |
|-----|----------|----------|---------|--------|--------|
| #1 No multi-sig | Critical | Admin ops на одном deployer key | Gnosis Safe 2-of-3 | Phase 3 mainnet | ❌ Open |
| #2 No universal pause | High | Нет аварийной остановки всех контрактов | PauseController контракт | Phase 2 | ❌ Open |
| #3 ORCID centralized | Medium | Single point of failure при верификации | Verification queue + fallback | Phase 2 | ❌ Open |
| #4 No XP rate limit | Medium | Нет лимита XP/день на пользователя | Per-user daily cap в DIUReputation | Phase 2 | ❌ Open |

**Текущий рейтинг**: B+ (внутренняя оценка, 10 Feb 2026)
**Целевой перед mainnet**: A (внешний аудит пройден, все gaps закрыты)

---

## Phase Checkpoints

### Phase 1 → Phase 2 (текущий момент, Mar 2026)
```
□ Fitness functions написаны для 5 контрактов (cargo test --test fitness)
□ Security checklist пройден по каждому контракту
□ cargo audit — 0 критических CVE
□ Baseline load test: 50 users, p95 < 3s (результат задокументирован)
□ Gaps #3 и #4 — план закрытия зафиксирован
```

### Phase 2 → Phase 3
```
□ Security review с Кириллом завершён (P-005, P-006, P-007)
□ Gap #2 закрыт (PauseController)
□ Gap #3 закрыт (ORCID verification queue)
□ Gap #4 закрыт (rate limiting в DIUReputation)
□ Spike load test: 100 users, 5 мин, p95 < 5s
□ Resilience tests: RPC + IPFS + Backend failover
□ QARs Q-01–Q-14 с измеренными результатами
□ npm audit + cargo audit: 0 critical
```

### Phase 3 → Mainnet
```
□ Внешний аудит завершён (P-009): рейтинг A
□ Gap #1 закрыт (Gnosis Safe multi-sig)
□ Chaos testing: все 4 сценария пройдены
□ Pentest: независимая security firm
□ QARs Q-01–Q-20 с измеренными результатами
```

---

## CI/CD Integration (Phase 2+)

```yaml
# .github/workflows/architecture.yml
name: Architecture Quality Gates
on: [push, pull_request]

jobs:
  fitness-contracts:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: |
          cd diu-contracts
          cargo test --test fitness
          cargo audit
          cargo clippy -- -D warnings

  fitness-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: |
          cd physics-tutorial/frontend
          npm ci
          npm test -- --testPathPattern=fitness
          npm audit --audit-level=high

  load-smoke:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - run: k6 run --vus 10 --duration 30s load-tests/basic.js
```

---

## See Also
- `ARCHITECTURE.md` — ADRs (D-019 Research Loop, D-021 Stateless WASM, D-028 VR/AR/MR)
- `ROADMAP.md` — фазы с QAT milestones, 90-day KPIs, Research Mode toolkit
- `diu-contracts/docs/SECURITY_AUDIT.md` — детальный security анализ
- `load-tests/` — k6 скрипты (создать в корне workspace)
