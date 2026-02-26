# Printfarm Dashboard Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a self-hosted web dashboard that aggregates real-time state from all Klipper printers via Moonraker WebSocket APIs, displays them in a unified card grid, supports basic controls, and sends Telegram alerts.

**Architecture:** FastAPI backend maintains one persistent Moonraker WebSocket per printer, merges state into an in-memory store, and fans updates out to the browser via a single WebSocket. React frontend renders a grid of printer cards updated in real-time. Telegram bot triggers on state transitions.

**Tech Stack:** Python 3.11, FastAPI, `websockets`, `python-telegram-bot` v20, React 18, TypeScript, Vite, Tailwind CSS, Docker Compose, nginx.

---

## Repo Setup

This is a **new standalone repo** at `~/printfarm-dashboard/` (separate from the config repo).

```bash
mkdir ~/printfarm-dashboard && cd ~/printfarm-dashboard
git init
```

---

### Task 1: Project Scaffolding

**Files:**
- Create: `backend/requirements.txt`
- Create: `frontend/package.json`
- Create: `printers.example.yaml`
- Create: `.gitignore`
- Create: `README.md`

**Step 1: Create backend requirements**

```
# backend/requirements.txt
fastapi==0.115.0
uvicorn[standard]==0.30.6
websockets==13.0
python-telegram-bot==21.5
pydantic==2.9.0
pyyaml==6.0.2
pytest==8.3.3
pytest-asyncio==0.24.0
httpx==0.27.2
```

**Step 2: Create frontend package.json**

```json
{
  "name": "printfarm-dashboard",
  "version": "0.1.0",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "test": "vitest run",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "recharts": "^2.12.7"
  },
  "devDependencies": {
    "@types/react": "^18.3.5",
    "@types/react-dom": "^18.3.0",
    "@vitejs/plugin-react": "^4.3.1",
    "@testing-library/react": "^16.0.1",
    "@testing-library/jest-dom": "^6.5.0",
    "autoprefixer": "^10.4.20",
    "postcss": "^8.4.47",
    "tailwindcss": "^3.4.12",
    "typescript": "^5.5.4",
    "vite": "^5.4.7",
    "vitest": "^2.1.1",
    "jsdom": "^25.0.1"
  }
}
```

**Step 3: Create example config**

```yaml
# printers.example.yaml
printers:
  - id: twin1
    name: "Ender 3 V2 Twin 1"
    moonraker_url: "http://192.168.1.100:7125"
    camera_url: "http://192.168.1.100:8080/?action=stream"
  - id: neptune
    name: "Neptune 3 Max"
    moonraker_url: "http://192.168.1.101:7125"
    camera_url: "http://192.168.1.101:8080/?action=stream"

telegram:
  bot_token: "YOUR_BOT_TOKEN"
  chat_id: "YOUR_CHAT_ID"
  quiet_hours:
    start: "23:00"
    end: "07:00"
```

**Step 4: Create .gitignore**

```
printers.yaml
__pycache__/
*.pyc
.pytest_cache/
node_modules/
dist/
.env
```

**Step 5: Commit**

```bash
git add .
git commit -m "feat: project scaffolding"
```

---

### Task 2: Config Loader

**Files:**
- Create: `backend/config.py`
- Create: `backend/tests/test_config.py`

**Step 1: Write failing tests**

```python
# backend/tests/test_config.py
import pytest
from pathlib import Path
from backend.config import load_config, Config, PrinterConfig

SAMPLE_YAML = """
printers:
  - id: test_printer
    name: "Test Printer"
    moonraker_url: "http://localhost:7125"
    camera_url: "http://localhost:8080/?action=stream"
telegram:
  bot_token: "abc123"
  chat_id: "456"
  quiet_hours:
    start: "23:00"
    end: "07:00"
"""

def test_load_config_parses_printers(tmp_path):
    cfg_file = tmp_path / "printers.yaml"
    cfg_file.write_text(SAMPLE_YAML)
    config = load_config(cfg_file)
    assert len(config.printers) == 1
    assert config.printers[0].id == "test_printer"
    assert config.printers[0].moonraker_url == "http://localhost:7125"

def test_load_config_parses_telegram(tmp_path):
    cfg_file = tmp_path / "printers.yaml"
    cfg_file.write_text(SAMPLE_YAML)
    config = load_config(cfg_file)
    assert config.telegram.bot_token == "abc123"
    assert config.telegram.quiet_hours.start == "23:00"

def test_load_config_missing_file_raises():
    with pytest.raises(FileNotFoundError):
        load_config(Path("/nonexistent/printers.yaml"))

def test_printer_config_camera_url_optional(tmp_path):
    yaml = """
printers:
  - id: p1
    name: "P1"
    moonraker_url: "http://localhost:7125"
telegram:
  bot_token: "x"
  chat_id: "y"
"""
    cfg_file = tmp_path / "printers.yaml"
    cfg_file.write_text(yaml)
    config = load_config(cfg_file)
    assert config.printers[0].camera_url is None
```

**Step 2: Run tests to confirm they fail**

```bash
cd backend && python -m pytest tests/test_config.py -v
```
Expected: `ModuleNotFoundError` or `ImportError`

**Step 3: Implement config loader**

```python
# backend/config.py
from pathlib import Path
from pydantic import BaseModel
import yaml

class PrinterConfig(BaseModel):
    id: str
    name: str
    moonraker_url: str
    camera_url: str | None = None

class QuietHours(BaseModel):
    start: str
    end: str

class TelegramConfig(BaseModel):
    bot_token: str
    chat_id: str
    quiet_hours: QuietHours | None = None

class Config(BaseModel):
    printers: list[PrinterConfig]
    telegram: TelegramConfig | None = None

def load_config(path: Path) -> Config:
    if not path.exists():
        raise FileNotFoundError(f"Config not found: {path}")
    with open(path) as f:
        data = yaml.safe_load(f)
    return Config(**data)
```

