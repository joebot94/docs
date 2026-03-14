# JoebotSDK — Shared Swift Toolkit
> Joebot Ecosystem Shared Foundation
> GitHub: github.com/joebot94/joebotsdk
> Document version 1.0 — March 2026

---

## What JoebotSDK Is

JoebotSDK is a shared Swift package that every Joebot ecosystem app imports. It contains common code, UI components, data models, and utilities that would otherwise be duplicated across every app.

Write it once. Every app gets it for free.

---

## Why It Exists

Without a shared SDK every app would:
- Implement Nexus WebSocket connection slightly differently
- Parse .jbt files with slightly different logic
- Show a slightly different Nexus status indicator
- Define slightly different data models for the same concepts
- Drift apart over time as each app evolves independently

With JoebotSDK:
- Fix a Nexus connection bug once — every app gets the fix
- Update the .jbt parser once — every app reads the new format
- Change the Nexus indicator design once — every app looks consistent
- Share data models — apps can hand objects to each other cleanly

---

## Tech Stack

| Component | Technology |
|---|---|
| Package type | Swift Package Manager |
| Platform targets | macOS 13+, iOS 16+ |
| UI framework | SwiftUI |
| Networking | Swift Network framework / URLSession WebSocket |
| File I/O | Foundation / Codable |
| Minimum Xcode | 15+ |

---

## What Lives in JoebotSDK

### 1. Nexus Client

The complete Nexus WebSocket connection layer. Every app imports this instead of rolling its own.

**Responsibilities:**
- WebSocket connection management
- Auto-reconnection with backoff
- Client registration on connect
- Heartbeat sending every 5 seconds
- Message sending and receiving
- Connection state tracking
- Capability discovery requests

**Usage in any app:**
```swift
// In your app's main state object
@StateObject var nexus = NexusClient(
    clientId: "dirtymixer_v1",
    clientType: "dirtymixer",
    capabilities: ["presets", "automation"]
)

// Connect
nexus.connect(to: "nexus.joe.bot", port: 8765)

// Send state update
nexus.sendStateUpdate(state: currentBoardState)

// Send intent
nexus.sendIntent(targets: ["textwall_v1"], action: "recall_preset", params: ["preset_id": 12])

// Receive messages
nexus.onMessage = { message in
    // handle incoming message
}
```

---

### 2. NexusStatusIndicator

The standard Nexus connection indicator that appears in every app. Consistent design, consistent behavior, consistent position — top right corner of every app.

**Visual states:**
- 🟢 Green dot — Connected
- 🔴 Red dot — Disconnected  
- 🟡 Yellow dot — Connecting / Reconnecting

**Behavior:**
- Tap → opens NexusSettingsPopover
- Always visible, never intrusive

**NexusSettingsPopover contains:**
```
┌─ Nexus ──────────────────────────┐
│  ● Connected                     │
│                                  │
│  Server                          │
│  [ nexus.joe.bot              ]  │
│                                  │
│  Port                            │
│  [ 8765 ]                        │
│                                  │
│  ☑ Auto-connect on launch        │
│                                  │
│  Connected as: dirtymixer_v1     │
│  Uptime: 00:42:17                │
│                                  │
│  [ Disconnect ]                  │
└──────────────────────────────────┘
```

**Accepts either:**
- IP address — `192.168.1.100`
- Hostname — `nexus.joe.bot`
- Local mDNS — `nexus.local`

**Usage in any app:**
```swift
// Drop into any toolbar
ToolbarItem(placement: .topBarTrailing) {
    NexusStatusIndicator(client: nexus)
}
```

One line. Every app gets consistent Nexus status UI.

---

### 3. JBT Parser and Writer

Complete .jbt file read/write implementation shared across all apps.

**Reading:**
```swift
let session = try JBT.load(from: url, as: GlitchSession.self)
let preset = try JBT.load(from: url, as: DirtyMixerPreset.self)
```

**Writing:**
```swift
try JBT.save(session, to: url)
try JBT.save(preset, to: url)
```

**Type detection:**
```swift
let type = try JBT.detectType(at: url)
// Returns "glitch_session", "dirtymixer_preset", etc.
```

Handles versioning, unknown fields, migration between versions.

---

### 4. Shared Data Models

Swift structs and classes for data types that multiple apps need to understand.

**Models included:**
- `NexusMessage` — standard message envelope
- `ClientRegistration` — registration payload
- `ClientCapabilities` — capability discovery response
- `JBTRoot` — base .jbt file structure
- `GlitchSession` — glitch_session .jbt type
- `DirtyMixerPreset` — dirtymixer_preset .jbt type
- `DirtyMixerChannel` — per channel state
- `ExtronSnapshot` — extron_snapshot .jbt type
- `NexusScene` — nexus_scene .jbt type

---

### 5. Joebot Theme System

Complete SwiftUI theming system. All apps share the same theme definitions and switch themes consistently.

**Available themes:**

| Theme ID | Name | Description |
|---|---|---|
| `joebot` | Joebot Classic | Dark grey and orange — signature look |
| `joebot_black` | Joebot Black | Full black and orange, high contrast |
| `cyberpunk` | Neo Cyberpunk | Deep purple/blue, cyan accents, neon |
| `dos` | DOS / Win3.11 | Navy background, cyan/white text |
| `amber` | Amber Terminal | Black background, amber/golden text |

**Usage:**
```swift
// Apply theme to entire app
@StateObject var theme = JoebotTheme.current

ContentView()
    .environmentObject(theme)
    .joebotTheme(theme)
```

**Switching themes:**
```swift
JoebotTheme.current.set(.dos)
// Entire app redraws instantly
```

