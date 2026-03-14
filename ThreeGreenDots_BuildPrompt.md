# Tonight's Build Session — Three Green Dots
> Paste this entire prompt at the start of a Claude Code or Codex session
> Goal: Nexus running + DirtyMixerApp connected + Glitch Catalog Swift shell connected + Observatory basic
> End state: 🦖🟢🟢🟢

---

## Context

I am building the Joebot Ecosystem — a suite of apps for controlling a glitch art studio. The central piece is a Python server called **Nexus** (port 8675) that coordinates all apps and devices. No app talks directly to another — everything goes through Nexus.

Full architecture docs are at:
- `https://raw.githubusercontent.com/joebot94/docs/main/Nexus_Architecture.md`
- `https://raw.githubusercontent.com/joebot94/docs/main/JoebotSDK_Guide.md`
- `https://raw.githubusercontent.com/joebot94/docs/main/Observatory_BuildGuide.md`

Please read these before starting.

---

## What Already Exists

### 1. DirtyMixerApp — Swift/SwiftUI — ALREADY BUILT
A SwiftUI macOS app with:
- 9 channel strips in a 3x3 grid
- Input A and B toggles per channel
- Mix slider 0-255 per channel
- Preset slots across the top (Recall/Save per slot)
- Connected/Disconnected status indicator
- Dark theme with orange accents

**What it needs added tonight:**
- NexusStatusIndicator in the top right toolbar
- NexusClient connection to Nexus on localhost port 8675
- Register with Nexus as `dirtymixer_v1`
- Send heartbeat every 5 seconds
- Send state update to Nexus when any channel parameter changes
- Green dot when connected, red when not

### 2. Glitch Catalog — Python/Qt6 — EXISTS BUT NEEDS SWIFT REWRITE
Current Python version uses SQLite database with these tables:
- `session` — id, title, date, location, notes
- `tape` — session_id, tape_id, format, label, storage_location, notes
- `gear` — id, name
- `session_gear` — session_id, gear_id, notes, photos
- `media` — session_id, file_path, kind, checksum, duration, width, height, codec, created_at, notes, thumbnail_path
- `tag` — id, name
- `session_tag` — session_id, tag_id

**What to build tonight:**
A basic Swift/SwiftUI shell that:
- Has the same three column layout as the Python version
- Left column: session list
- Middle column: tapes and gear chain
- Right column: digital files/media
- DOS/Win3.11 theme — navy background, cyan/white text, chunky bordered buttons
- NexusStatusIndicator in toolbar
- Connects to Nexus as `glitch_catalog`
- Sends heartbeat every 5 seconds
- Has a "Snapshot" button that sends scene_save request to Nexus
- Uses .jbt files instead of SQLite (migration tool for existing data is a future task)
- Works perfectly without Nexus connected — Snapshot button just grays out

---

## What Needs To Be Built From Scratch Tonight

### 3. Nexus — Python Server
The central coordinator. Full spec at the GitHub URL above.

**Core requirements:**
- Python 3.11+
- asyncio + websockets
- Listen on 0.0.0.0 port **8675** (yes 8675, like 867-5309 Jenny 🦖)
- Accept multiple simultaneous WebSocket connections
- Handle these message types:
  - `register` — store client info, respond with `registered`
  - `heartbeat` — track last seen, detect offline after 3 missed beats
  - `state_update` — store client state
  - `query` — return stored state for requested client
  - `intent` — fan out to target clients simultaneously
  - `capabilities.query` — forward to target, return capabilities to requester
  - `scene_save` — poll all clients for state, bundle response
- State store — in memory dict, retain last known state when client goes offline
- Mark client offline after 15 seconds no heartbeat
- Alert monitor clients when something goes offline or comes back online
- Central log bus — log everything with timestamp
- Startup message must include 🦖

**Folder structure:**
```
nexus/
├── main.py
├── requirements.txt
├── README.md
├── config/
│   └── settings.py          # NEXUS_PORT = 8675
├── core/
│   ├── registry.py
│   ├── state_store.py
│   ├── heartbeat.py
│   └── log_bus.py
├── api/
│   ├── models.py            # Pydantic v2 message models
│   ├── websocket_server.py
│   └── handlers.py
└── test_client.py
```

