# Nexus — Grand Server Architecture
> Joebot Ecosystem Central Coordinator
> GitHub: github.com/joebot94/nexus
> Document version 1.1 — March 2026
> Changes: Added action vocabulary, adapter pattern, and device registry from ShowControl v0

---

## What Nexus Is

Nexus is the central coordination server for the entire Joebot studio ecosystem. It is the single point through which all apps, devices, and interfaces communicate. No app talks directly to another app — everything goes through Nexus.

Nexus is not a controller. It does not make creative decisions. It routes intent, maintains state, and keeps every connected client informed.

---

## Core Responsibilities

- Accept and maintain persistent connections from all clients
- Route messages between clients based on intent
- Maintain a live state store of every connected device and app
- Hold last known state when a client goes offline
- Alert Observatory when a client drops unexpectedly
- Handle .jbt scene snapshots — save and recall full studio state
- Provide a queryable API so any client can ask "what is X doing right now"
- Dispatch logical actions to device adapters
- Translate logical actions into device-native commands

---

## What Nexus Is NOT

- Not a hardware controller — that's Atlas, DirtyMixerApp, etc.
- Not a show controller — apps handle their own timelines
- Not a file manager — Glitch Catalog handles session archives
- Not a UI — Observatory is a separate Swift client
- Not a creative decision maker — it only routes what clients tell it to

---

## System Architecture

```
                    [ Nexus Server — Python ]
                           |
        ┌──────────────────┼──────────────────────┐
        │                  │                       │
   [ Atlas ]      [ DirtyMixerApp ]       [ Glitch Catalog ]
        │                  │
[ Extron Hardware ] [ Dirty Mixer Board ]

        ┌──────────────────┼──────────────────────┐
        │                  │                       │
 [ Text Wall ]    [ Observatory ]         [ Future Apps ]
```

Every box is a peer client connected to Nexus. No direct connections between peers. Nexus knows everything. Clients know only themselves.

---

## Tech Stack

| Component | Technology |
|---|---|
| Server runtime | Python 3.11+ |
| Async framework | asyncio |
| Transport protocol | WebSockets |
| Message validation | Pydantic v2 |
| State store | In-memory Python dict (Redis optional later) |
| File I/O | .jbt JSON format |
| Dashboard client | Swift / SwiftUI — Observatory |
| Future scaling | MQTT broker optional for device-heavy expansion |

---

## Connection Model

### Client Registration

When any app or device connects to Nexus it immediately sends a registration message:

```json
{
  "type": "register",
  "client_id": "dirtymixer_v1",
  "client_type": "dirtymixer",
  "version": "1.0.0",
  "capabilities": ["presets", "automation", "random_mode"]
}
```

Nexus responds with acknowledgement and current ecosystem state summary.

### Heartbeat

Every connected client sends a heartbeat every 5 seconds:

```json
{
  "type": "heartbeat",
  "client_id": "dirtymixer_v1",
  "timestamp": 1234567890
}
```

If Nexus misses 3 consecutive heartbeats from a client:
- Client is marked offline in state store
- Last known state is retained
- Observatory receives an alert
- Other clients that care are notified

### State Reporting

Clients report their full state to Nexus whenever something changes:

```json
{
  "type": "state_update",
  "client_id": "dirtymixer_v1",
  "state": {
    "connected_to_board": true,
    "active_preset": "Preset 12",
    "channels": [
      { "id": 1, "a": true, "b": true, "mix": 128 },
      { "id": 2, "a": true, "b": false, "mix": 255 }
    ]
  }
}
```

---

## Message Envelope

All messages use a standard envelope:

```json
{
  "id": "msg_001",
  "type": "scene.request",
  "source": "client_name",
  "payload": { ... }
}
```

All message models are defined using Pydantic v2.

---

## Message Types

### Client → Nexus

| Message Type | Description |
|---|---|
| `register` | Initial connection and capability declaration |
| `heartbeat` | Keep-alive ping |
| `state_update` | Report current state to Nexus |
| `intent` | Ask Nexus to coordinate an action across clients |
| `query` | Ask Nexus for current state of another client |
| `scene_save` | Ask Nexus to capture full ecosystem snapshot |
| `scene_recall` | Ask Nexus to restore a saved scene snapshot |
| `device.list` | Query all known devices and status |
| `scene.list` | Query all known scenes |
| `device.command` | Send a logical action to a specific device |
| `scene.request` | Request a scene be fired |

### Nexus → Client

| Message Type | Description |
|---|---|
| `registered` | Confirmation of successful registration |
| `command` | Instruction routed from another client |
| `state_response` | Answer to a query request |
| `client_offline` | Alert that another client has gone offline |
| `client_online` | Alert that a client has reconnected |
| `scene_saved` | Confirmation that scene snapshot was captured |
| `scene_recalled` | Confirmation that scene recall was dispatched |
| `log.event` | Broadcast log entry to Observatory |
| `device.status` | Device connection/state change broadcast |
| `scene.activated` | Broadcast that a scene was fired |
| `error` | Error response |

