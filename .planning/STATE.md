# STATE

## Project Reference

See: `.planning/PROJECT.md` (updated 2026-03-25)

**Core value:** `Mission Control` має бути спостережуваним, ізольованим і керованим так, щоб не ламати основний `OpenClaw` runtime та робочі workspace-и користувача.
**Current focus:** Phase 1 - Project Baseline And Operational Context

## Current Status

- Репозиторій ініціалізовано документацією, але поки без локального коду чи automation-скриптів.
- `2026-03-25` виконано першу фактичну серверну звірку deployment state.
- Підтверджено `BASE_URL=http://backend:8000`, роботу контейнерів `backend/frontend/webhook-worker/db/redis` і повторювані `POST /api/v1/agent/heartbeat` з `200 OK`.
- Виявлено дрейф: у dedicated root існують два `workspace-gateway-*`, а в `openclaw.json` присутній глобальний `tools.exec` fallback з `host=gateway`, `security=full`, `ask=off`.
- Формальна heartbeat-stability перевірка в observation window `2026-03-25 13:31:51 UTC` -> `2026-03-25 13:40:53 UTC` не пройдена: агенти лишалися `online`, але `last_seen_at` не оновлювався і нових `POST /api/v1/agent/heartbeat` у backend logs не з'явилося.

## Known Facts

- Install root: `/home/oc/openclaw-mission-control`
- Dedicated workspace root: `/opt/openclaw/config/mission-control`
- Verified gateway ids in MC workspace naming:
  - `a736f444-d548-448e-9736-11a4516a8735`
  - `b7cc7ef4-c968-4e86-93a2-a63a64335da6`
- Verified OpenClaw agent ids:
  - `mc-gateway-a736f444-d548-448e-9736-11a4516a8735`
  - `mc-gateway-b7cc7ef4-c968-4e86-93a2-a63a64335da6`
- User-facing UI/API: `Tailscale` endpoints on ports `4444/4445`
- Internal backend URL for MC templates: `http://backend:8000`

## Risks

- `Mission Control` не є read-only і може створювати реальні зміни у `OpenClaw`.
- Надто широкий `exec` policy уже підтверджений глобально в `openclaw.json`, а не лише per-agent.
- Другий `workspace-gateway-*` може означати легітимний другий gateway або orphaned provisioning state; це треба окремо пояснити.
- Якщо внутрішній `BASE_URL` дрейфне назад на зовнішній URL, heartbeat/templates можуть знову деградувати.
- `Mission Control` може показувати gateway agents як `online` навіть тоді, коли heartbeat evidence у його власних БД/logs не оновлюється.

## Next Actions

1. Пояснити й зафіксувати MC-side критерій, за яким gateway agent лишається `online` без свіжого heartbeat evidence.
2. Визначити, чи глобальний `tools.exec` fallback потрібен, чи це випадкове розширення policy.
3. Після цього повторити heartbeat verification і лише тоді переходити до board-flow тестів.

## Recent Decisions

- Стартувати проєкт із документації та операційної пам'яті, а не з довільних змін конфігурації.
- Тримати окремий `docs/SERVER_CONTEXT.md` поза `.planning`, щоб server facts було зручно швидко відкривати під час операційної роботи.
- Не вважати початкові server notes остаточною істиною без фактичної звірки через SSH.

---
*Last updated: 2026-03-25 after initial project initialization*
