# Design: Printfarm Dashboard

**Date:** 2026-02-26
**Status:** Approved

## Overview

A self-hosted web dashboard for monitoring and controlling all printers in the farm from a single browser tab. Aggregates real-time state from each printer's Moonraker API, provides unified controls, and sends Telegram alerts on key events.

Deployed via Docker Compose on a dedicated Raspberry Pi. Accessible locally and remotely via VPN (Tailscale).

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│               Dedicated Pi (Docker)              │
│                                                  │
│  ┌──────────────────┐    ┌────────────────────┐  │
│  │  FastAPI backend  │    │  React frontend    │  │
│  │                  │◄──►│  (nginx, port 80)  │  │
│  │  - WS server     │    └────────────────────┘  │
│  │  - REST API      │                            │
│  │  - Telegram bot  │                            │
│  └────────┬─────────┘                            │
└───────────┼──────────────────────────────────────┘
            │  WebSocket (one per printer)
            ▼
   ┌────────────────────────────────────────┐
   │  Moonraker instances (your printers)   │
   │  Twin 1 :7125  Twin 2 :7125           │
   │  Ender 3 Pro   Neptune 3 Max          │
   │  Ender 7       Wanhao D6  ...         │
   └────────────────────────────────────────┘
```

### Data flow

1. On startup the backend opens one persistent WebSocket per printer to Moonraker and subscribes to: `print_stats`, `heaters`, `virtual_sdcard`, `display_status`, `fan`
2. Moonraker pushes state diffs in real-time; the backend merges these into an in-memory state object per printer
3. The browser connects to the backend via a single WebSocket and receives a full snapshot on connect, then incremental diffs as things change
4. User actions (pause, cancel, emergency stop) POST to the backend which forwards the gcode command to the appropriate Moonraker REST endpoint (`POST /printer/gcode/script`)
5. Telegram bot runs in the same process, triggered by printer state transitions

### Printer configuration

A `printers.yaml` file mounted as a Docker volume — one entry per printer. Adding a printer requires only a new YAML entry, no code changes.

```yaml
printers:
  - name: "Ender 3 V2 Twin 1"
    moonraker_url: "http://192.168.x.x:7125"
    camera_url: "http://192.168.x.x:8080"
  - name: "Neptune 3 Max"
    moonraker_url: "http://192.168.x.x:7125"
    camera_url: "http://192.168.x.x:8080"
  # ...

telegram:
  bot_token: "..."
  chat_id: "..."
  quiet_hours:
    start: "23:00"
    end: "07:00"
```

---

## Tech Stack

| Layer | Choice | Reason |
|-------|--------|--------|
| Backend | Python 3.11 + FastAPI | Async WebSocket support, familiar in Klipper ecosystem |
| WS client | `websockets` library | Persistent connections to Moonraker |
| Telegram | `python-telegram-bot` | Well-maintained, async-native |
| Frontend | React 18 + TypeScript + Vite | Component-based, strong real-time update patterns |
| Styling | Tailwind CSS | Rapid responsive layout, no custom CSS overhead |
| Deployment | Docker Compose | Single command deploy, volume-mounted config |
| Web server | nginx | Serves built React app, proxies `/api` to backend |

---

## UI Design

### Farm header (top bar)

Global summary across all printers:

```
PrintFarm  |  7 printers · 3 printing · 1 paused · 3 idle  |  [⛔ EMERGENCY STOP ALL]
```

### Printer card grid

Responsive grid: 2 columns desktop, 1 column mobile.

```
┌─────────────────────────────────┐
│ 🟢 Neptune 3 Max          45:23 │  ← status dot + ETA
│─────────────────────────────────│
│ [webcam thumbnail / stream]     │
│─────────────────────────────────│
│ benchy.gcode              67%   │
│ ████████████░░░░░░░░            │
│─────────────────────────────────│
│ 🌡 235° / 235°   🛏 65° / 65°  │
│─────────────────────────────────│
│ [Pause]  [Cancel]  [Mainsail ↗] │
└─────────────────────────────────┘
```

**Status dot states:**
- 🟢 Printing
- 🟡 Paused
- ⚪ Idle / Standby
- 🔴 Error
- ⛔ Offline / Unreachable

### Detail view

Clicking a card expands to a detail panel with:
- Full temperature history graphs (nozzle + bed, last 30 min)
- Gcode console (send arbitrary commands)
- Current file metadata (layer, print time elapsed/remaining)
- "Open in Mainsail ↗" for anything more advanced

---

## Alerts (Telegram)

| Event | Message | Suppressed in quiet hours? |
|-------|---------|---------------------------|
| Print complete | ✅ **Neptune 3 Max** — `benchy.gcode` done (1h 23m) | Yes |
| Print failed / cancelled | ❌ **Ender 7** — print cancelled: `vase.gcode` | No |
| Printer goes offline | ⛔ **Wanhao D6** — lost connection | No |
| Printer comes back online | 🟢 **Wanhao D6** — reconnected | Yes |
| Temperature anomaly | 🌡 **Ender 3 Pro** — bed temp dropped 20°C mid-print | No |

Quiet hours suppress non-critical notifications (complete, reconnected) to avoid 3am wakeups.

---

## Repo Structure

New standalone repo: `printfarm-dashboard` (separate from this config repo).

```
printfarm-dashboard/
  backend/
    main.py           # FastAPI app, WebSocket server
    moonraker.py      # WebSocket client per printer, state subscription
    state.py          # Aggregated in-memory state store
    telegram.py       # Bot + alert logic
    config.py         # printers.yaml loader + validation
    requirements.txt
  frontend/
    src/
      components/
        PrinterCard.tsx
        FarmHeader.tsx
        DetailView.tsx
        TempGraph.tsx
      App.tsx
      main.tsx
    package.json
    vite.config.ts
  printers.yaml        # Gitignored — contains IPs/tokens
  printers.example.yaml
  docker-compose.yml
  Dockerfile.backend
  Dockerfile.frontend
  .gitignore
```

---

## Deployment

```bash
# On the dedicated Pi:
git clone https://github.com/drupi79/printfarm-dashboard
cp printers.example.yaml printers.yaml
# edit printers.yaml with IPs and Telegram token
docker-compose up -d
```

Dashboard available at `http://<pi-ip>` locally and via Tailscale VPN remotely.

---

## Out of Scope (v1)

- Print queue / job scheduling
- File upload / management (use Mainsail for this)
- Webcam recording / timelapse (handled by Moonraker timelapse plugin)
- Multi-user auth (VPN provides access control)
- K1 Max integration (no Moonraker — would need Creality API)
