# Roadmap: Mission Control Operations

**Version:** v1
**Date:** 2026-03-25

## Milestone

Стабілізувати й операційно оформити `Mission Control` як безпечний control-plane sidecar для поточного `OpenClaw gateway`, не зачіпаючи існуючі user workspace-и.

## Phase 1: Project Baseline And Operational Context

**Goal:** Зафіксувати реальний стан інсталяції, інтеграції, ізоляції та тимчасових security exceptions у репозиторії.

**Requirements**:
- DOC-01
- DOC-02
- SAFE-01
- SAFE-02
- SAFE-03
- SAFE-04
- SAFE-05
- SAFE-06

**Exit Criteria:**
- Базова проектна документація існує локально й відображає поточний verified deployment state.
- Є окремий server context документ з критичними шляхами, URL, ідентифікаторами та technical lessons.
- Є короткий operations runbook для підключення та первинної перевірки системи.
- Пояснено різницю між початковою операційною пам'яттю і verified state, якщо на сервері існують додаткові `mc-gateway-*` або ширші policy.

## Phase 2: Heartbeat Stability Validation

**Goal:** Підтвердити стабільну роботу кількох `heartbeat` cycles і сформувати базову діагностику деградацій.

**Requirements**:
- MON-01
- MON-02
- MON-03

**Exit Criteria:**
- Зібрано кілька послідовних heartbeat спостережень без збоїв.
- Задокументовано, де дивитися логи та стан сервісів у разі проблем.
- Зрозуміло, чи проблема належить `Mission Control`, gateway або control-plane агенту.

## Phase 3: Board Flow Validation

**Goal:** Перевірити `board creation` і `board lead flow` без побічного впливу на чинне `OpenClaw` середовище.

**Requirements**:
- FLOW-01
- FLOW-02

**Exit Criteria:**
- Обидва сценарії протестовано end-to-end.
- Задокументовано очікувану поведінку і known caveats.
- Не зафіксовано несанкціонованого provisioning за межами dedicated MC workspace root.

### Phase 03.1: Mission Control Onboarding Flow Reliability Hardening (INSERTED)

**Goal:** Усунути silent onboarding stalls у `Mission Control`, щоб board creation flow більше не зависав через `NO_REPLY`, shared heartbeat session або відсутність bounded failure handling.
**Requirements**: FLOW-03, FLOW-04, FLOW-05
**Depends on:** Phase 3
**Plans:** 2 plans

Plans:
- [x] `03.1-01-PLAN.md` - Isolate onboarding session contract from gateway-main silent control-plane traffic
- [x] `03.1-02-PLAN.md` - Add bounded failure state, UI recovery path, and regression coverage for onboarding stalls

**Exit Criteria:**
- `Board onboarding` більше не reuse-ить routine `agent:mc-gateway-...:main` heartbeat semantics без додаткового захисту.
- Після кожної onboarding answer система або зберігає наступне питання / completion payload, або переходить у явний `stalled/failed` state.
- UI більше не зависає безкінечно на optimistic тексті `Usually takes a few seconds`.
- Є regression coverage на кейс, де gateway session завершується `NO_REPLY` без onboarding callback.
- Є операторський маршрут, де видно причину stall і наступний recovery step.

## Phase 4: Policy Hardening For mc-gateway

**Goal:** Спробувати звузити `exec` policy для `mc-gateway-*` після того, як стабільність буде підтверджено.

**Requirements**:
- SECU-01
- SECU-03

**Exit Criteria:**
- Є порівняння поточних потреб control-plane агента з мінімально потрібними дозволами.
- Запропоновано або впроваджено вужчу policy, або задокументовано чому це поки неможливо.

## Phase 5: Governance Of Board Agent Control

**Goal:** Прийняти архітектурне рішення щодо рівня довіри до `Mission Control` у керуванні `board agents`.

**Requirements**:
- SECU-02

**Exit Criteria:**
- Є чітко задокументована policy-позиція: дозволяти, обмежувати або забороняти deeper board-agent management.
- Рішення спирається на результати heartbeat validation, board flow testing і policy hardening.

## Notes

- Phase `03.1` виконана `2026-03-25`; наступний крок: live board creation / onboarding smoke test на задеплоєному коді.
- Якщо спочатку потрібна звірка стану репозиторію та напрямку, використати `$gsd-progress`

---
*Last updated: 2026-03-25 after executing Phase 03.1 onboarding reliability hardening*
