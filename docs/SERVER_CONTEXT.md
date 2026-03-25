# Server Context

## Purpose

Цей документ фіксує поточний відомий стан інсталяції `OpenClaw Mission Control` на сервері та служить точкою істини для подальших змін у цьому репозиторії.

## Access

- SSH from this workstation: `ssh -i ~/.ssh/oc_hetzner_oc oc@89.167.61.146`
- UI: `https://ubuntu-oc.tailbd4e1c.ts.net:4444`
- API: `https://ubuntu-oc.tailbd4e1c.ts.net:4445`

## Install Layout

- Install root: `/home/oc/openclaw-mission-control`
- Install manifest: `/home/oc/openclaw-mission-control/.ops/install-manifest.txt`
- Uninstall script: `/home/oc/openclaw-mission-control/uninstall.sh`
- Important environment file: `/home/oc/openclaw-mission-control/.env`
- Compose files:
  - `/home/oc/openclaw-mission-control/compose.yml`
  - `/home/oc/openclaw-mission-control/compose.override.yml`

## Runtime Endpoints

- Frontend local bind: `127.0.0.1:3000`
- Backend local bind: `127.0.0.1:8000`
- Mission Control <-> OpenClaw gateway websocket: `ws://openclaw-gateway:18789`

## Workspace Isolation

- Dedicated host workspace root: `/opt/openclaw/config/mission-control`
- Dedicated container workspace root: `/home/node/.openclaw/mission-control`
- Verified workspaces on `2026-03-25`:
  - `/opt/openclaw/config/mission-control/workspace-gateway-a736f444-d548-448e-9736-11a4516a8735`
  - `/opt/openclaw/config/mission-control/workspace-gateway-b7cc7ef4-c968-4e86-93a2-a63a64335da6`

## Identifiers

- Verified gateway/agent pairs on `2026-03-25`:
  - `a736f444-d548-448e-9736-11a4516a8735` / `mc-gateway-a736f444-d548-448e-9736-11a4516a8735` / `OpenClaw Sandbox Gateway Agent`
  - `b7cc7ef4-c968-4e86-93a2-a63a64335da6` / `mc-gateway-b7cc7ef4-c968-4e86-93a2-a63a64335da6` / `MissionControl01 Gateway Agent`

## Confirmed Working State

- `Mission Control` встановлений як окремий sidecar.
- Він уже інтегрований з поточним `OpenClaw gateway`, а не працює окремо.
- Він використовує той самий `OpenClaw runtime` і той самий стек підключених провайдерів/моделей.
- `backend` і `webhook-worker` запущені та підключені і до `openclaw-mission-control_default`, і до `app_default`.
- `BASE_URL` у `.env` дорівнює `http://backend:8000`.
- Provisioning не вийшов за межі dedicated root `/opt/openclaw/config/mission-control`.
- Backend `Mission Control` приймає повторювані `POST /api/v1/agent/heartbeat` з `200 OK`.
- У worker logs є принаймні один `status=online` для lifecycle reconcile.

## Important Technical Lessons

1. `Mission Control` не є read-only панеллю; він реально provision-ить своїх агентів у `OpenClaw`.
2. Не можна давати йому існуючий user workspace; тільки окремий root.
3. Для `OpenClaw`-side heartbeat/templates не можна використовувати зовнішній `Tailscale` backend URL.
4. User-facing URL може лишатися `Tailscale`, але внутрішній `BASE_URL` для MC-generated templates має бути `http://backend:8000`.
5. `Mission Control gateway-main agent` зараз залежить від `curl` у `HEARTBEAT.md` і `TOOLS.md`.

## Current Practical Fix

У файлі `/opt/openclaw/config/openclaw.json` підтверджено per-agent override для:

- `mc-gateway-a736f444-d548-448e-9736-11a4516a8735`

Для нього задано:

- `tools.exec.host = gateway`
- `tools.exec.ask = off`
- `tools.exec.security = full`

Однак це не повна картина: у тому ж файлі також присутній глобальний fallback:

- `tools.exec.host = gateway`
- `tools.exec.ask = off`
- `tools.exec.security = full`

Отже, поточний server state ширший за початкову гіпотезу про суто локальний виняток і має бути предметом окремого hardening.

## Additional Completed Work

- Device pairing для `Mission Control backend` схвалений.
- Templates пересинхронізовані після token rotation.
- Backend `.env` переведений на `BASE_URL=http://backend:8000`.

## Files To Watch

- `/home/oc/openclaw-mission-control/.env`
- `/home/oc/openclaw-mission-control/compose.yml`
- `/home/oc/openclaw-mission-control/compose.override.yml`
- `/opt/openclaw/config/openclaw.json`
- `/opt/openclaw/config/mission-control/workspace-gateway-a736f444-d548-448e-9736-11a4516a8735/AGENTS.md`
- `/opt/openclaw/config/mission-control/workspace-gateway-a736f444-d548-448e-9736-11a4516a8735/TOOLS.md`
- `/opt/openclaw/config/mission-control/workspace-gateway-a736f444-d548-448e-9736-11a4516a8735/HEARTBEAT.md`

## Open Work

1. Перевірити стабільність кількох `heartbeat` cycles.
2. Протестувати `board creation` і `board lead flow`.
3. Пояснити походження другого `workspace-gateway-*` / `mc-gateway-*`.
4. Звузити policy для `mc-gateway-*` і перевірити, чи глобальний `tools.exec` взагалі потрібен.
5. Оцінити, чи варто дозволяти `Mission Control` глибше керувати `board agents`.

---
*Updated on 2026-03-25 after SSH verification against live server state*