**Step 4: Run tests to confirm they pass**

```bash
python -m pytest tests/test_config.py -v
```
Expected: 4 passed

**Step 5: Commit**

```bash
git add backend/config.py backend/tests/test_config.py
git commit -m "feat: config loader with pydantic validation"
```

---

### Task 3: Printer State Model

**Files:**
- Create: `backend/state.py`
- Create: `backend/tests/test_state.py`

**Step 1: Write failing tests**

```python
# backend/tests/test_state.py
from backend.state import PrinterState, StateStore, PrinterStatus

def test_initial_state_is_offline():
    state = PrinterState(id="p1", name="Test")
    assert state.status == PrinterStatus.OFFLINE

def test_merge_print_stats_updates_status():
    state = PrinterState(id="p1", name="Test")
    state.merge({"print_stats": {"state": "printing", "filename": "benchy.gcode", "print_duration": 300.0}})
    assert state.status == PrinterStatus.PRINTING
    assert state.filename == "benchy.gcode"

def test_merge_heaters_updates_temps():
    state = PrinterState(id="p1", name="Test")
    state.merge({"heaters": {"extruder": {"temperature": 235.1, "target": 235.0}, "heater_bed": {"temperature": 65.2, "target": 65.0}}})
    assert state.nozzle_temp == pytest.approx(235.1, abs=0.1)
    assert state.bed_temp == pytest.approx(65.2, abs=0.1)

def test_merge_virtual_sdcard_updates_progress():
    state = PrinterState(id="p1", name="Test")
    state.merge({"virtual_sdcard": {"progress": 0.67}})
    assert state.progress == pytest.approx(0.67, abs=0.01)

def test_state_store_get_all():
    store = StateStore()
    store.add_printer("p1", "Printer 1")
    store.add_printer("p2", "Printer 2")
    all_states = store.get_all()
    assert len(all_states) == 2

def test_state_store_mark_offline():
    store = StateStore()
    store.add_printer("p1", "Printer 1")
    store.merge("p1", {"print_stats": {"state": "printing", "filename": "x.gcode", "print_duration": 0}})
    store.mark_offline("p1")
    assert store.get("p1").status == PrinterStatus.OFFLINE
```

**Step 2: Run tests to confirm they fail**

```bash
python -m pytest tests/test_state.py -v
```
Expected: `ImportError`

**Step 3: Implement state model**

```python
# backend/state.py
from dataclasses import dataclass, field
from enum import Enum

class PrinterStatus(str, Enum):
    PRINTING = "printing"
    PAUSED = "paused"
    IDLE = "idle"
    ERROR = "error"
    OFFLINE = "offline"

MOONRAKER_TO_STATUS = {
    "printing": PrinterStatus.PRINTING,
    "paused": PrinterStatus.PAUSED,
    "standby": PrinterStatus.IDLE,
    "complete": PrinterStatus.IDLE,
    "cancelled": PrinterStatus.IDLE,
    "error": PrinterStatus.ERROR,
}

@dataclass
class PrinterState:
    id: str
    name: str
    status: PrinterStatus = PrinterStatus.OFFLINE
    filename: str | None = None
    progress: float = 0.0
    print_duration: float = 0.0
    nozzle_temp: float = 0.0
    nozzle_target: float = 0.0
    bed_temp: float = 0.0
    bed_target: float = 0.0
    fan_speed: float = 0.0
    eta: float | None = None

    def merge(self, data: dict) -> None:
        if ps := data.get("print_stats"):
            if raw := ps.get("state"):
                self.status = MOONRAKER_TO_STATUS.get(raw, PrinterStatus.IDLE)
            if fn := ps.get("filename"):
                self.filename = fn
            if dur := ps.get("print_duration"):
                self.print_duration = dur
        if h := data.get("heaters"):
            if ext := h.get("extruder"):
                self.nozzle_temp = ext.get("temperature", self.nozzle_temp)
                self.nozzle_target = ext.get("target", self.nozzle_target)
            if bed := h.get("heater_bed"):
                self.bed_temp = bed.get("temperature", self.bed_temp)
                self.bed_target = bed.get("target", self.bed_target)
        if vsd := data.get("virtual_sdcard"):
            self.progress = vsd.get("progress", self.progress)
            if self.progress > 0 and self.print_duration > 0:
                elapsed = self.print_duration
                total = elapsed / self.progress if self.progress > 0 else 0
                self.eta = max(0, total - elapsed)
        if fan := data.get("fan"):
            self.fan_speed = fan.get("speed", self.fan_speed)

    def to_dict(self) -> dict:
        return {
            "id": self.id,
            "name": self.name,
            "status": self.status.value,
            "filename": self.filename,
            "progress": round(self.progress, 3),
            "print_duration": round(self.print_duration, 1),
            "nozzle_temp": round(self.nozzle_temp, 1),
            "nozzle_target": round(self.nozzle_target, 1),
            "bed_temp": round(self.bed_temp, 1),
            "bed_target": round(self.bed_target, 1),
            "fan_speed": round(self.fan_speed, 2),
            "eta": round(self.eta, 1) if self.eta is not None else None,
        }

class StateStore:
    def __init__(self):
        self._states: dict[str, PrinterState] = {}

    def add_printer(self, id: str, name: str) -> None:
        self._states[id] = PrinterState(id=id, name=name)

    def merge(self, printer_id: str, data: dict) -> PrinterState:
        self._states[printer_id].merge(data)
        return self._states[printer_id]

    def mark_offline(self, printer_id: str) -> None:
        self._states[printer_id].status = PrinterStatus.OFFLINE

    def get(self, printer_id: str) -> PrinterState:
        return self._states[printer_id]

    def get_all(self) -> list[PrinterState]:
        return list(self._states.values())
```

