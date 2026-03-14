# DirtyMixerApp — Build Guide & Briefing Document
> For use with Claude Code, Cursor, or any AI coding assistant

---

## What This Project Is

DirtyMixerApp is a native Mac/iOS Swift application that controls a custom-built 9-channel analog dirty video mixer board. It acts as the translation and intelligence layer between higher-level show control software (Atlas server) and the dumb analog hardware board.

This document covers hardware context, software architecture, short term build priorities, and longer term roadmap.

---

## Hardware Context

### The Board (Physical Device)

- 9 independent dirty mixer channels
- Each channel has 2 composite video inputs (A and B) and 1 composite video output
- Signals are mixed in a crude/intentionally dirty analog way — sync fights, brightness wars, unstable hybrids
- This is NOT a clean broadcast mixer. Dirty behavior is the point.
- One central microcontroller (RP2350-based board with USB-C) manages all 9 channels
- Each channel has a digipot (MCP4151 or similar, 8-10 bit) that controls the mix parameter
- Controller communicates via SPI to digipots, CS lines per channel for addressing
- USB-C connection to host computer (V1)
- Ethernet planned for later versions

### Per-Channel Control Model (V1)

Each channel exposes these parameters:
- `input_a_enabled` — bool
- `input_b_enabled` — bool  
- `mix` — int 0–255 (or 0–1023 for 10-bit)

### Board Communication Protocol (V1, USB Serial)

Simple ASCII command format over USB serial:

```
CH1 MIX 128
CH3 A ON
CH7 B OFF
CH1 MIX 255
```

Board acknowledges with OK or ERR responses. DirtyMixerApp handles all protocol details — nothing above this layer ever speaks raw board protocol.

### Hardware Versions Planned

- Single cell prototype (breadboard, 1 channel, RCA jacks)
- 3-channel test board (EasyEDA/JLCPCB, proving multi-channel control)
- 9-channel full board (final version, rack mount BNC, desktop RCA, portable 3.5mm variants)

---

## System Architecture

### Three Layer Stack

```
[ Atlas / Server / Other Apps ]
           ↓ high-level intent
   [ DirtyMixerApp ]
           ↓ board protocol
      [ Hardware Board ]
```

**Atlas / Server / Other Apps** — send intent:
- "Recall preset 12"
- "Ramp channels 1,4,6,9 from 0 to 120 to 0 over 10 seconds"
- "Enable random mode on channel 7"

**DirtyMixerApp** — handles everything in between:
- Manages USB/Ethernet connection to board
- Translates high-level commands into channel parameter updates
- Runs automation, timelines, envelopes
- Manages presets
- Reports state back to server if needed
- Provides manual UI for direct control

**Hardware Board** — dumb execution:
- Receives parameter updates
- Applies them to digipots
- Does not make decisions
- Does not store show data

---

## DirtyMixerApp — Two Operating Modes

### Standalone Mode
DirtyMixerApp connects directly to board and acts as full controller:
- Manual channel control UI
- Preset load/save/recall
- Timeline playback
- Automation execution
- No server required

### Managed Mode
DirtyMixerApp is controlled by Atlas or other apps:
- Accepts high-level commands over network/IPC
- Translates to board actions
- Reports state back
- UI reflects externally driven state

---

## .jbt File Format

`.jbt` is the shared Joebot ecosystem file format. For DirtyMixerApp it covers:

| Type | Description |
|---|---|
| `dirtymixer_preset` | A stored board state snapshot |
| `dirtymixer_timeline` | Keyframed automation over time |
| `dirtymixer_clip` | A reusable automation segment |
| `dirtymixer_project` | Full session including presets and timelines |

Structure is JSON-based with a type field at root:

