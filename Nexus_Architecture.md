# Nexus — Grand Server Architecture
> Joebot Ecosystem Central Coordinator
> GitHub: github.com/joebot94/nexus
> Document version 1.2 — March 2026
> Changes: Corrected architecture — Nexus owns ALL hardware adapters. Atlas and DirtyMixerApp are UI clients only. DirtyMixerApp exception noted for USB serial.

---

## The One Sentence Version

**Apps speak Nexus. Nexus speaks hardware. Users see magic.** 🦖

---

## What Nexus Is

Nexus is the central coordination server for the entire Joebot studio ecosystem. It is the single point through which all apps communicate AND the only thing that ever talks to physical hardware.

Every app is just a UI. Nexus is the brain, the translator, and the executor.

---

## The Final Architecture — Locked

```
┌─────────────────────────────────────────────────────────┐
│                      NEXUS                              │
│                                                         │
│  Hardware Adapters:                                     │
│  MTPXAdapter     → mtpx1.extron.video:23               │
│  MGPAdapter      → mgp1.extron.video:23                │
│  DMSAdapter      → dms.extron.video:23                 │
│  MatrixAdapter   → mx.extron.video:23                  │
│  IPCPAdapter     → ipcp505-1.extron.video (HTTP)       │
│  IPLAdapter      → iplt-s4.extron.video:23             │
│  DirtyMixerAdap  → via DirtyMixerApp (see exception)  │
└─────────────────────────────────────────────────────────┘
         ↑↓         ↑↓        ↑↓       ↑↓        ↑↓
    GlitchBoard   Atlas  DirtyMixer  Glitch   Observatory
    (timeline)   (Extron  (Mixer    Catalog   (monitor)
                  UI)      UI)      (archive)
```

**Every app is a UI client. Nexus handles all hardware.**

---

## App Roles

| App | Role | Hardware Access |
|---|---|---|
| GlitchBoard | Timeline/DAW show control UI | None — sends intents to Nexus |
| Atlas | Extron gear control UI | None — sends intents to Nexus |
| DirtyMixerApp | Mixer control UI + board owner | USB Serial to board only (see exception) |
| Glitch Catalog | Session archive UI | None — sends intents to Nexus |
| Observatory | Monitor and launcher UI | None — read only from Nexus |
| Text Wall | Display UI | None — receives commands from Nexus |
| DAW App | Beat sync UI | None — sends intents to Nexus |
| MIDI App | Controller mapping UI | None — sends intents to Nexus |

---

## The DirtyMixerApp Exception

DirtyMixerApp is the one exception to the rule.

DirtyMixerApp communicates directly with the physical dirty mixer board over USB serial. This is because:
- The dirty mixer board is a custom built device
- It connects via USB-C to the host Mac
- USB serial is inherently local and direct
- DirtyMixerApp is the designated owner of that hardware

**However** — DirtyMixerApp still participates in Nexus fully:
- Registers with Nexus on launch
- Reports board state to Nexus constantly
- Receives commands FROM Nexus when other apps want to change something
- When GlitchBoard wants the mixer to do something it goes through Nexus → DirtyMixerApp → USB → Board

DirtyMixerApp is the hardware owner. Nexus is the coordinator. They work together.

**The dirty mixer flow:**
```
GlitchBoard → Nexus → DirtyMixerApp → USB Serial → Board
```

**DirtyMixerApp reporting state:**
```
Board state changes → DirtyMixerApp → Nexus state store → all apps notified
```

---

## What Atlas Is

Atlas is a beautiful SwiftUI control surface for Extron gear. It is NOT a hardware controller.

Atlas sends intents to Nexus in plain Nexus protocol. Nexus translates those intents into SIS commands via its adapters. Atlas has zero knowledge of SIS syntax, TCP connections, or device hostnames.

**Atlas does:**
- Provides UI for recalling presets, adjusting skew, firing IR, changing routing
- Shows live device status from Nexus state store
- Sends intents to Nexus
- Reports its own UI state to Nexus

**Atlas does NOT:**
- Speak SIS
- Open TCP connections to Extron devices
- Know device hostnames or ports
- Talk to any hardware directly

---

## What Nexus Does

Nexus is the only thing with hardware knowledge. It has an adapter for every device type.

**Nexus responsibilities:**
- Accept WebSocket connections from all apps
- Route messages between apps
- Maintain state store for every connected client
- Handle scene save and recall
- Own and manage ALL hardware adapters
- Translate logical actions into device-native commands
- Execute hardware commands directly
- Handle unsolicited responses from hardware (front panel knob changes etc)
- Log all events with timestamps for session recording

