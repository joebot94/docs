# Observatory — Build Guide
> Joebot Ecosystem Dashboard and Launcher
> GitHub: github.com/joebot94/observatory
> Document version 1.0 — March 2026

---

## What Observatory Is

Observatory is the native macOS/iOS Swift app that serves as the home base for the entire Joebot ecosystem. It connects to Nexus and displays live status of every connected app and device. It also acts as a launcher — click any device card to open the relevant controller app.

Observatory is a Nexus client like any other app. It has no special privileges. It just subscribes to everything and displays it beautifully.

---

## Two Modes

### Monitoring Mode — Default View

The main screen. A dynamic grid of device cards, one per connected client. Cards appear as devices come online and gray out when they go offline. The grid adjusts automatically — no hardcoded layout.

### Launcher Mode — Click Any Card

Click a device card to open the relevant controller app. Observatory knows which app controls which device and launches it. If the app is already open it brings it to focus.

| Device Card | Launches |
|---|---|
| Dirty Mixer | DirtyMixerApp |
| Atlas / Extron devices | Atlas |
| Glitch Catalog | Glitch Catalog |
| Text Wall | Text Wall App |
| DAW | DAW App |
| Unknown device | Generic detail view |

---

## Tech Stack

| Component | Technology |
|---|---|
| UI Framework | SwiftUI |
| Platform | macOS first, iOS companion later |
| Shared foundation | JoebotSDK |
| Nexus connection | JoebotSDK NexusClient |
| Theme | JoebotSDK theme system |

---

## Device Cards

Each device gets a card. Cards are dynamic — built from whatever state Nexus reports, not hardcoded.

### Card States

**Online — full color:**
```
┌─────────────────────────┐
│ 🟢  Dirty Mixer         │
│                         │
│ Preset: Chaos Mode      │
│ CH1-9: Active           │
│ Board: Connected        │
│                         │
│ [ Open DirtyMixerApp ]  │
└─────────────────────────┘
```

**Offline — grayed out:**
```
┌─────────────────────────┐
│ 🔴  Dirty Mixer         │
│                         │
│ Last seen: 2 min ago    │
│ Last preset: Chaos Mode │
│                         │
│ [ Last Known State ]    │
└─────────────────────────┘
```

**Connecting — yellow:**
```
┌─────────────────────────┐
│ 🟡  Dirty Mixer         │
│                         │
│ Connecting...           │
│                         │
└─────────────────────────┘
```

### Device-Specific Card Content

Each device type shows relevant info in its card:

**DirtyMixerApp card:**
- Connection status to physical board
- Active preset name
- Number of active channels
- Mix values as mini visualizer

**Atlas card:**
- Connected Extron devices count
- Active matrix preset
- MGP states
- Any device warnings

**Glitch Catalog card:**
- Total session count
- Most recent session name and date
- Last snapshot time

**Text Wall card:**
- Current content preview
- Display mode

**Unknown/generic card:**
- Client ID
- Client type
- Last seen timestamp
- Raw capability list

---

## Main Layout

```
┌─────────────────────────────────────────────────────────────┐
│  Observatory                              🟢 nexus.joe.bot  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │🟢 Dirty  │  │🟢 Atlas  │  │🟢 Glitch │  │🔴 Text   │  │
│  │  Mixer   │  │          │  │ Catalog  │  │  Wall    │  │
│  │          │  │          │  │          │  │          │  │
│  │Preset 12 │  │Matrix: 5 │  │24 sessions│  │Offline  │  │
│  │CH 1-9 On │  │MGP: OK   │  │Last: 2h  │  │2min ago  │  │
│  │          │  │          │  │          │  │          │  │
│  │ [ Open ] │  │ [ Open ] │  │ [ Open ] │  │ [ Open ] │  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  │
│                                                             │
│  ┌──────────┐  ┌──────────┐                               │
│  │🟢 DAW    │  │🟢 Obs.   │                               │
│  │          │  │ (self)   │                               │
│  │No song   │  │          │                               │
│  │loaded    │  │Monitor   │                               │
│  │          │  │only      │                               │
│  │ [ Open ] │  │          │                               │
│  └──────────┘  └──────────┘                               │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  Alerts: ⚠️ Text Wall offline for 2 minutes               │
├─────────────────────────────────────────────────────────────┤
│  [ Live Log ]                                               │
└─────────────────────────────────────────────────────────────┘
```

---

## Dynamic Grid Behavior

The card grid is entirely data-driven. No hardcoded device list.

```swift
// Grid builds itself from whatever Nexus reports
LazyVGrid(columns: adaptiveColumns) {
    ForEach(nexus.connectedClients) { client in
        DeviceCard(client: client)
            .onTapGesture {
                launchApp(for: client)
            }
    }
}
```

