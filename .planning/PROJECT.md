# Mission Control Operations

## What This Is

Це операційний проєкт для супроводу `OpenClaw Mission Control`, який уже розгорнуто на сервері як окремий sidecar і підключено до наявного `OpenClaw gateway`. Проєкт потрібен для безпечної стабілізації інтеграції, документування реального стану інсталяції та подальшого налаштування/моніторингу без ризику для існуючих user workspaces.

## Core Value

`Mission Control` має бути спостережуваним, ізольованим і керованим так, щоб не ламати основний `OpenClaw` runtime та робочі workspace-и користувача.

## Requirements

### Validated

- ✓ `Mission Control` встановлений як окремий sidecar і вже інтегрований з поточним `OpenClaw gateway`.
- ✓ `Mission Control` використовує той самий `OpenClaw runtime` і той самий стек підключених провайдерів/моделей.
- ✓ Для агентів `Mission Control` використовується окремий ізольований workspace root.
- ✓ На `2026-03-25` підтверджено, що `Mission Control` створює gateway-specific `mc-gateway-*` агентів усередині dedicated workspace root, і heartbeat cycles успішно доходять до backend `200 OK`.

### Active

- [ ] Зафіксувати в репозиторії повний операційний контекст інсталяції, інтеграції та поточних винятків безпеки.
- [ ] Перевірити стабільність кількох `heartbeat` cycles і задокументувати ознаки деградації.
- [ ] Пояснити й задокументувати, чому у dedicated root уже існують два `workspace-gateway-*` та два `mc-gateway-*` агенти.
- [ ] Підтвердити `board creation` та `board lead flow` end-to-end.
- [ ] Після стабілізації звузити policy для `mc-gateway-*` з менш широкими дозволами, ніж поточний `security=full`.
- [ ] Прийняти окреме рішення щодо того, наскільки глибоко `Mission Control` може керувати `board agents` у цьому `OpenClaw` середовищі.

### Out of Scope

- Передавати `Mission Control` існуючі user workspace-и — це порушує ізоляцію та підвищує ризик впливу на `main/reminder/harvester`.
- Робити `Mission Control` просто read-only панеллю — фактична архітектура вже інша, і документація має відображати реальну поведінку provisioning.
- Виносити внутрішній backend URL для MC-generated templates на зовнішній `Tailscale` endpoint — це вже довелося виправляти, і такий підхід нестабільний для heartbeat/templates.

## Context

`Mission Control` розгорнуто на сервері `89.167.61.146` в каталозі `/home/oc/openclaw-mission-control`, а його backend і `webhook-worker` підключені до docker network поточного `OpenClaw`. UI і API доступні через `Tailscale`, але локальні порти лишаються `127.0.0.1:3000` для frontend та `127.0.0.1:8000` для backend.

Головний операційний нюанс у тому, що `Mission Control` реально provision-ить своїх агентів у `OpenClaw`, тому безпечна ізоляція workspace root є критичною вимогою, а не побажанням. Для MC-generated templates і heartbeat внутрішній `BASE_URL` має бути `http://backend:8000`, тоді як user-facing доступ лишається через `Tailscale`.

Під час перевірки `2026-03-25` підтверджено два `workspace-gateway-*` каталоги та два `mc-gateway-*` агенти в `openclaw.json`, а не лише один. Також виявлено, що поряд із per-agent override для `mc-gateway-a736f444-d548-448e-9736-11a4516a8735` у `openclaw.json` присутній і глобальний `tools.exec` fallback із `host=gateway`, `security=full`, `ask=off`.

## Constraints

- **Runtime**: `Mission Control` використовує той самий `OpenClaw runtime` і той самий стек моделей/провайдерів — зміни можуть зачепити спільний gateway.
- **Isolation**: дозволений лише окремий workspace root `/opt/openclaw/config/mission-control` — user workspace-и чіпати не можна.
- **Network**: user-facing URL лишається `Tailscale`, але внутрішній `BASE_URL` для шаблонів і heartbeat має бути `http://backend:8000`.
- **Security**: фактичний стан server-side policy треба вважати ширшим за початкове припущення, бо підтверджено і per-agent override, і глобальний `tools.exec` fallback.
- **Operations**: проєкт поки що стартує з порожнього локального репозиторію, тому вся операційна пам'ять має бути зафіксована документацією до внесення подальших змін.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Тримати `Mission Control` як sidecar, а не окремий ізольований стек | Уже інтегрований з чинним gateway і використовує спільний runtime | ✓ Good |
| Дати `Mission Control` окремий workspace root | Захищає `main/reminder/harvester` від provisioning side effects | ✓ Good |
| Для MC-generated templates використовувати `BASE_URL=http://backend:8000` | Зовнішній `Tailscale` backend URL ламав heartbeat/templates | ✓ Good |
| Вважати Phase 1 незавершеною, доки не пояснено другий `mc-gateway-*` і глобальний `tools.exec` fallback | Verified state уже відрізняється від початкової операційної пам'яті | ✓ Good |
| Застосувати тимчасовий `exec` виняток лише до dedicated `mc-gateway-*` агента | Початкова мета була такою, але фактичний server state ширший | ⚠️ Revisit |
| Зберігати операційний контекст у цьому репозиторії перед подальшим hardening | Репозиторій був порожній, а критичні знання існували тільки в діалозі | — Pending |

---
*Last updated: 2026-03-25 after initial project initialization*