---

## Hardware Ownership

| Hardware | Owner | Protocol |
|---|---|---|
| MTPX Plus series | Nexus MTPXAdapter | Extron SIS TCP port 23 |
| MGP 464 | Nexus MGPAdapter | Extron SIS TCP port 23 |
| DMS 3600 | Nexus DMSAdapter | Extron SIS TCP port 23 |
| Matrix 12800 | Nexus MatrixAdapter | Extron SIS TCP port 23 |
| IPCP 505 | Nexus IPCPAdapter | HTTP + Extron SIS |
| IPL T series | Nexus IPLAdapter | HTTP + Extron SIS |
| VSC series | Nexus VSCAdapter | Via IPCP serial passthrough |
| DSC 401A | Nexus DSCAdapter | Extron SIS TCP port 23 |
| Dirty Mixer Board | DirtyMixerApp | USB Serial (exception) |

---

## Action Flow Examples

**GlitchBoard wants MTPX blue skew at maximum:**
```
GlitchBoard sends intent to Nexus:
  action: set_input_skew
  device: device.mtpx.1
  params: {input: 3, red: 0, green: 0, blue: 31}

Nexus MTPXAdapter translates:
  3*0*0*31*4Iseq↵

Nexus sends to mtpx1.extron.video:23
Hardware responds
Nexus updates state store
All apps notified of new state
```

**Atlas UI user recalls MGP preset 5:**
```
Atlas sends intent to Nexus:
  action: recall_preset
  device: device.mgp.1
  params: {preset: 5}

Nexus MGPAdapter translates:
  5.↵

Nexus sends to mgp1.extron.video:23
Hardware responds
Nexus updates state store
```

**GlitchBoard wants dirty mixer channel 3 mix at 200:**
```
GlitchBoard sends intent to Nexus:
  action: set_channel_mix
  device: device.dirtymixer.1
  params: {channel: 3, mix: 200}

Nexus routes to DirtyMixerApp (exception path)
DirtyMixerApp sends: CH3 MIX 200↵ over USB serial
Board responds OK
DirtyMixerApp reports new state to Nexus
Nexus updates state store
```

**IPCP relay pulse on beat:**
```
GlitchBoard sends intent to Nexus:
  action: pulse_relay
  device: device.ipcp505.1
  params: {relay: 1}

Nexus IPCPAdapter fires:
  GET http://ipcp505-1.extron.video/W=1R01

Relay clicks on front panel
```

---

## Unsolicited Hardware Responses

When a user turns a knob on the front panel of an Extron device the device sends an unsolicited response back over TCP. Nexus adapters must listen continuously and handle these responses.

Example: User turns MTPX horizontal skew knob manually
```
MTPX sends: Iseq03•00•00•15↵  (unprompted)
Nexus MTPXAdapter receives and parses it
Nexus updates state store for device.mtpx.1
All apps notified — Atlas UI updates to show new value
Glitch Catalog event log records the change with timestamp
```

This is critical for session recording — every physical hardware change gets captured automatically.

---

## Core Responsibilities

- Accept and maintain persistent WebSocket connections from all clients
- Route messages between clients based on intent
- Maintain a live state store of every connected app and device
- Hold last known state when a client goes offline
- Alert Observatory when a client drops unexpectedly
- Handle .jbt scene snapshots — poll all clients and adapters, bundle state
- Provide a queryable API so any client can ask "what is X doing right now"
- Own and execute ALL hardware adapters (except dirty mixer — see exception)
- Listen for unsolicited hardware responses and update state accordingly
- Log all events with timestamps for session replay

---

## What Nexus Is NOT

- Not a UI — Observatory, Atlas, GlitchBoard etc are the UIs
- Not a file manager — Glitch Catalog handles session archives
- Not a creative decision maker — it only routes what clients tell it to
- Not aware of show logic — apps handle their own timelines and automation

---

## Capability Discovery

Nexus exposes device capabilities so apps can build dynamic UIs without hardcoding device knowledge.

When GlitchBoard asks "what can the MTPX do":
```json
{
  "type": "capabilities.query",
  "payload": { "target_client_id": "device.mtpx.1" }
}
```

Nexus responds from its adapter knowledge:
```json
{
  "capabilities": {
    "actions": [
      {
        "action": "set_input_skew",
        "params": {
          "input": {"type": "int", "range": [1,12]},
          "red": {"type": "int", "range": [0,31]},
          "green": {"type": "int", "range": [0,31]},
          "blue": {"type": "int", "range": [0,31]}
        }
      },
      {
        "action": "recall_preset",
        "params": {
          "preset": {"type": "int", "range": [1,32]}
        }
      }
    ]
  }
}
```

