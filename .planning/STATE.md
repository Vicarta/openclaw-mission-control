# STATE

## Project Reference

See: `.planning/PROJECT.md` (updated 2026-03-25)

**Core value:** `Mission Control` має бути спостережуваним, ізольованим і керованим так, щоб не ламати основний `OpenClaw` runtime та робочі workspace-и користувача.
**Current focus:** Post-deploy validation after Phase 03.1 onboarding reliability hardening

## Current Status

- Репозиторій ініціалізовано документацією, але поки без локального коду чи automation-скриптів.
- `2026-03-25` виконано першу фактичну серверну звірку deployment state.
- Підтверджено `BASE_URL=http://backend:8000`, роботу контейнерів `backend/frontend/webhook-worker/db/redis` і повторювані `POST /api/v1/agent/heartbeat` з `200 OK`.
- Виявлено дрейф: у dedicated root існують два `workspace-gateway-*`, а в `openclaw.json` присутній глобальний `tools.exec` fallback з `host=gateway`, `security=full`, `ask=off`.
- Формальна heartbeat-stability перевірка в observation window `2026-03-25 13:31:51 UTC` -> `2026-03-25 13:40:53 UTC` не пройдена: агенти лишалися `online`, але `last_seen_at` не оновлювався і нових `POST /api/v1/agent/heartbeat` у backend logs не з'явилося.
- Під час live `board onboarding` перевірки board `Money` створився успішно, але flow завис після другої user answer.
- Root cause був на MC-side: onboarding dispatch reuse-ив `GatewayAgentIdentity.session_key(gateway)` (`agent:mc-gateway-...:main`) з `deliver=False`, а gateway session завершував turn як `NO_REPLY` замість нового onboarding callback.
- `2026-03-25` виконано всю Phase `03.1`:
  - onboarding start dispatch переведено на dedicated board-scoped session key `agent:mc-gateway-<gateway_id>:onboarding:<board_id>`;
  - onboarding prompt тепер прямо забороняє `NO_REPLY` як валідне завершення turn;
  - `GET /api/v1/boards/{id}/onboarding` повертає computed `stalled` state, якщо MC чекає callback від агента довше 20 секунд після останнього `user` message;
  - frontend більше не poll-ить безкінечно на stalled state і показує `Retry last step`.
- Для перевірки Phase `03.1` пройдено:
  - backend unit tests: `11 passed`;
  - frontend component tests: `3 passed`;
  - production rebuild `backend + webhook-worker + frontend`;
  - smoke-check після deploy: локальні `http://127.0.0.1:8000/openapi.json` і `http://127.0.0.1:3000/` повернули `200`.

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
- Interactive onboarding тепер має bounded stalled path, але ще не пройдений новий live board-onboarding smoke test після deploy.

## Next Actions

1. Повторити live board creation / onboarding smoke test уже на задеплоєному коді й підтвердити один з трьох фіналів: next question, `status=complete`, або `stalled`.
2. Якщо smoke test зелений, повернутися до основної Phase 3 verification і зафіксувати очікувану поведінку board flow.
3. Після цього вирішувати, чи є сенс рухатись у policy hardening.

## Recent Decisions

- Стартувати проєкт із документації та операційної пам'яті, а не з довільних змін конфігурації.
- Тримати окремий `docs/SERVER_CONTEXT.md` поза `.planning`, щоб server facts було зручно швидко відкривати під час операційної роботи.
- Не вважати початкові server notes остаточною істиною без фактичної звірки через SSH.
- Не змішувати onboarding reliability bugfixing з `OpenClaw` policy/workspace питаннями; вести це як окрему Phase `03.1`.
- Не додавати bounded failure handling через міграцію БД, доки computed read-state покриває кейс безпечніше і простіше.

## Accumulated Context

### Roadmap Evolution

- Phase `03.1` inserted after Phase `3`: Mission Control Onboarding Flow Reliability Hardening (URGENT)
- Phase `03.1` executed and deployed on `2026-03-25`

---
*Last updated: 2026-03-25 after executing and deploying Phase 03.1*