**Step 4: Run tests**

```bash
python -m pytest tests/test_state.py -v
```
Expected: 6 passed

**Step 5: Commit**

```bash
git add backend/state.py backend/tests/test_state.py
git commit -m "feat: printer state model and store"
```

---

### Task 4: Moonraker WebSocket Client

**Files:**
- Create: `backend/moonraker.py`
- Create: `backend/tests/test_moonraker.py`

**Step 1: Write failing tests**

```python
# backend/tests/test_moonraker.py
import asyncio
import json
import pytest
import pytest_asyncio
from unittest.mock import AsyncMock, patch, MagicMock
from backend.moonraker import MoonrakerClient
from backend.state import StateStore, PrinterStatus

@pytest.mark.asyncio
async def test_client_marks_offline_on_connect_failure():
    store = StateStore()
    store.add_printer("p1", "Test")
    client = MoonrakerClient("p1", "ws://localhost:9999", store, on_change=AsyncMock())
    # Port 9999 should refuse connection
    await client.connect_once()
    assert store.get("p1").status == PrinterStatus.OFFLINE

@pytest.mark.asyncio
async def test_client_parses_status_update():
    store = StateStore()
    store.add_printer("p1", "Test")
    on_change = AsyncMock()
    client = MoonrakerClient("p1", "ws://irrelevant", store, on_change=on_change)

    notify = {
        "jsonrpc": "2.0",
        "method": "notify_status_update",
        "params": [
            {"print_stats": {"state": "printing", "filename": "test.gcode", "print_duration": 60.0}},
            {}
        ]
    }
    await client._handle_message(json.dumps(notify))
    assert store.get("p1").status == PrinterStatus.PRINTING
    on_change.assert_called_once_with("p1")
```

**Step 2: Run tests to confirm they fail**

```bash
python -m pytest tests/test_moonraker.py -v
```
Expected: `ImportError`

**Step 3: Implement Moonraker client**

```python
# backend/moonraker.py
import asyncio
import json
import logging
from typing import Callable, Awaitable
import websockets
from backend.state import StateStore

logger = logging.getLogger(__name__)

SUBSCRIBE_OBJECTS = {
    "print_stats": None,
    "heaters": None,
    "virtual_sdcard": None,
    "display_status": None,
    "fan": None,
}

class MoonrakerClient:
    def __init__(
        self,
        printer_id: str,
        url: str,
        store: StateStore,
        on_change: Callable[[str], Awaitable[None]],
    ):
        self.printer_id = printer_id
        # Convert http:// to ws:// if needed
        self.url = url.replace("http://", "ws://").replace("https://", "wss://") + "/websocket"
        self.store = store
        self.on_change = on_change
        self._running = False

    async def run(self) -> None:
        """Run forever with exponential backoff reconnection."""
        self._running = True
        delay = 2
        while self._running:
            await self.connect_once()
            if self._running:
                logger.info(f"{self.printer_id}: reconnecting in {delay}s")
                await asyncio.sleep(delay)
                delay = min(delay * 2, 60)

    async def connect_once(self) -> None:
        try:
            async with websockets.connect(self.url, open_timeout=5) as ws:
                logger.info(f"{self.printer_id}: connected")
                await self._subscribe(ws)
                async for message in ws:
                    await self._handle_message(message)
        except Exception as e:
            logger.warning(f"{self.printer_id}: connection error: {e}")
            self.store.mark_offline(self.printer_id)
            await self.on_change(self.printer_id)

    async def _subscribe(self, ws) -> None:
        payload = {
            "jsonrpc": "2.0",
            "method": "printer.subscribe_objects",
            "params": {"objects": SUBSCRIBE_OBJECTS},
            "id": 1,
        }
        await ws.send(json.dumps(payload))

    async def _handle_message(self, raw: str) -> None:
        try:
            msg = json.loads(raw)
        except json.JSONDecodeError:
            return

        method = msg.get("method", "")

        if method == "notify_status_update":
            params = msg.get("params", [{}])
            data = params[0] if params else {}
            self.store.merge(self.printer_id, data)
            await self.on_change(self.printer_id)

        elif method == "notify_klippy_disconnected":
            self.store.mark_offline(self.printer_id)
            await self.on_change(self.printer_id)

        elif msg.get("id") == 1:
            # Response to subscribe — contains initial full state
            result = msg.get("result", {})
            status = result.get("status", {})
            if status:
                self.store.merge(self.printer_id, status)
                await self.on_change(self.printer_id)

    def stop(self) -> None:
        self._running = False
```

**Step 4: Run tests**

```bash
python -m pytest tests/test_moonraker.py -v
```
Expected: 2 passed

**Step 5: Commit**

```bash
git add backend/moonraker.py backend/tests/test_moonraker.py
git commit -m "feat: moonraker websocket client with reconnection"
```

---

### Task 5: Telegram Alert Bot

**Files:**
- Create: `backend/telegram_bot.py`
- Create: `backend/tests/test_telegram_bot.py`

**Step 1: Write failing tests**