---

## Action Vocabulary

Nexus operates on logical actions. Adapters translate these into device-native commands. No device-specific syntax ever appears in Nexus core logic.

### Universal Actions

| Action | Description |
|---|---|
| `recall_preset` | Recall a named or numbered preset on a device |

### Video / Display Actions

| Action | Description |
|---|---|
| `set_video_mute` | Mute video output visually — signal stays synced, output goes black |
| `set_skew` | Set RGB skew compensation values (MTPX) |
| `reset_skew` | Reset RGB skew to default (MTPX) |
| `set_prepeaking` | Set pre-peaking voltage boost level (MTPX) |

### Control / Relay Actions

| Action | Description |
|---|---|
| `pulse_relay` | Trigger a relay pulse on IPCP device |
| `trigger_ir` | Fire an IR command via IPCP EIR list |
| `serial_passthrough` | Send raw SIS or device-native serial through IPCP serial port |

### DirtyMixer Actions

| Action | Description |
|---|---|
| `set_channel_mix` | Set mix value for a channel (0–255) |
| `set_channel_input_a` | Enable or disable input A on a channel |
| `set_channel_input_b` | Enable or disable input B on a channel |
| `recall_dirtymixer_preset` | Recall a DirtyMixerApp preset by ID |
| `set_random_mode` | Enable or disable random mode on a channel |

### Example Action Payload

```json
{
  "id": "msg_042",
  "type": "device.command",
  "source": "atlas",
  "payload": {
    "device_id": "device.dms.main",
    "action": "recall_preset",
    "params": { "preset": 1 }
  }
}
```

---

## Adapter Pattern

Each device type has its own adapter. Adapters translate logical actions into device-native protocol commands. No device-specific syntax ever lives in Nexus core.

### Base Adapter Interface

```python
class BaseAdapter:
    async def connect(self) -> bool: ...
    async def disconnect(self) -> None: ...
    async def execute(self, action: str, params: dict) -> dict: ...
    def get_capabilities(self) -> list[str]: ...
```

### Known Adapters

| Adapter | Device | Protocol |
|---|---|---|
| `MatrixAdapter` | Matrix 12800 | Extron SIS TCP |
| `DMSAdapter` | DMS 3600 | Extron SIS TCP |
| `MGPAdapter` | MGP 464 | Extron SIS TCP |
| `IPCP505Adapter` | IPCP 505 | HTTP + SIS |
| `MTPXAdapter` | MTPX Plus | Extron SIS TCP |
| `DirtyMixerAdapter` | Dirty Mixer Board | USB Serial |

### Extron Preset Recall

For all Extron SIS devices, preset recall syntax is:

```
{preset_number}.
```

Example: Recall preset 1 → send `1.` over TCP port 23.

Logical action `recall_preset` with `{"preset": 1}` maps to `"1."` in all Extron adapters.

### IPCP 505 Relay

Relay pulse performed via HTTP:

```
http://ipcp505-1.extron.video/W=1R01
```

Logical action `pulse_relay` with relay ID maps to this URL pattern.
IR and serial passthrough are stubbed pending exact syntax — marked TODO in adapter code.

---

## Device Registry

### Known Devices

| Device ID | Label | Type | Hostname | Port |
|---|---|---|---|---|
| `device.matrix.main` | Matrix 12800 | matrix | mx.extron.video | 23 |
| `device.dms.main` | DMS 3600 | dms | dms.extron.video | 23 |
| `device.mgp.1` | MGP 464 #1 | mgp | mgp1.extron.video | 23 |
| `device.mgp.2` | MGP 464 #2 | mgp | mgp2.extron.video | 23 |
| `device.mgp.3` | MGP 464 #3 | mgp | mgp3.extron.video | 23 |
| `device.ipcp505.1` | IPCP 505 #1 | ipcp505 | ipcp505-1.extron.video | 23 |
| `device.mtpx.1` | MTPX Plus #1 | mtpx | mtpx1.extron.video | 23 |
| `device.mtpx.2` | MTPX Plus #2 | mtpx | mtpx2.extron.video | 23 |
| `device.dirtymixer.1` | Dirty Mixer Board | dirtymixer | USB | — |

Default port for Extron TCP devices: **23**
Master server hostname: **show.joe.bot**

---

## Intent Routing

Atlas sends intent to Nexus — never directly to another app:

```json
{
  "type": "intent",
  "from": "atlas",
  "targets": ["dirtymixer_v1", "textwall_v1"],
  "action": "recall_preset",
  "payload": { "preset_id": 12 },
  "sync": true,
  "timestamp": 1234567890
}
```

Nexus fans this out simultaneously to all targets. Everything fires in sync.

---

## State Store

```python
state_store = {
    "atlas": {
        "status": "online",
        "last_seen": 1234567890,
        "last_state": { ... }
    },
    "dirtymixer_v1": {
        "status": "online",
        "last_seen": 1234567890,
        "last_state": { ... }
    },
    "textwall_v1": {
        "status": "offline",
        "last_seen": 1234567880,
        "last_state": { ... }  # retained even when offline
    }
}
```