### 4. Observatory — Swift/SwiftUI — NEW
Basic monitoring dashboard. Full spec at GitHub URL above.

**What to build tonight — minimal viable version:**
- SwiftUI macOS app
- Dark theme, orange accents — Joebot Classic
- NexusStatusIndicator top right
- Connects to Nexus as `observatory` with type `monitor`
- Main view is a LazyVGrid of DeviceCards
- DeviceCard shows:
  - Client ID and type
  - 🟢 green dot when online
  - 🔴 red dot when offline
  - Last known state summary
  - "Open App" button (stub for now, just prints to console)
- Cards appear automatically when clients connect
- Cards gray out when clients go offline
- Empty state shows 🦖 and "Waiting for connections..."
- Works without Nexus — shows "Nexus Offline" banner

---

## JoebotSDK — Shared Swift Package

Both DirtyMixerApp and Observatory and Glitch Catalog Swift need shared code. Create a local Swift Package called JoebotSDK with:

**NexusClient.swift:**
```swift
class NexusClient: ObservableObject {
    @Published var isConnected: Bool = false
    @Published var connectedClients: [NexusClientInfo] = []
    
    func connect(to host: String, port: Int)
    func disconnect()
    func sendHeartbeat()
    func sendStateUpdate(_ state: [String: Any])
    func sendIntent(targets: [String], action: String, params: [String: Any])
    func requestCapabilities(of clientId: String)
}
```

**NexusStatusIndicator.swift:**
```swift
// Small dot in toolbar — green/yellow/red
// Tap opens popover with:
// - Server address field (accepts IP or hostname like nexus.joe.bot)
// - Port field (default 8675)
// - Auto-connect toggle
// - Connect/Disconnect button
// - Status and uptime display
struct NexusStatusIndicator: View {
    @ObservedObject var client: NexusClient
}
```

**NexusMessage.swift:**
```swift
// Pydantic-equivalent Swift structs for all message types
struct NexusMessage: Codable {
    let id: String
    let type: String
    let source: String
    let payload: [String: AnyCodable]
}
```

---

## Message Format

All messages use this envelope:
```json
{
  "id": "msg_001",
  "type": "message_type",
  "source": "client_id",
  "payload": {}
}
```

---

## Tonight's Success Criteria

When this session is done:

1. `python main.py` starts Nexus — see 🦖 in terminal output on port 8675
2. `python test_client.py` connects — see registration in Nexus logs
3. DirtyMixerApp launches — green dot appears, registered in Nexus
4. Glitch Catalog Swift launches — green dot appears, registered in Nexus
5. Observatory launches — shows cards for both connected apps
6. Move a slider in DirtyMixerApp — Observatory card updates
7. Kill DirtyMixerApp — Observatory card goes gray
8. Relaunch DirtyMixerApp — Observatory card goes green again
9. Click Snapshot in Glitch Catalog — Nexus logs show scene_save request

**🦖🟢🟢🟢 — three green dots simultaneously in Observatory**

That's the screenshot. That's the proof. Everything else builds on this.

---

## Important Notes

- Nexus port is **8675** — not 8765. 8675. Like 867-5309. 🦖📞
- 🦖 must appear in Nexus startup logs. Non negotiable.
- Every app works without Nexus — graceful degradation always
- Nexus-dependent features gray out cleanly when disconnected
- Do NOT build Extron adapters yet
- Do NOT build full .jbt file system yet — stubs are fine
- Do NOT build scene management yet
- Focus on WebSocket layer and three green dots

---

## Code Quality

- Type hints throughout Python
- Pydantic v2 for all Nexus message models
- SwiftUI best practices — ObservableObject, @Published, environmentObject
- Small focused files — nothing monolithic
- Graceful error handling everywhere
- 🦖 in file header comments is encouraged

---

## Requirements

**Nexus (requirements.txt):**
```
websockets>=12.0
pydantic>=2.0
```

**Swift packages:**
- No external dependencies beyond JoebotSDK (local package)
- Standard Foundation and Network frameworks only

---

*Three green dots. One server. One dino. LFG.*
*🦖🟢🟢🟢*
*github.com/joebot94*
*Joebot Ecosystem — March 2026*