```python
# backend/tests/test_telegram_bot.py
import pytest
from unittest.mock import AsyncMock, patch
from datetime import time
from backend.telegram_bot import AlertBot, is_quiet_hours
from backend.state import PrinterState, PrinterStatus

def test_quiet_hours_within_range():
    assert is_quiet_hours("23:00", "07:00", current=time(23, 30)) is True
    assert is_quiet_hours("23:00", "07:00", current=time(3, 0)) is True

def test_quiet_hours_outside_range():
    assert is_quiet_hours("23:00", "07:00", current=time(12, 0)) is False
    assert is_quiet_hours("23:00", "07:00", current=time(22, 59)) is False

@pytest.mark.asyncio
async def test_alert_sent_on_print_complete():
    bot = AlertBot(bot_token="fake", chat_id="123")
    bot._send = AsyncMock()

    prev = PrinterState(id="p1", name="Neptune 3 Max", status=PrinterStatus.PRINTING, filename="benchy.gcode")
    curr = PrinterState(id="p1", name="Neptune 3 Max", status=PrinterStatus.IDLE, filename="benchy.gcode")
    await bot.on_state_change(prev, curr)

    bot._send.assert_called_once()
    msg = bot._send.call_args[0][0]
    assert "Neptune 3 Max" in msg
    assert "benchy.gcode" in msg

@pytest.mark.asyncio
async def test_alert_sent_on_printer_offline():
    bot = AlertBot(bot_token="fake", chat_id="123")
    bot._send = AsyncMock()

    prev = PrinterState(id="p1", name="Ender 7", status=PrinterStatus.IDLE)
    curr = PrinterState(id="p1", name="Ender 7", status=PrinterStatus.OFFLINE)
    await bot.on_state_change(prev, curr)

    bot._send.assert_called_once()
    assert "Ender 7" in bot._send.call_args[0][0]

@pytest.mark.asyncio
async def test_no_alert_when_no_transition():
    bot = AlertBot(bot_token="fake", chat_id="123")
    bot._send = AsyncMock()

    state = PrinterState(id="p1", name="P1", status=PrinterStatus.PRINTING)
    await bot.on_state_change(state, state)
    bot._send.assert_not_called()
```

**Step 2: Run tests to confirm they fail**

```bash
python -m pytest tests/test_telegram_bot.py -v
```
Expected: `ImportError`

**Step 3: Implement alert bot**

```python
# backend/telegram_bot.py
import logging
from datetime import datetime, time
from copy import deepcopy
from telegram import Bot
from backend.state import PrinterState, PrinterStatus

logger = logging.getLogger(__name__)

def is_quiet_hours(start: str, end: str, current: time | None = None) -> bool:
    if current is None:
        current = datetime.now().time()
    start_t = time(*map(int, start.split(":")))
    end_t = time(*map(int, end.split(":")))
    if start_t > end_t:  # spans midnight
        return current >= start_t or current < end_t
    return start_t <= current < end_t

class AlertBot:
    def __init__(self, bot_token: str, chat_id: str, quiet_start: str = None, quiet_end: str = None):
        self.chat_id = chat_id
        self.quiet_start = quiet_start
        self.quiet_end = quiet_end
        self._bot = Bot(token=bot_token) if bot_token != "fake" else None

    async def _send(self, message: str) -> None:
        if self._bot:
            await self._bot.send_message(chat_id=self.chat_id, text=message)
        else:
            logger.info(f"[TELEGRAM] {message}")

    def _in_quiet_hours(self) -> bool:
        if not self.quiet_start or not self.quiet_end:
            return False
        return is_quiet_hours(self.quiet_start, self.quiet_end)

    async def on_state_change(self, prev: PrinterState, curr: PrinterState) -> None:
        msg = None
        suppress = False

        # Print complete: was printing/paused, now idle
        if prev.status in (PrinterStatus.PRINTING, PrinterStatus.PAUSED) and curr.status == PrinterStatus.IDLE:
            mins = int(prev.print_duration / 60)
            msg = f"✅ {curr.name} — `{prev.filename}` done ({mins}m)"
            suppress = True  # suppress in quiet hours

        # Print failed
        elif prev.status == PrinterStatus.PRINTING and curr.status == PrinterStatus.ERROR:
            msg = f"❌ {curr.name} — print error: `{prev.filename}`"

        # Cancelled
        elif prev.status in (PrinterStatus.PRINTING, PrinterStatus.PAUSED) and curr.status == PrinterStatus.IDLE and curr.filename is None:
            msg = f"⚠️ {curr.name} — print cancelled"

        # Went offline
        elif prev.status != PrinterStatus.OFFLINE and curr.status == PrinterStatus.OFFLINE:
            msg = f"⛔ {curr.name} — lost connection"

        # Came back online
        elif prev.status == PrinterStatus.OFFLINE and curr.status != PrinterStatus.OFFLINE:
            msg = f"🟢 {curr.name} — reconnected"
            suppress = True

        if msg and not (suppress and self._in_quiet_hours()):
            await self._send(msg)
```

**Step 4: Run tests**

```bash
python -m pytest tests/test_telegram_bot.py -v
```
Expected: 4 passed

**Step 5: Commit**

```bash
git add backend/telegram_bot.py backend/tests/test_telegram_bot.py
git commit -m "feat: telegram alert bot with quiet hours"
```

---

### Task 6: FastAPI App + WebSocket Server

**Files:**
- Create: `backend/main.py`
- Create: `backend/tests/test_api.py`

**Step 1: Write failing tests**

```python
# backend/tests/test_api.py
import pytest
from httpx import AsyncClient, ASGITransport
from unittest.mock import patch, MagicMock
from backend.state import StateStore, PrinterStatus

@pytest.mark.asyncio
async def test_health_endpoint():
    from backend.main import create_app
    store = StateStore()
    app = create_app(store, clients=[], alert_bot=None)
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as client:
        resp = await client.get("/api/health")
    assert resp.status_code == 200
    assert resp.json()["status"] == "ok"

@pytest.mark.asyncio
async def test_printers_endpoint_returns_state():
    from backend.main import create_app
    store = StateStore()
    store.add_printer("p1", "Test Printer")
    app = create_app(store, clients=[], alert_bot=None)
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as client:
        resp = await client.get("/api/printers")
    assert resp.status_code == 200
    data = resp.json()
    assert len(data) == 1
    assert data[0]["id"] == "p1"
    assert data[0]["status"] == "offline"
```

