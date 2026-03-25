# Verification 2026-03-25

## Scope

Первинна SSH-звірка live server state для `OpenClaw Mission Control` після стартової ініціалізації цього репозиторію.

## Verified

- Сервер доступний по SSH як `oc@89.167.61.146`.
- Host: `ubuntu-OC`
- OS: `Ubuntu 24.04`
- Docker: `29.2.1`
- Docker Compose: `v5.0.2`
- Install root існує: `/home/oc/openclaw-mission-control`
- Файли `.env`, `compose.yml`, `compose.override.yml`, `uninstall.sh`, `.ops/install-manifest.txt` присутні.
- `docker compose ps` показує запущені `backend`, `frontend`, `webhook-worker`, `db`, `redis`.
- `BASE_URL` у `.env` дорівнює `http://backend:8000`.
- `backend` і `webhook-worker` приєднані як до `openclaw-mission-control_default`, так і до `app_default`.
- У backend logs видно численні `POST /api/v1/agent/heartbeat` з `200 OK`.
- У worker logs є lifecycle reconcile із `status=online`.

## Findings

### 1. Dedicated workspace root містить два gateway workspaces

Підтверджені каталоги:

- `/opt/openclaw/config/mission-control/workspace-gateway-a736f444-d548-448e-9736-11a4516a8735`
- `/opt/openclaw/config/mission-control/workspace-gateway-b7cc7ef4-c968-4e86-93a2-a63a64335da6`

Другий workspace був bootstrap-нутий пізніше того ж дня. Це суперечить початковій операційній пам'яті про єдиний `mc-gateway-*` agent/workspace і потребує пояснення.

### 2. `openclaw.json` має не лише per-agent override, а й глобальний `tools.exec`

Підтверджено:

- per-agent override для `mc-gateway-a736f444-d548-448e-9736-11a4516a8735`
- окремий глобальний `tools.exec` з `host=gateway`, `security=full`, `ask=off`

Це означає, що поточна policy поверхня ширша, ніж очікувалося, а другий `mc-gateway-*` агент, імовірно, покладається саме на глобальну policy.

## Immediate Follow-up

1. З'ясувати походження другого gateway/agent pair.
2. Перевірити, чи глобальний `tools.exec` був свідомим рішенням або тимчасовим practical fix.
3. Лише після цього переходити до policy hardening, щоб не зламати heartbeat/template flow.

---
*Generated from live SSH verification on 2026-03-25*