- Device comes online → card appears automatically
- Device goes offline → card grays out, last known state shown
- New unknown device → generic card appears
- All devices offline → empty state with "Waiting for connections"

---

## Alerts Panel

A slim panel at the bottom of the main view. Shows active alerts from Nexus:

- Client went offline
- Client reconnected
- Any error broadcasts from Nexus

Alerts auto-dismiss after acknowledgement or when the condition resolves.

---

## Live Log

A collapsible panel showing the Nexus event log in real time. Subscribes to `log_event` broadcasts from Nexus.

- Timestamped entries
- Color coded by level — info, warning, error
- Filterable by subsystem
- Auto-scrolls to latest
- Can be hidden when not needed

---

## Scene Controls

Observatory can fire scenes directly without opening another app:

- Quick scene buttons for common scenes
- Populated from Nexus scene registry
- One tap fires the scene through Nexus
- Confirmation shown in alerts panel

---

## Nexus Connection

Observatory uses JoebotSDK NexusClient and registers as:

```json
{
  "client_id": "observatory",
  "client_type": "monitor",
  "version": "1.0.0",
  "capabilities": ["monitoring", "scene_fire", "launch"]
}
```

Observatory subscribes to all state updates and log events. It does not send commands to other apps directly — everything goes through Nexus.

**Nexus offline behavior:**
- Shows all cards in unknown/offline state
- Still launches apps normally
- Displays "Nexus offline" banner
- Auto-reconnects when Nexus comes back

---

## App Launcher Registry

Observatory maintains a local registry mapping client types to app bundle IDs:

```swift
let appRegistry: [String: String] = [
    "dirtymixer": "com.joebot.dirtymixerapp",
    "atlas": "com.joebot.atlas",
    "glitch_catalog": "com.joebot.glitchcatalog",
    "textwall": "com.joebot.textwall",
    "daw": "com.joebot.dawapp"
]
```

When a card is tapped Observatory looks up the bundle ID and launches or focuses that app via NSWorkspace on macOS.

---

## Theme

Observatory uses the Joebot Classic dark theme by default. Theme can be switched in settings — all themes from JoebotSDK are available.

The card grid looks especially good in Neo Cyberpunk theme for live performance use.

---

## iOS Companion

A future iOS/iPadOS version of Observatory is planned. Same functionality, touch-optimized layout. The card grid becomes a scrollable list or larger touch-friendly cards on iPad. Ideal for monitoring the studio from a tablet on a stand.

---

## Repo Structure

```
observatory/
├── ObservatoryApp.swift
├── ContentView.swift
├── Views/
│   ├── MainDashboard.swift
│   ├── DeviceCard.swift
│   ├── AlertsPanel.swift
│   ├── LiveLog.swift
│   └── SceneControls.swift
├── Models/
│   ├── AppRegistry.swift
│   └── AlertModel.swift
├── Services/
│   └── AppLauncher.swift
└── Settings/
    └── ObservatorySettings.swift
```

---

## Build Priority

1. Basic SwiftUI shell with NexusStatusIndicator from JoebotSDK
2. Dynamic device card grid from Nexus state
3. Online/offline card states
4. App launcher — tap card to open app
5. Alerts panel
6. Live log panel
7. Scene fire buttons
8. iOS companion app

---

## Short Term Target

For the proof of concept session the goal is simple:

- Observatory connects to Nexus
- Shows a card for DirtyMixerApp when it connects
- Shows a card for Glitch Catalog when it connects
- Cards go gray when apps disconnect
- Three green dots visible simultaneously — Observatory, DirtyMixerApp, Glitch Catalog all connected

That's the screenshot. That's the proof the architecture works.

---

## First Session Prompt for Claude Code

> "I am building Observatory, a native SwiftUI macOS app that connects to a WebSocket server called Nexus and displays live status cards for every connected client. It uses JoebotSDK for the Nexus connection. Build the main dashboard view with a LazyVGrid of DeviceCard views that are populated dynamically from a list of connected clients. Each card shows the client ID, client type, a green/yellow/red status dot, and basic state info. Cards should gray out when a client goes offline but retain last known state. Use the Joebot Classic dark theme — dark grey background, orange accents. Include the NexusStatusIndicator from JoebotSDK in the top right toolbar."

---

## Related Documents

- `Nexus_Architecture.md` — Nexus server spec
- `JoebotSDK_Guide.md` — shared Swift toolkit
- `DirtyMixerApp_BuildGuide.md` — DirtyMixerApp spec
- `JBT_Format_Spec.md` — .jbt file format reference

---

*Observatory — Dashboard and launcher for the Joebot ecosystem*
*github.com/joebot94/observatory*
*Document version 1.0 — March 2026*