**Theme definition structure:**
```swift
struct ThemeDefinition {
    let background: Color
    let surface: Color
    let accent: Color
    let text: Color
    let textSecondary: Color
    let border: Color
    let success: Color
    let warning: Color
    let error: Color
    let fontPrimary: String
    let fontMono: String
}
```

---

### 6. Common UI Components

Reusable SwiftUI views used across multiple apps.

**Components included:**

| Component | Description |
|---|---|
| `NexusStatusIndicator` | Nexus connection dot and settings popover |
| `JoebotButton` | Standard themed button |
| `StatusDot` | Green/yellow/red status indicator dot |
| `SectionHeader` | Consistent section header style |
| `EmptyStateView` | Standard empty state with icon and message |
| `LoadingView` | Standard loading indicator |
| `ErrorBanner` | Non-intrusive error display |
| `CapabilityGrid` | Dynamic grid built from capability data |
| `PresetGrid` | Tappable preset selector grid |
| `ChannelSelector` | Multi-select channel picker with quick select |

**CapabilityGrid and PresetGrid** are particularly important for the DAW app — they build themselves dynamically from whatever capability data Nexus returns.

---

## Capability Discovery Helper

JoebotSDK includes a helper for capability discovery so any app can ask Nexus what another app can do:

```swift
// Ask Nexus what the dirty mixer can do
nexus.queryCapabilities(of: "dirtymixer_v1") { capabilities in
    // capabilities.channels == 9
    // capabilities.presets == 24
    // capabilities.mixRange == 0...255
    // Build your UI from this
}
```

The CapabilityGrid component accepts capabilities directly:

```swift
CapabilityGrid(
    capabilities: dirtymixerCapabilities,
    onSelection: { selectedItems in
        // User selected these presets/channels/whatever
    }
)
```

---

## App Integration Pattern

Every Joebot app follows this pattern:

```swift
@main
struct DirtyMixerApp: App {
    @StateObject var nexus = NexusClient(
        clientId: "dirtymixer_v1",
        clientType: "dirtymixer",
        capabilities: ["presets", "automation", "random_mode"]
    )
    @StateObject var theme = JoebotTheme(initial: .joebot)
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(nexus)
                .environmentObject(theme)
        }
    }
}
```

From that point every view in the app has access to the Nexus client and theme via environment.

---

## Graceful Nexus Degradation

Every app using JoebotSDK handles Nexus being unavailable gracefully. The SDK provides:

```swift
// Check before using Nexus-dependent features
if nexus.isConnected {
    // Show full feature set
} else {
    // Show degraded but functional UI
}
```

Features that require Nexus are automatically disabled with a clear visual indicator when not connected. Apps never crash or show errors because Nexus is offline — they just show reduced functionality cleanly.

---

## Repo Structure

```
joebotsdk/
├── Package.swift
├── README.md
├── Sources/
│   └── JoebotSDK/
│       ├── Nexus/
│       │   ├── NexusClient.swift
│       │   ├── NexusMessage.swift
│       │   ├── NexusState.swift
│       │   └── CapabilityDiscovery.swift
│       ├── JBT/
│       │   ├── JBTParser.swift
│       │   ├── JBTWriter.swift
│       │   └── Models/
│       │       ├── GlitchSession.swift
│       │       ├── DirtyMixerPreset.swift
│       │       ├── DirtyMixerTimeline.swift
│       │       └── ExtronSnapshot.swift
│       ├── UI/
│       │   ├── NexusStatusIndicator.swift
│       │   ├── CapabilityGrid.swift
│       │   ├── PresetGrid.swift
│       │   ├── ChannelSelector.swift
│       │   └── Common/
│       │       ├── JoebotButton.swift
│       │       ├── StatusDot.swift
│       │       └── SectionHeader.swift
│       └── Theme/
│           ├── JoebotTheme.swift
│           ├── ThemeDefinition.swift
│           └── Themes/
│               ├── JoebotClassic.swift
│               ├── JoebotBlack.swift
│               ├── NeoCyberpunk.swift
│               ├── DOS.swift
│               └── Amber.swift
└── Tests/
    └── JoebotSDKTests/
```

---

## Build Priority

1. NexusClient — WebSocket connection, registration, heartbeat
2. NexusStatusIndicator — the universal Nexus dot
3. JoebotTheme — theme system with Joebot Classic and DOS themes
4. JBT parser/writer — basic read/write for glitch_session and dirtymixer_preset
5. Shared data models
6. CapabilityGrid and PresetGrid — for DAW app
7. Remaining UI components

---

## First Session Prompt for Claude Code

> "I am building JoebotSDK, a Swift Package that is shared across all apps in the Joebot ecosystem. Start by building the NexusClient class that manages a WebSocket connection to the Nexus server, handles registration, sends heartbeats every 5 seconds, tracks connection state, and exposes a simple API for sending and receiving messages. Then build NexusStatusIndicator, a SwiftUI view that shows a green/yellow/red dot indicating connection status and opens a small popover when tapped where the user can enter a server hostname or IP address and port. The popover should show connection status, uptime, and a disconnect button. Use the Joebot Classic dark theme — dark grey background, orange accents."

---

## Related Documents

- `Nexus_Architecture.md` — Nexus server spec
- `Observatory_BuildGuide.md` — Observatory app spec
- `DirtyMixerApp_BuildGuide.md` — DirtyMixerApp spec
- `JBT_Format_Spec.md` — .jbt file format reference

---

*JoebotSDK — Shared Swift foundation for the Joebot ecosystem*
*github.com/joebot94/joebotsdk*
*Document version 1.0 — March 2026*