**Step 2: Run tests to confirm they fail**

```bash
python -m pytest tests/test_api.py -v
```
Expected: `ImportError`

**Step 3: Implement FastAPI app**

```python
# backend/main.py
import asyncio
import json
import logging
from contextlib import asynccontextmanager
from copy import deepcopy
from pathlib import Path
from typing import Optional

from fastapi import FastAPI, WebSocket, WebSocketDisconnect, HTTPException
from fastapi.middleware.cors import CORSMiddleware

from backend.config import load_config
from backend.moonraker import MoonrakerClient
from backend.state import StateStore, PrinterState
from backend.telegram_bot import AlertBot

logger = logging.getLogger(__name__)

class ConnectionManager:
    def __init__(self):
        self.active: list[WebSocket] = []

    async def connect(self, ws: WebSocket):
        await ws.accept()
        self.active.append(ws)

    def disconnect(self, ws: WebSocket):
        self.active.remove(ws)

    async def broadcast(self, message: dict):
        dead = []
        for ws in self.active:
            try:
                await ws.send_json(message)
            except Exception:
                dead.append(ws)
        for ws in dead:
            self.active.remove(ws)

def create_app(store: StateStore, clients: list, alert_bot: Optional[AlertBot]) -> FastAPI:
    manager = ConnectionManager()
    prev_states: dict[str, PrinterState] = {}

    async def on_printer_change(printer_id: str):
        curr = store.get(printer_id)
        prev = prev_states.get(printer_id)
        if prev and alert_bot:
            await alert_bot.on_state_change(prev, curr)
        prev_states[printer_id] = deepcopy(curr)
        await manager.broadcast({"type": "update", "printer": curr.to_dict()})

    app = FastAPI(title="Printfarm Dashboard")
    app.add_middleware(CORSMiddleware, allow_origins=["*"], allow_methods=["*"], allow_headers=["*"])

    @app.get("/api/health")
    async def health():
        return {"status": "ok"}

    @app.get("/api/printers")
    async def get_printers():
        return [p.to_dict() for p in store.get_all()]

    @app.post("/api/printers/{printer_id}/command")
    async def send_command(printer_id: str, body: dict):
        import httpx
        cfg_printers = {c.printer_id: c for c in clients}
        if printer_id not in cfg_printers:
            raise HTTPException(status_code=404, detail="Printer not found")
        moonraker_url = cfg_printers[printer_id].url.replace("ws://", "http://").replace("/websocket", "")
        script = body.get("script", "")
        async with httpx.AsyncClient() as http:
            resp = await http.post(f"{moonraker_url}/printer/gcode/script", json={"script": script})
        return {"ok": resp.status_code == 200}

    @app.websocket("/ws")
    async def websocket_endpoint(ws: WebSocket):
        await manager.connect(ws)
        # Send full snapshot on connect
        await ws.send_json({"type": "snapshot", "printers": [p.to_dict() for p in store.get_all()]})
        try:
            while True:
                await ws.receive_text()  # keep-alive
        except WebSocketDisconnect:
            manager.disconnect(ws)

    return app

def build_app_from_config(config_path: Path = Path("printers.yaml")) -> FastAPI:
    config = load_config(config_path)
    store = StateStore()
    clients = []
    alert_bot = None

    if config.telegram:
        qh = config.telegram.quiet_hours
        alert_bot = AlertBot(
            bot_token=config.telegram.bot_token,
            chat_id=config.telegram.chat_id,
            quiet_start=qh.start if qh else None,
            quiet_end=qh.end if qh else None,
        )

    async def on_change(printer_id: str):
        pass  # replaced after app creation

    for printer in config.printers:
        store.add_printer(printer.id, printer.name)
        client = MoonrakerClient(printer.id, printer.moonraker_url, store, on_change=on_change)
        clients.append(client)

    app = create_app(store, clients, alert_bot)
    return app

if __name__ == "__main__":
    import uvicorn
    app = build_app_from_config()
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

**Step 4: Run tests**

```bash
python -m pytest tests/test_api.py -v
```
Expected: 2 passed

**Step 5: Run all backend tests**

```bash
python -m pytest tests/ -v
```
Expected: all pass

**Step 6: Commit**

```bash
git add backend/main.py backend/tests/test_api.py
git commit -m "feat: fastapi app with websocket broadcast and command proxy"
```

---

### Task 7: React Frontend Scaffolding

**Files:**
- Create: `frontend/index.html`
- Create: `frontend/src/main.tsx`
- Create: `frontend/src/App.tsx`
- Create: `frontend/src/types.ts`
- Create: `frontend/src/hooks/useFarmSocket.ts`
- Create: `frontend/tailwind.config.js`
- Create: `frontend/vite.config.ts`
- Create: `frontend/tsconfig.json`

**Step 1: Init and install dependencies**

```bash
cd frontend
npm install
npx tailwindcss init -p
```

**Step 2: Create type definitions**

```typescript
// frontend/src/types.ts
export type PrinterStatus = "printing" | "paused" | "idle" | "error" | "offline";

export interface PrinterState {
  id: string;
  name: string;
  status: PrinterStatus;
  filename: string | null;
  progress: number;
  print_duration: number;
  nozzle_temp: number;
  nozzle_target: number;
  bed_temp: number;
  bed_target: number;
  fan_speed: number;
  eta: number | null;
}
```

**Step 3: Create WebSocket hook**

```typescript
// frontend/src/hooks/useFarmSocket.ts
import { useEffect, useRef, useState } from "react";
import { PrinterState } from "../types";