---

## Scene Snapshot System

### Saving a Scene

1. Nexus receives `scene_save` request
2. Nexus polls all connected clients for current state
3. Each client responds with full state dump
4. Nexus bundles into a `glitch_session` .jbt file
5. .jbt returned to Glitch Catalog for storage

### Recalling a Scene

1. Glitch Catalog sends `scene_recall` with .jbt payload
2. Nexus parses the bundle
3. Nexus fans relevant chunks to each client
4. Each client executes its own recall
5. Nexus confirms completion

### Initial Scene Definitions

| Scene ID | Description |
|---|---|
| `scene.video_wall.3x3` | Full 3x3 wall — matrix + DMS + MGP presets |
| `scene.video_wall.1x1` | Single fullscreen |
| `scene.text.fullscreen` | Text wall fullscreen mode |
| `scene.wall.blackout` | All outputs muted/blacked |

### Initial Pattern Definitions

| Pattern ID | Description |
|---|---|
| `pattern.random_chaos.beats_1_3` | Random preset chaos on beats 1 and 3 |
| `pattern.blank.snake_3x3` | Sequential mute/unmute across 3x3 wall |
| `pattern.blank.checkerboard` | Alternating mute/unmute groups |
| `pattern.lyric.advance_relay` | Pulse IPCP relay on lyric/text advance |

---

## Client Offline Handling

When a client misses 3 heartbeats:

1. Nexus marks client `offline` in state store
2. Last known state retained — NOT cleared
3. Observatory receives alert with last known state
4. On reconnect — Nexus marks `online`, notifies Observatory
5. Client re-registers and reports fresh state

---

## Client Registry

| Client ID | Type | App | Controls |
|---|---|---|---|
| `atlas` | controller | Atlas | Extron MGP, DMS 3600, Matrix 12800 |
| `dirtymixer_v1` | controller | DirtyMixerApp | Dirty Mixer Board |
| `glitch_catalog` | archive | Glitch Catalog | Session files, scene recall |
| `textwall_v1` | display | Text Wall App | Text display output |
| `observatory` | monitor | Observatory | Read-only monitoring |
| `future_*` | unknown | TBD | Extensible |

New clients self-register on connection. Nexus requires no prior knowledge of them.

---

## Nexus Server Structure

```
nexus/
├── main.py
├── requirements.txt
├── README.md
├── config/
│   ├── devices.json
│   ├── actions.json
│   ├── scenes.json
│   └── patterns.json
├── core/
│   ├── app_state.py
│   ├── device_registry.py
│   ├── scene_registry.py
│   ├── pattern_registry.py
│   ├── log_bus.py
│   ├── dispatcher.py
│   └── adapter_manager.py
├── api/
│   ├── models.py
│   ├── protocol.py
│   ├── websocket_server.py
│   └── handlers.py
├── adapters/
│   ├── base.py
│   ├── extron_common.py
│   ├── matrix_adapter.py
│   ├── dms_adapter.py
│   ├── mgp_adapter.py
│   ├── ipcp505_adapter.py
│   ├── mtpx_adapter.py
│   └── dirtymixer_adapter.py
└── jbt/
    ├── parser.py
    └── writer.py
```

---

## Observatory

Observatory is the Swift/SwiftUI monitoring dashboard for Nexus.
GitHub: `github.com/joebot94/observatory`

Displays:
- Live connection status of all clients
- Per-device state cards with device-specific detail
- Extron device health — temps, fans, power supply
- Alerts panel for offline clients
- Scene fire buttons
- Live log feed from Nexus log bus

Observatory is read-only by default. It observes and reports, does not command.

---

## Configuration

```python
NEXUS_HOST = "0.0.0.0"
NEXUS_PORT = 8765
HEARTBEAT_INTERVAL = 5
HEARTBEAT_TIMEOUT = 3
STATE_STORE_PATH = "~/.nexus/state/"
JBT_LIBRARY_PATH = "~/.nexus/jbt/"
MASTER_HOSTNAME = "show.joe.bot"
```

---

## Build Priority

1. WebSocket server accepting connections
2. Client registration and heartbeat monitoring
3. Pydantic message models
4. State store with query support
5. Adapter base class and Extron common module
6. Device registry and config loading
7. Intent routing to multiple targets
8. Scene save/recall with .jbt
9. Observatory basic connection and state display
10. Pattern execution scaffolding

---

## Longer Term Roadmap

- Authentication between clients
- MQTT bridge for very high device counts
- Remote access over internet with auth
- Full logging and traffic playback for debugging
- iOS Observatory companion app

---

## Related Documents

- `DirtyMixerApp_BuildGuide.md` — DirtyMixerApp Swift app spec
- `JBT_Format_Spec.md` — .jbt file format canonical reference
- `Ecosystem_Overview.md` — full Joebot ecosystem map (TODO)

---

*Nexus — Central coordinator for the Joebot studio ecosystem*
*github.com/joebot94/nexus*
*Document version 1.1 — March 2026*
