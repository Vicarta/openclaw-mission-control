# Requirements: Mission Control Operations

**Defined:** 2026-03-25
**Core Value:** `Mission Control` має бути спостережуваним, ізольованим і керованим так, щоб не ламати основний `OpenClaw` runtime та робочі workspace-и користувача.

## v1 Requirements

### Documentation And Context

- [ ] **DOC-01**: Репозиторій зберігає актуальний опис інсталяції, інтеграції, шляхів, URL і критичних operational lessons для `Mission Control`.
- [ ] **DOC-02**: Оператор має короткий runbook для підключення, перевірки стану сервісів і навігації по ключових файлах на сервері.

### Isolation And Safety

- [ ] **SAFE-01**: `Mission Control` використовує лише dedicated workspace root `/opt/openclaw/config/mission-control`.
- [ ] **SAFE-02**: provisioning `Mission Control` не чіпає існуючі user workspace-и `main/reminder/harvester`.
- [ ] **SAFE-03**: `Mission Control` створює лише свого dedicated `mc-gateway-*` control-plane агента, а не побічні user agents.
- [ ] **SAFE-04**: Для MC-generated templates і heartbeat внутрішній backend URL лишається `http://backend:8000`, а не зовнішній `Tailscale` backend endpoint.
- [ ] **SAFE-05**: Поточний виняток `tools.exec` задокументований як локальний і застосовується лише до `mc-gateway-a736f444-d548-448e-9736-11a4516a8735`.
- [ ] **SAFE-06**: Кожен додатковий `mc-gateway-*` агент або `workspace-gateway-*` каталог пояснений і визнаний очікуваним, або ж видалений як orphan/stale state.

### Stability And Monitoring

- [ ] **MON-01**: Оператор може підтвердити кілька успішних `heartbeat` cycles поспіль для `mc-gateway-*` агента.
- [ ] **MON-02**: Оператор може звірити heartbeat на боці `Mission Control backend`, включно з `POST /api/v1/agent/heartbeat` і переходом агента в `online`.
- [ ] **MON-03**: Для збоїв heartbeat існує базовий діагностичний маршрут і список файлів/сервісів для перевірки.

### Functional Validation

- [ ] **FLOW-01**: `Board creation` проходить end-to-end без пошкодження існуючої інтеграції `OpenClaw`.
- [ ] **FLOW-02**: `Board lead flow` проходить end-to-end з очікуваною поведінкою control-plane агента.
- [ ] **FLOW-03**: Interactive `board onboarding` використовує окремий board-scoped session contract або інший механізм, ізольований від routine `gateway-main` heartbeat/control-plane turns.
- [ ] **FLOW-04**: Після кожного onboarding `start` або `answer` `Mission Control` переходить або до наступного питання, або до `status=complete`, або до явного `stalled/failed` state в межах bounded timeout; infinite loading не допускається.
- [ ] **FLOW-05**: Silent `NO_REPLY` і missing callback outcomes у `board onboarding` виявляються, логуються і покриті regression tests / operator diagnostics.

### Security And Governance

- [ ] **SECU-01**: Після стабілізації оцінено більш вузьку policy-модель для `mc-gateway-*`, ніж `tools.exec.security=full`.
- [ ] **SECU-03**: Підтверджено, чи глобальний `tools.exec` у `/opt/openclaw/config/openclaw.json` має існувати взагалі для цього середовища.
- [ ] **SECU-02**: Задокументовано рішення, чи має `Mission Control` право глибше керувати `board agents` у цьому середовищі.

## v2 Requirements

### Automation And Operations

- **AUTO-01**: Моніторинг heartbeat автоматизовано регулярною перевіркою або automation-задачею.
- **AUTO-02**: Board flows мають повторюваний smoke-test сценарій.
- **AUTO-03**: Runbook-и доповнені відновленням після токен-ротації, template resync та device pairing issues.

## Out of Scope

| Feature | Reason |
|---------|--------|
| Перевикладка всього `OpenClaw` як окремого greenfield стеку | Поточне завдання про стабілізацію вже наявної інтеграції, а не повний replatform |
| Міграція `Mission Control` у спільні user workspace-и | Це суперечить вимогам ізоляції та створює ризик побічних змін |
| Глобальне відключення exec approvals для всіх агентів | Поточний виняток виправданий лише для dedicated MC control-plane агента |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| DOC-01 | Phase 1 | Pending |
| DOC-02 | Phase 1 | Pending |
| SAFE-01 | Phase 1 | Pending |
| SAFE-02 | Phase 1 | Pending |
| SAFE-03 | Phase 1 | Pending |
| SAFE-04 | Phase 1 | Pending |
| SAFE-05 | Phase 1 | Pending |
| SAFE-06 | Phase 1 | Pending |
| MON-01 | Phase 2 | Pending |
| MON-02 | Phase 2 | Pending |
| MON-03 | Phase 2 | Pending |
| FLOW-01 | Phase 3 | Pending |
| FLOW-02 | Phase 3 | Pending |
| FLOW-03 | Phase 03.1 | Pending |
| FLOW-04 | Phase 03.1 | Pending |
| FLOW-05 | Phase 03.1 | Pending |
| SECU-01 | Phase 4 | Pending |
| SECU-03 | Phase 4 | Pending |
| SECU-02 | Phase 5 | Pending |

**Coverage:**
- v1 requirements: 19 total
- Mapped to phases: 19
- Unmapped: 0 ✓

---
*Requirements defined: 2026-03-25*
*Last updated: 2026-03-25 after initial project initialization*