const WS_URL = import.meta.env.VITE_WS_URL ?? "ws://localhost:8000/ws";

export function useFarmSocket() {
  const [printers, setPrinters] = useState<Record<string, PrinterState>>({});
  const [connected, setConnected] = useState(false);
  const ws = useRef<WebSocket | null>(null);

  useEffect(() => {
    function connect() {
      ws.current = new WebSocket(WS_URL);

      ws.current.onopen = () => setConnected(true);
      ws.current.onclose = () => {
        setConnected(false);
        setTimeout(connect, 3000);
      };
      ws.current.onmessage = (e) => {
        const msg = JSON.parse(e.data);
        if (msg.type === "snapshot") {
          const map: Record<string, PrinterState> = {};
          msg.printers.forEach((p: PrinterState) => { map[p.id] = p; });
          setPrinters(map);
        } else if (msg.type === "update") {
          setPrinters((prev) => ({ ...prev, [msg.printer.id]: msg.printer }));
        }
      };
    }
    connect();
    return () => ws.current?.close();
  }, []);

  return { printers: Object.values(printers), connected };
}
```

**Step 4: Create vite config**

```typescript
// frontend/vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      "/api": "http://localhost:8000",
      "/ws": { target: "ws://localhost:8000", ws: true },
    },
  },
  test: {
    environment: "jsdom",
    setupFiles: ["./src/test-setup.ts"],
  },
});
```

**Step 5: Commit**

```bash
cd ..
git add frontend/
git commit -m "feat: react frontend scaffolding with websocket hook"
```

---

### Task 8: FarmHeader Component

**Files:**
- Create: `frontend/src/components/FarmHeader.tsx`
- Create: `frontend/src/components/FarmHeader.test.tsx`

**Step 1: Write failing tests**

```typescript
// frontend/src/components/FarmHeader.test.tsx
import { render, screen } from "@testing-library/react";
import { FarmHeader } from "./FarmHeader";
import { PrinterState } from "../types";

const makePrinter = (id: string, status: PrinterState["status"]): PrinterState => ({
  id, name: id, status, filename: null, progress: 0,
  print_duration: 0, nozzle_temp: 0, nozzle_target: 0,
  bed_temp: 0, bed_target: 0, fan_speed: 0, eta: null,
});

test("shows correct printing count", () => {
  const printers = [
    makePrinter("p1", "printing"),
    makePrinter("p2", "printing"),
    makePrinter("p3", "idle"),
  ];
  render(<FarmHeader printers={printers} onEmergencyStop={() => {}} />);
  expect(screen.getByText(/2 printing/)).toBeInTheDocument();
});

test("shows emergency stop button", () => {
  render(<FarmHeader printers={[]} onEmergencyStop={() => {}} />);
  expect(screen.getByRole("button", { name: /emergency stop/i })).toBeInTheDocument();
});
```

**Step 2: Run tests to confirm they fail**

```bash
cd frontend && npm test
```
Expected: fail with component not found

**Step 3: Implement FarmHeader**

```typescript
// frontend/src/components/FarmHeader.tsx
import { PrinterState } from "../types";

interface Props {
  printers: PrinterState[];
  onEmergencyStop: () => void;
}

export function FarmHeader({ printers, onEmergencyStop }: Props) {
  const counts = printers.reduce(
    (acc, p) => { acc[p.status] = (acc[p.status] ?? 0) + 1; return acc; },
    {} as Record<string, number>
  );

  return (
    <header className="flex items-center justify-between px-6 py-3 bg-gray-900 text-white border-b border-gray-700">
      <div className="flex items-center gap-6">
        <span className="font-bold text-lg">PrintFarm</span>
        <span className="text-sm text-gray-400">
          {printers.length} printers
          {counts.printing ? ` · ${counts.printing} printing` : ""}
          {counts.paused ? ` · ${counts.paused} paused` : ""}
          {counts.idle ? ` · ${counts.idle} idle` : ""}
          {counts.offline ? ` · ${counts.offline} offline` : ""}
        </span>
      </div>
      <button
        onClick={onEmergencyStop}
        className="bg-red-600 hover:bg-red-700 text-white text-sm font-semibold px-4 py-2 rounded"
        aria-label="Emergency Stop All"
      >
        ⛔ Emergency Stop All
      </button>
    </header>
  );
}
```

**Step 4: Run tests**

```bash
npm test
```
Expected: pass

**Step 5: Commit**

```bash
cd .. && git add frontend/src/components/FarmHeader.tsx frontend/src/components/FarmHeader.test.tsx
git commit -m "feat: FarmHeader component with summary counts and emergency stop"
```

---

### Task 9: PrinterCard Component

**Files:**
- Create: `frontend/src/components/PrinterCard.tsx`
- Create: `frontend/src/components/PrinterCard.test.tsx`

**Step 1: Write failing tests**

```typescript
// frontend/src/components/PrinterCard.test.tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { PrinterCard } from "./PrinterCard";
import { PrinterState } from "../types";

const printingPrinter: PrinterState = {
  id: "p1", name: "Neptune 3 Max", status: "printing",
  filename: "benchy.gcode", progress: 0.67, print_duration: 3600,
  nozzle_temp: 235, nozzle_target: 235, bed_temp: 65, bed_target: 65,
  fan_speed: 1.0, eta: 1800,
};

test("shows printer name and filename", () => {
  render(<PrinterCard printer={printingPrinter} onCommand={() => {}} onExpand={() => {}} />);
  expect(screen.getByText("Neptune 3 Max")).toBeInTheDocument();
  expect(screen.getByText("benchy.gcode")).toBeInTheDocument();
});