GlitchBoard builds its cue editor dropdowns from this response. No hardcoding. Add a new device — Nexus has the adapter — every app immediately gets access to its capabilities.

---

## Connection Model

### Client Registration

Every app registers on connect:
```json
{
  "type": "register",
  "client_id": "atlas",
  "client_type": "extron_controller",
  "version": "1.0.0",
  "capabilities": ["preset_recall", "skew_control", "routing"]
}
```

### Heartbeat

Every client sends heartbeat every 5 seconds. Miss 3 = marked offline. Last known state retained.

### State Reporting

Clients report state whenever something changes. Nexus stores it. Anyone can query it.

---

## Nexus Server Structure

```
nexus/
├── main.py
├── requirements.txt
├── config/
│   ├── devices.json
│   ├── actions.json
│   ├── scenes.json
│   └── patterns.json
├── core/
│   ├── registry.py
│   ├── state_store.py
│   ├── heartbeat.py
│   ├── log_bus.py
│   ├── dispatcher.py
│   └── adapter_manager.py
├── api/
│   ├── models.py
│   ├── websocket_server.py
│   └── handlers.py
├── adapters/
│   ├── base.py
│   ├── extron_common.py      ← shared SIS helpers
│   ├── mtpx_adapter.py       ← skew, peaking, routing
│   ├── mgp_adapter.py        ← preset recall, mute
│   ├── dms_adapter.py        ← preset recall, mute
│   ├── matrix_adapter.py     ← routing, presets
│   ├── ipcp_adapter.py       ← relay, IR, serial passthrough
│   ├── ipl_adapter.py        ← serial passthrough
│   └── dirtymixer_adapter.py ← routes to DirtyMixerApp (exception)
└── jbt/
    ├── parser.py
    └── writer.py
```

---

## Device Registry

| Device ID | Label | Adapter | Hostname | Port |
|---|---|---|---|---|
| `device.mtpx.1` | MTPX Plus #1 | MTPXAdapter | mtpx1.extron.video | 23 |
| `device.mtpx.2` | MTPX Plus #2 | MTPXAdapter | mtpx2.extron.video | 23 |
| `device.mgp.1` | MGP 464 #1 | MGPAdapter | mgp1.extron.video | 23 |
| `device.mgp.2` | MGP 464 #2 | MGPAdapter | mgp2.extron.video | 23 |
| `device.mgp.3` | MGP 464 #3 | MGPAdapter | mgp3.extron.video | 23 |
| `device.dms.main` | DMS 3600 | DMSAdapter | dms.extron.video | 23 |
| `device.matrix.main` | Matrix 12800 | MatrixAdapter | mx.extron.video | 23 |
| `device.ipcp505.1` | IPCP 505 #1 | IPCPAdapter | ipcp505-1.extron.video | HTTP |
| `device.dirtymixer.1` | Dirty Mixer Board | DirtyMixerAdapter* | via DirtyMixerApp | USB |

*DirtyMixerAdapter routes through DirtyMixerApp — it does not connect directly to hardware.

---

## Configuration

```python
NEXUS_HOST = "0.0.0.0"
NEXUS_PORT = 8675          # Jenny 📞
HEARTBEAT_INTERVAL = 5     # seconds
HEARTBEAT_TIMEOUT = 3      # missed beats before offline
STATE_STORE_PATH = "~/.nexus/state/"
JBT_LIBRARY_PATH = "~/.nexus/jbt/"
MASTER_HOSTNAME = "show.joe.bot"
```

---

## Deployment Options

### Single Machine (typical user)
- Nexus runs as background service on same Mac as all apps
- Apps connect to localhost:8675
- User never interacts with Nexus directly

### Dedicated Server (studio setup)
- Nexus runs on N100 mini PC or Mac Mini
- Apps connect over local network to nexus.joe.bot:8675
- Always on, headless, silent

---

## Related Documents

- `JBT_Format_Spec.md` — shared file format
- `Extron_SIS_Reference.md` — device command reference
- `DirtyMixerApp_BuildGuide.md` — dirty mixer app spec
- `Observatory_BuildGuide.md` — Observatory spec
- `GlitchBoard_Spec.md` — GlitchBoard DAW spec

---

*Nexus — Central coordinator for the Joebot studio ecosystem*
*Apps speak Nexus. Nexus speaks hardware. Users see magic.* 🦖
*github.com/joebot94/nexus*
*Document version 1.2 — March 2026*
