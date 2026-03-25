# Roadmap: Mission Control Operations

**Version:** v1
**Date:** 2026-03-25

## Milestone

Стабілізувати й операційно оформити `Mission Control` як безпечний control-plane sidecar для поточного `OpenClaw gateway`, не зачіпаючи існуючі user workspace-и.

## Phase 1: Project Baseline And Operational Context

**Goal:** Зафіксувати реальний стан інсталяції, інтеграції, ізоляції та тимчасових security exceptions у репозиторії.

**Requirements:**
- DOC-01
- DOC-02
- SAFE-01
- SAFE-02
- SAFE-03
- SAFE-04
- SAFE-05

**Exit Criteria:**
- Базова проектна документація існує локально й відображає поточний verified deployment state.
- Є окремий server context документ з критичними шляхами, URL, ідентифікаторами та technical lessons.
- Є короткий operations runbook для підключення та первинної перевірки системи.

## Phase 2: Heartbeat Stability Validation

**Goal:** Підтвердити стабільну роботу кількох `heartbeat` cycles і сформувати базову діагностику деградацій.

**Requirements:**
- MON-01
- MON-02
- MON-03

**Exit Criteria:**
- Зібрано кілька послідовних heartbeat спостережень без збоїв.
- Задокументовано, де дивитися логи та стан сервісів у разі проблем.
- Зрозуміло, чи проблема належить `Mission Control`, gateway або control-plane агенту.

## Phase 3: Board Flow Validation

**Goal:** Перевірити `board creation` і `board lead flow` без побічного впливу на чинне `OpenClaw` середовище.

**Requirements:**
- FLOW-01
- FLOW-02

**Exit Criteria:**
- Обидва сценарії протестовано end-to-end.
- Задокументовано очікувану поведінку і known caveats.
- Не зафіксовано несанкціонованого provisioning за межами dedicated MC workspace root.

## Phase 4: Policy Hardening For mc-gateway

**Goal:** Спробувати звузити `exec` policy для `mc-gateway-*` після того, як стабільність буде підтверджено.

**Requirements:**
- SECU-01

**Exit Criteria:**
- Є порівняння поточних потреб control-plane агента з мінімально потрібними дозволами.
- Запропоновано або впроваджено вужчу policy, або задокументовано чому це поки неможливо.

## Phase 5: Governance Of Board Agent Control

**Goal:** Прийняти архітектурне рішення щодо рівня довіри до `Mission Control` у керуванні `board agents`.

**Requirements:**
- SECU-02

**Exit Criteria:**
- Є чітко задокументована policy-позиція: дозволяти, обмежувати або забороняти deeper board-agent management.
- Рішення спирається на результати heartbeat validation, board flow testing і policy hardening.

## Notes

- Рекомендований наступний крок у GSD-потоці: `$gsd-plan-phase 1`
- Якщо спочатку потрібна звірка стану репозиторію та напрямку, використати `$gsd-progress`

---
*Last updated: 2026-03-25 after initial roadmap creation*