test("shows progress percentage", () => {
  render(<PrinterCard printer={printingPrinter} onCommand={() => {}} onExpand={() => {}} />);
  expect(screen.getByText("67%")).toBeInTheDocument();
});

test("shows temperatures", () => {
  render(<PrinterCard printer={printingPrinter} onCommand={() => {}} onExpand={() => {}} />);
  expect(screen.getByText(/235/)).toBeInTheDocument();
  expect(screen.getByText(/65/)).toBeInTheDocument();
});

test("calls onCommand with pause script", async () => {
  const onCommand = vi.fn();
  render(<PrinterCard printer={printingPrinter} onCommand={onCommand} onExpand={() => {}} />);
  await userEvent.click(screen.getByRole("button", { name: /pause/i }));
  expect(onCommand).toHaveBeenCalledWith("p1", "PAUSE");
});
```

**Step 2: Run tests to confirm they fail**

```bash
npm test
```
Expected: fail

**Step 3: Implement PrinterCard**

```typescript
// frontend/src/components/PrinterCard.tsx
import { PrinterState } from "../types";

const STATUS_DOT: Record<PrinterState["status"], string> = {
  printing: "🟢", paused: "🟡", idle: "⚪", error: "🔴", offline: "⛔",
};

function formatEta(seconds: number | null): string {
  if (!seconds) return "";
  const h = Math.floor(seconds / 3600);
  const m = Math.floor((seconds % 3600) / 60);
  return h > 0 ? `${h}h ${m}m` : `${m}m`;
}

interface Props {
  printer: PrinterState;
  cameraUrl?: string;
  onCommand: (printerId: string, script: string) => void;
  onExpand: (printerId: string) => void;
}

export function PrinterCard({ printer, cameraUrl, onCommand, onExpand }: Props) {
  const isActive = printer.status === "printing" || printer.status === "paused";

  return (
    <div className="bg-gray-800 rounded-lg overflow-hidden border border-gray-700 flex flex-col">
      {/* Header */}
      <div className="flex items-center justify-between px-4 py-2 bg-gray-750">
        <span className="font-semibold text-white">
          {STATUS_DOT[printer.status]} {printer.name}
        </span>
        {printer.eta && (
          <span className="text-sm text-gray-400">{formatEta(printer.eta)}</span>
        )}
      </div>

      {/* Webcam */}
      {cameraUrl ? (
        <img src={cameraUrl} alt="webcam" className="w-full aspect-video object-cover bg-gray-900" />
      ) : (
        <div className="w-full aspect-video bg-gray-900 flex items-center justify-center text-gray-600 text-sm">
          No camera
        </div>
      )}

      {/* Progress */}
      <div className="px-4 py-2">
        {printer.filename && (
          <div className="flex justify-between text-sm text-gray-300 mb-1">
            <span className="truncate">{printer.filename}</span>
            <span className="ml-2 shrink-0">{Math.round(printer.progress * 100)}%</span>
          </div>
        )}
        <div className="h-2 bg-gray-700 rounded-full">
          <div
            className="h-2 bg-blue-500 rounded-full transition-all"
            style={{ width: `${printer.progress * 100}%` }}
          />
        </div>
      </div>

      {/* Temps */}
      <div className="px-4 py-2 flex gap-4 text-sm text-gray-300 border-t border-gray-700">
        <span>🌡 {printer.nozzle_temp.toFixed(0)}° / {printer.nozzle_target.toFixed(0)}°</span>
        <span>🛏 {printer.bed_temp.toFixed(0)}° / {printer.bed_target.toFixed(0)}°</span>
      </div>

      {/* Actions */}
      <div className="px-4 py-2 flex gap-2 border-t border-gray-700">
        {printer.status === "printing" && (
          <button onClick={() => onCommand(printer.id, "PAUSE")}
            className="text-xs bg-yellow-600 hover:bg-yellow-700 text-white px-3 py-1 rounded">
            Pause
          </button>
        )}
        {printer.status === "paused" && (
          <button onClick={() => onCommand(printer.id, "RESUME")}
            className="text-xs bg-green-600 hover:bg-green-700 text-white px-3 py-1 rounded">
            Resume
          </button>
        )}
        {isActive && (
          <button onClick={() => onCommand(printer.id, "CANCEL_PRINT")}
            className="text-xs bg-red-700 hover:bg-red-800 text-white px-3 py-1 rounded">
            Cancel
          </button>
        )}
        <button onClick={() => onExpand(printer.id)}
          className="text-xs bg-gray-600 hover:bg-gray-500 text-white px-3 py-1 rounded ml-auto">
          Details
        </button>
      </div>
    </div>
  );
}
```

**Step 4: Run tests**

```bash
npm test
```
Expected: pass

**Step 5: Commit**

```bash
cd .. && git add frontend/src/components/PrinterCard.tsx frontend/src/components/PrinterCard.test.tsx
git commit -m "feat: PrinterCard component with temps, progress, and controls"
```

---

### Task 10: App Shell + Wire-Up

**Files:**
- Modify: `frontend/src/App.tsx`

**Step 1: Implement App**

```typescript
// frontend/src/App.tsx
import { useState } from "react";
import { useFarmSocket } from "./hooks/useFarmSocket";
import { FarmHeader } from "./components/FarmHeader";
import { PrinterCard } from "./components/PrinterCard";

const CAMERA_URLS: Record<string, string> = {
  // Populated from env or hardcoded per deployment
  // e.g. twin1: "http://192.168.1.100:8080/?action=stream"
};

async function sendCommand(printerId: string, script: string) {
  await fetch(`/api/printers/${printerId}/command`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ script }),
  });
}

