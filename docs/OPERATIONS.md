# Operations Runbook

## Purpose

Короткий runbook для підключення до сервера, перевірки стану `Mission Control` і швидкої діагностики перед наступними змінами.

## Connect

```bash
ssh -i ~/.ssh/oc_hetzner_oc oc@89.167.61.146
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

Перевірити `mc-gateway-*` і `tools.exec` в `openclaw.json`:

```bash
grep -n -C 3 'mc-gateway-\|tools.exec\|security\|ask\|host' /opt/openclaw/config/openclaw.json
```

## Heartbeat Validation Checklist

1. Переконатися, що `backend` і `webhook-worker` запущені.
2. Переглянути логи backend на наявність `POST /api/v1/agent/heartbeat`.
3. Переконатися, що агент `mc-gateway-a736f444-d548-448e-9736-11a4516a8735` лишається `online` у UI/API.
4. Повторити перевірку через кілька циклів, а не лише один раз.
5. Звірити, чи не з'явилися нові `workspace-gateway-*` у `/opt/openclaw/config/mission-control`.

## Board Flow Validation Checklist

1. Створити тестовий board у `Mission Control`.
2. Переконатися, що provisioning не виходить за межі `/opt/openclaw/config/mission-control`.
3. Запустити `board lead flow`.
4. Перевірити, чи не з'явилися несподівані агенти або зміни у user workspace-ах.

## Onboarding Stall Diagnosis

Ознака проблеми після Phase `03.1`:

- UI більше не має висіти безкінечно на `Usually takes a few seconds`.
- Якщо gateway не повернув next question або completion payload, onboarding має перейти у `stalled`.

Що перевіряти:

```bash
docker compose logs --tail=200 backend | grep 'onboarding'
```

```bash
docker compose logs --tail=200 webhook-worker
```

Ознаки confirmed stall:

1. Board створений, але `GET /api/v1/boards/{board_id}/onboarding` лишається без нового assistant payload.
2. Останнє повідомлення в onboarding transcript має `role=user`.
3. Від цього `user` message минуло понад 20 секунд.
4. API/UI повертає `status=stalled` і пропонує `Retry last step`.

Що означає `Retry last step`:

- frontend повторно викликає `POST /api/v1/boards/{board_id}/onboarding/start`;
- backend редиспачить останню user answer у dedicated onboarding session;
- це безпечний recovery path для MC-side stalls без ручного редагування БД.

Як відрізнити stall від успішного прогресу:

- Якщо onboarding healthy, після user answer з'являється або нове assistant question payload, або `status=complete`.
- Якщо board draft уже зібраний, session переходить у `completed`, а не `stalled`.
- Якщо `status=stalled`, проблема вже не маскується під normal waiting.

## If Heartbeat Breaks

1. Перевірити `BASE_URL` у `/home/oc/openclaw-mission-control/.env`.
2. Перевірити доступність `ws://openclaw-gateway:18789` з контейнерного оточення `Mission Control`.
3. Переглянути `AGENTS.md`, `TOOLS.md`, `HEARTBEAT.md` у dedicated workspace агента.
4. Перевірити, чи не було змінено per-agent або глобальний `tools.exec` в `/opt/openclaw/config/openclaw.json`.
5. Звірити, чи не зламалися pairing, token rotation або template sync.

## Guardrails

- Не використовувати існуючі user workspace-и для `Mission Control`.
- Не вважати `tools.exec.ask=off` і `tools.exec.security=full` локальним винятком, доки це не підтверджено в `openclaw.json`.
- Не міняти внутрішній `BASE_URL` назад на зовнішній `Tailscale` endpoint.
- Не дебажити stalled onboarding тільки по UI; завжди звіряти backend logs, останній onboarding transcript і факт `status=stalled`.