```json
{
  "jbt_type": "dirtymixer_preset",
  "version": "1.0",
  "name": "Preset 12 — Chaos Mode",
  "channels": [
    { "id": 1, "a": true, "b": true, "mix": 128 },
    { "id": 2, "a": true, "b": false, "mix": 255 },
    ...
  ]
}
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| UI Framework | SwiftUI |
| Platform targets | macOS first, iOS/iPadOS later |
| Serial/USB comms | Swift + IOKit or serialport library |
| Networking (later) | Swift Network framework |
| File format | JSON (.jbt) |
| State management | ObservableObject / @Published |

---

## Short Term Build Priorities

### Phase 1 — UI Skeleton (Current Focus)

Build the visual shell of DirtyMixerApp with no real hardware connection yet. Use simulated/mock board state.

**Target UI components:**

1. **Channel Strip View** — one per channel (9 total)
   - Channel number label
   - Input A toggle
   - Input B toggle
   - Mix slider (0–255)
   - Mix value display
   - Active/inactive indicator

2. **Main Board View**
   - Grid of 9 channel strips
   - Connection status indicator (Connected / Disconnected)
   - Board name/port display

3. **Preset Bar**
   - Row of preset slots (12 minimum)
   - Recall button per slot
   - Save-to-slot button
   - Preset name display

4. **App Toolbar**
   - Connect/Disconnect button
   - Load project button
   - Save project button
   - Standalone / Managed mode indicator

**Mock data model for UI development:**

```swift
struct ChannelState: Identifiable, ObservableObject {
    let id: Int
    @Published var inputAEnabled: Bool = true
    @Published var inputBEnabled: Bool = true
    @Published var mix: Double = 128
}

class BoardState: ObservableObject {
    @Published var channels: [ChannelState] = (1...9).map { ChannelState(id: $0) }
    @Published var isConnected: Bool = false
    @Published var portName: String = "Not Connected"
}
```

### Phase 2 — USB Serial Connection

- Port discovery and connection UI
- Send channel commands to real hardware
- Receive ACK/ERR responses
- Connection state management
- Reconnection handling

### Phase 3 — Preset System

- Save current board state as named preset
- Recall preset (instant snap)
- Recall preset with interpolation over duration
- Load/save .jbt preset files

### Phase 4 — Automation & Timeline

- Envelope shapes: linear, triangle, stepped
- Group channel automation
- Timeline playback with transport controls
- Load/save .jbt timeline files

---

## Longer Term Roadmap

### Hardware Expansion
- 3-channel prototype board
- 9-channel full board (rack BNC, desktop RCA, portable 3.5mm)
- Ethernet module on board
- Onboard preset storage

### Software Expansion
- Managed mode / Atlas integration
- Network control (replace USB with Ethernet)
- iOS/iPadOS companion app
  - Full channel control
  - Preset recall
  - Performance-focused UI
  - WiFi connected to same board
- Random mode engine (app-driven first)
- Seed-based deterministic random for show sync
- OSC or similar protocol for Atlas integration

### Ecosystem Integration
- DirtyMixerApp receives cues from Atlas
- Lyric app / tile app / text wall app trigger presets via Atlas
- .jbt project files shareable across ecosystem apps
- Show file bundles multiple .jbt types into one package

---

## Naming & Conventions

- App name: **DirtyMixerApp**
- Board protocol prefix: `CH` followed by channel number (1-indexed)
- File extension: `.jbt`
- Channel parameters: `mix` (0–255 V1, 0–1023 if 10-bit digipot)
- All channel IDs are 1-indexed (CH1 through CH9)

---

## Key Design Rules

1. **Board is dumb** — it only applies state, never decides anything
2. **DirtyMixerApp is the only thing that speaks board protocol** — nothing above it ever sends raw commands
3. **Standalone mode always works** — server being offline never breaks direct control
4. **Dirty behavior is preserved** — the software layer should never sanitize or smooth the analog weirdness out of existence
5. **Swift native all the way** — Mac first, iOS companion later, no cross-platform compromises

---

## First Session Prompt for Claude Code

> "I am building DirtyMixerApp, a native SwiftUI macOS application that controls a 9-channel analog dirty video mixer board over USB serial. I need to start with the UI skeleton using mock data — no real hardware connection yet. Please build the main app structure including a BoardState model with 9 ChannelState objects, a ChannelStripView showing input A toggle, input B toggle, mix slider and value, and a MainBoardView arranging 9 channel strips in a grid. Use a dark theme appropriate for AV/studio software. Refer to DirtyMixerApp_BuildGuide.md for full architecture context."

---

*Document version 1.0 — generated from design session March 2026*
*Hardware status: concept/planning phase*
*Software status: pre-development*
