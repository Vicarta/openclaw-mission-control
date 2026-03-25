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
- Verified on `2026-03-25`: two `mc-gateway-*` workspaces/agents currently exist under the dedicated workspace root
- Known gateway/agent:
  - `a736f444-d548-448e-9736-11a4516a8735` / `mc-gateway-a736f444-d548-448e-9736-11a4516a8735`
  - `b7cc7ef4-c968-4e86-93a2-a63a64335da6` / `mc-gateway-b7cc7ef4-c968-4e86-93a2-a63a64335da6`

## Operating Principles

- `Mission Control` не read-only: він реально provision-ить агентів у `OpenClaw`.
- Для `Mission Control` дозволений тільки окремий workspace root, без доступу до існуючих `main/reminder/harvester`.
- Внутрішній `BASE_URL` для MC-generated templates має лишатися `http://backend:8000`.
- Потрібно вважати `tools.exec` policy недовіреною зоною до окремої перевірки: на сервері підтверджено не лише per-agent override, а й глобальний `tools.exec` fallback з `host=gateway`, `security=full`, `ask=off`.

## Immediate Priorities

1. Підтвердити стабільність кількох `heartbeat` cycles.
2. Протестувати `board creation` і `board lead flow`.
3. Спробувати звузити policy для `mc-gateway-*` після стабілізації.
4. Оцінити, чи варто дозволяти `Mission Control` глибше керувати `board agents`.
