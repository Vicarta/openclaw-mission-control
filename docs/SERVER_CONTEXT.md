# Server Context

## Purpose

Цей документ фіксує поточний відомий стан інсталяції `OpenClaw Mission Control` на сервері та служить точкою істини для подальших змін у цьому репозиторії.

## Access

- SSH: `ssh -i ~/.ssh/oc_hetzner__oc oc@89.167.61.146`
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
- Mission Control agent workspace:
  `/opt/openclaw/config/mission-control/workspace-gateway-a736f444-d548-448e-9736-11a4516a8735`

## Identifiers

- Gateway id: `a736f444-d548-448e-9736-11a4516a8735`
- Gateway name: `OpenClaw Sandbox`
- Mission Control OpenClaw agent id: `mc-gateway-a736f444-d548-448e-9736-11a4516a8735`

## Confirmed Working State

- `Mission Control` встановлений як окремий sidecar.
- Він уже інтегрований з поточним `OpenClaw gateway`, а не працює окремо.
- Він використовує той самий `OpenClaw runtime` і той самий стек підключених провайдерів/моделей.
- Provisioning не чіпає існуючі user workspace-и.
- `Mission Control` створює тільки свого `mc-gateway-*` агента.
- Heartbeat цього агента вже успішно дійшов.
- Backend `Mission Control` прийняв `POST /api/v1/agent/heartbeat`.
- Агент у `Mission Control` перейшов у `online`.

## Important Technical Lessons

1. `Mission Control` не є read-only панеллю; він реально provision-ить своїх агентів у `OpenClaw`.
2. Не можна давати йому існуючий user workspace; тільки окремий root.
3. Для `OpenClaw`-side heartbeat/templates не можна використовувати зовнішній `Tailscale` backend URL.
4. User-facing URL може лишатися `Tailscale`, але внутрішній `BASE_URL` для MC-generated templates має бути `http://backend:8000`.
5. `Mission Control gateway-main agent` зараз залежить від `curl` у `HEARTBEAT.md` і `TOOLS.md`.

## Current Practical Fix

Локальний виняток зроблено лише для:

- `mc-gateway-a736f444-d548-448e-9736-11a4516a8735`

У файлі `/opt/openclaw/config/openclaw.json` для нього задано:

- `tools.exec.host = gateway`
- `tools.exec.ask = off`
- `tools.exec.security = full`

Це не глобальна policy і має бути предметом окремого hardening після стабілізації.

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
3. Спробувати звузити policy для `mc-gateway-*` після стабілізації.
4. Оцінити, чи варто дозволяти `Mission Control` глибше керувати `board agents`.

---
*Captured on 2026-03-25 from current operator context*
