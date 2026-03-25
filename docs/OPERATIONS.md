# Operations Runbook

## Purpose

Короткий runbook для підключення до сервера, перевірки стану `Mission Control` і швидкої діагностики перед наступними змінами.

## Connect

```bash
ssh -i ~/.ssh/oc_hetzner__oc oc@89.167.61.146
```

## Navigate

```bash
cd /home/oc/openclaw-mission-control
```

## Key Files

- `.env`
- `compose.yml`
- `compose.override.yml`
- `.ops/install-manifest.txt`
- `uninstall.sh`
- `/opt/openclaw/config/openclaw.json`
- `/opt/openclaw/config/mission-control/workspace-gateway-a736f444-d548-448e-9736-11a4516a8735/AGENTS.md`
- `/opt/openclaw/config/mission-control/workspace-gateway-a736f444-d548-448e-9736-11a4516a8735/TOOLS.md`
- `/opt/openclaw/config/mission-control/workspace-gateway-a736f444-d548-448e-9736-11a4516a8735/HEARTBEAT.md`

## First Checks

```bash
docker compose ps
```

```bash
docker compose logs --tail=200 backend
```

```bash
docker compose logs --tail=200 webhook-worker
```

## Verify Internal Configuration

Переконатися, що backend використовує внутрішній URL:

```bash
grep '^BASE_URL=' /home/oc/openclaw-mission-control/.env
```

Очікуване значення:

```text
BASE_URL=http://backend:8000
```

Переконатися, що workspace root лишається ізольованим:

```bash
ls -la /opt/openclaw/config/mission-control
```

Перевірити локальний виняток `mc-gateway-*` в `openclaw.json`:

```bash
rg -n "mc-gateway|tools\\.exec|security|ask|host" /opt/openclaw/config/openclaw.json
```

## Heartbeat Validation Checklist

1. Переконатися, що `backend` і `webhook-worker` запущені.
2. Переглянути логи backend на наявність `POST /api/v1/agent/heartbeat`.
3. Переконатися, що агент `mc-gateway-a736f444-d548-448e-9736-11a4516a8735` лишається `online` у UI/API.
4. Повторити перевірку через кілька циклів, а не лише один раз.

## Board Flow Validation Checklist

1. Створити тестовий board у `Mission Control`.
2. Переконатися, що provisioning не виходить за межі `/opt/openclaw/config/mission-control`.
3. Запустити `board lead flow`.
4. Перевірити, чи не з'явилися несподівані агенти або зміни у user workspace-ах.

## If Heartbeat Breaks

1. Перевірити `BASE_URL` у `/home/oc/openclaw-mission-control/.env`.
2. Перевірити доступність `ws://openclaw-gateway:18789` з контейнерного оточення `Mission Control`.
3. Переглянути `AGENTS.md`, `TOOLS.md`, `HEARTBEAT.md` у dedicated workspace агента.
4. Перевірити, чи не було змінено локальний `tools.exec` виняток в `/opt/openclaw/config/openclaw.json`.
5. Звірити, чи не зламалися pairing, token rotation або template sync.

## Guardrails

- Не використовувати існуючі user workspace-и для `Mission Control`.
- Не поширювати `tools.exec.ask=off` і `tools.exec.security=full` на інших агентів без окремого рішення.
- Не міняти внутрішній `BASE_URL` назад на зовнішній `Tailscale` endpoint.
