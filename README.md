# Mission Control

Операційний репозиторій для подальшого налаштування, стабілізації та моніторингу `OpenClaw Mission Control`, який уже розгорнуто як sidecar на продуктивному сервері та інтегровано з поточним `OpenClaw gateway`.

Цей репозиторій фіксує поточний стан інсталяції, обмеження інтеграції, вимоги до безпечної експлуатації та дорожню карту наступних перевірок. Тут має жити вся документація, runbook-и, майбутні automation-и та конфігураційні зміни, пов'язані саме з `Mission Control`.

## Start Here

- [Project Context](./.planning/PROJECT.md)
- [Requirements](./.planning/REQUIREMENTS.md)
- [Roadmap](./.planning/ROADMAP.md)
- [Current State](./.planning/STATE.md)
- [Server Context](./docs/SERVER_CONTEXT.md)
- [Operations Runbook](./docs/OPERATIONS.md)

## Current Deployment Snapshot

- Server SSH: `ssh -i ~/.ssh/oc_hetzner__oc oc@89.167.61.146`
- UI: `https://ubuntu-oc.tailbd4e1c.ts.net:4444`
- API: `https://ubuntu-oc.tailbd4e1c.ts.net:4445`
- Mission Control install root: `/home/oc/openclaw-mission-control`
- Dedicated workspace root: `/opt/openclaw/config/mission-control`
- Mission Control gateway id: `a736f444-d548-448e-9736-11a4516a8735`
- Mission Control agent id: `mc-gateway-a736f444-d548-448e-9736-11a4516a8735`

## Operating Principles

- `Mission Control` не read-only: він реально provision-ить агентів у `OpenClaw`.
- Для `Mission Control` дозволений тільки окремий workspace root, без доступу до існуючих `main/reminder/harvester`.
- Внутрішній `BASE_URL` для MC-generated templates має лишатися `http://backend:8000`.
- Тимчасове розширення `exec` policy дозволене лише для dedicated `mc-gateway-*` control-plane агента.

## Immediate Priorities

1. Підтвердити стабільність кількох `heartbeat` cycles.
2. Протестувати `board creation` і `board lead flow`.
3. Спробувати звузити policy для `mc-gateway-*` після стабілізації.
4. Оцінити, чи варто дозволяти `Mission Control` глибше керувати `board agents`.