async function emergencyStopAll(printerIds: string[]) {
  if (!confirm("Emergency stop ALL printers?")) return;
  await Promise.all(printerIds.map((id) => sendCommand(id, "FIRMWARE_RESTART")));
}

export default function App() {
  const { printers, connected } = useFarmSocket();
  const [expanded, setExpanded] = useState<string | null>(null);

  return (
    <div className="min-h-screen bg-gray-950 text-white">
      <FarmHeader
        printers={printers}
        onEmergencyStop={() => emergencyStopAll(printers.map((p) => p.id))}
      />
      {!connected && (
        <div className="text-center py-2 bg-yellow-800 text-yellow-200 text-sm">
          Connecting to farm…
        </div>
      )}
      <main className="p-4 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        {printers.map((printer) => (
          <PrinterCard
            key={printer.id}
            printer={printer}
            cameraUrl={CAMERA_URLS[printer.id]}
            onCommand={sendCommand}
            onExpand={setExpanded}
          />
        ))}
      </main>
    </div>
  );
}
```

**Step 2: Run dev server to manually verify**

```bash
cd frontend && npm run dev
# Open http://localhost:5173
# Should show connected header and empty grid (no printers.yaml yet)
```

**Step 3: Commit**

```bash
cd .. && git add frontend/src/App.tsx
git commit -m "feat: app shell wired to farm websocket"
```

---

### Task 11: Docker Configuration

**Files:**
- Create: `Dockerfile.backend`
- Create: `Dockerfile.frontend`
- Create: `nginx.conf`
- Create: `docker-compose.yml`

**Step 1: Backend Dockerfile**

```dockerfile
# Dockerfile.backend
FROM python:3.11-slim
WORKDIR /app
COPY backend/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY backend/ ./backend/
COPY printers.yaml .
CMD ["uvicorn", "backend.main:build_app_from_config", "--host", "0.0.0.0", "--port", "8000", "--factory"]
```

**Step 2: Frontend Dockerfile**

```dockerfile
# Dockerfile.frontend
FROM node:20-alpine AS build
WORKDIR /app
COPY frontend/package.json frontend/package-lock.json ./
RUN npm ci
COPY frontend/ .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
```

**Step 3: nginx config**

```nginx
# nginx.conf
server {
    listen 80;

    location /api/ {
        proxy_pass http://backend:8000;
        proxy_set_header Host $host;
    }

    location /ws {
        proxy_pass http://backend:8000/ws;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 86400;
    }

    location / {
        root /usr/share/nginx/html;
        try_files $uri $uri/ /index.html;
    }
}
```

**Step 4: docker-compose.yml**

```yaml
# docker-compose.yml
services:
  backend:
    build:
      context: .
      dockerfile: Dockerfile.backend
    restart: unless-stopped
    volumes:
      - ./printers.yaml:/app/printers.yaml:ro
    networks:
      - farm

  frontend:
    build:
      context: .
      dockerfile: Dockerfile.frontend
    restart: unless-stopped
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - farm

networks:
  farm:
```

**Step 5: Build and verify**

```bash
cp printers.example.yaml printers.yaml
# edit printers.yaml with one real printer IP to test
docker-compose build
docker-compose up
# Open http://localhost — should see dashboard
```

**Step 6: Commit**

```bash
git add Dockerfile.backend Dockerfile.frontend nginx.conf docker-compose.yml
git commit -m "feat: docker compose deployment with nginx reverse proxy"
```

---

### Task 12: README + Deployment Docs

**Files:**
- Create: `README.md`

**Step 1: Write README**

````markdown
# printfarm-dashboard

Real-time web dashboard for a Klipper 3D printer farm. Aggregates all
printers via Moonraker WebSocket, displays a card grid, sends Telegram
alerts on print events.

## Requirements

- Docker + Docker Compose
- Each printer running Klipper + Moonraker (port 7125)
- Telegram bot token + chat ID (optional)

## Setup

```bash
git clone https://github.com/drupi79/printfarm-dashboard
cd printfarm-dashboard
cp printers.example.yaml printers.yaml
# Edit printers.yaml with your printer IPs and Telegram credentials
docker-compose up -d
```

Open `http://<host-ip>` in a browser.

## Remote Access (Tailscale)

Install Tailscale on the Pi, then access via `http://<tailscale-ip>`.

## Adding a Printer

Add a new entry to `printers.yaml` and run `docker-compose restart backend`.

## Development

```bash
# Backend
cd backend && pip install -r requirements.txt
python -m pytest tests/ -v
uvicorn backend.main:build_app_from_config --reload --factory

# Frontend
cd frontend && npm install && npm run dev
```
````

**Step 2: Commit**

```bash
git add README.md
git commit -m "docs: add README with setup and development instructions"
```

---

### Task 13: End-to-End Smoke Test

**Goal:** Verify the full stack works against a real printer.

**Step 1: Point printers.yaml at one real printer**

Edit `printers.yaml` to include one printer you can reach (e.g. Neptune 3 Max or Wanhao D6).

**Step 2: Start the stack**

```bash
docker-compose up
```

**Step 3: Verify checklist**

- [ ] Dashboard loads at `http://localhost`
- [ ] Printer card appears with correct name
- [ ] Status dot reflects actual printer state
- [ ] Temps update in real-time (watch them change)
- [ ] If printer is printing: progress bar and ETA shown
- [ ] Pause/Resume/Cancel buttons visible when printing
- [ ] Backend logs show WebSocket connected to Moonraker
- [ ] Disconnect printer ethernet → card goes ⛔ Offline within ~10s
- [ ] Reconnect → card recovers

**Step 4: Push to GitHub**

```bash
git remote add origin https://github.com/drupi79/printfarm-dashboard.git
git push -u origin main
```
