# STATE

## Project Reference

See: `.planning/PROJECT.md` (updated 2026-03-25)

**Core value:** `Mission Control` має бути спостережуваним, ізольованим і керованим так, щоб не ламати основний `OpenClaw` runtime та робочі workspace-и користувача.
**Current focus:** Phase 1 - Project Baseline And Operational Context

## Current Status

- Репозиторій ініціалізовано документацією, але поки без локального коду чи automation-скриптів.
- Поточний deployment state зафіксовано зі слів оператора й має бути надалі звірений під час фактичних серверних перевірок.
- Найважливіший тимчасовий виняток: `tools.exec.ask=off` і `tools.exec.security=full` лише для `mc-gateway-a736f444-d548-448e-9736-11a4516a8735`.

## Known Facts

- Install root: `/home/oc/openclaw-mission-control`
- Dedicated workspace root: `/opt/openclaw/config/mission-control`
- Current gateway id: `a736f444-d548-448e-9736-11a4516a8735`
- Current agent id: `mc-gateway-a736f444-d548-448e-9736-11a4516a8735`
- User-facing UI/API: `Tailscale` endpoints on ports `4444/4445`
- Internal backend URL for MC templates: `http://backend:8000`

## Risks

- `Mission Control` не є read-only і може створювати реальні зміни у `OpenClaw`.
- Надто широкий `exec` policy для control-plane агента може стати постійним боргом без окремого hardening.
- Якщо внутрішній `BASE_URL` дрейфне назад на зовнішній URL, heartbeat/templates можуть знову деградувати.

## Next Actions

1. Запланувати й виконати Phase 1 verification на сервері.
2. Після цього перейти до Phase 2 і зібрати кілька heartbeat observations.
3. Підготувати Phase 3 для board flows лише після підтвердження стабільності heartbeat.

## Recent Decisions

- Стартувати проєкт із документації та операційної пам'яті, а не з довільних змін конфігурації.
- Тримати окремий `docs/SERVER_CONTEXT.md` поза `.planning`, щоб server facts було зручно швидко відкривати під час операційної роботи.

---
*Last updated: 2026-03-25 after initial project initialization*
