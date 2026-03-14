# GlitchBoard — Build Specification
> Joebot Ecosystem DAW-Style Show Control App
> GitHub: github.com/joebot94/daw-app
> Document version 1.0 — March 2026
> 🦖 Joebot Ecosystem

---

## What GlitchBoard Is

GlitchBoard is a native SwiftUI macOS DAW-style show control application. It loads audio, displays a waveform with a bar/beat grid, and lets you place cues on per-device lanes that fire through Nexus to control hardware in sync with music.

It is the bridge between music and hardware. Load a song, place cues, hit play, and your MTPX skew changes, dirty mixer presets recall, and Atlas fires routing commands — all synchronized to the beat.

---

## Core Philosophy

- **Reproducible** — every performance can be saved as a .jbt setlist and replayed identically
- **Graceful** — offline devices don't kill the show, they skip or queue
- **Discoverable** — lanes and available actions come from Nexus capability discovery, nothing is hardcoded
- **Musical** — snap to grid, polyrhythms, step sequencer patterns, humanize
- **Live-friendly** — performance dashboard, APC Mini MIDI control, setlist mode

---

## Layout

```
┌─ GlitchBoard ──────────────────────────────────────────────┐
│  [Load Audio] [Play] [Pause] [Stop]  BPM: 140  🟢 Nexus   │
│  Mode: [ ✏️ Edit | ↔️ Select | 🔪 Razor | 👆 Cursor ]     │
├────────────────────────────────────────────────────────────┤
│  BAR: 1    2    3    4    5    6    7    8                 │
│  BEAT:1234 1234 1234 1234 1234 1234 1234 1234             │
│                                                            │
│  ┌─ 🟢 MTPX Plus #1 ───────────────────────────────────┐  │
│  │  ●           ●              ●                       │  │
│  │  skew B=31   skew B=0       preset 5                │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌─ 🟢 Dirty Mixer ────────────────────────────────────┐  │
│  │       ●──────────────●   ●                          │  │
│  │       ramp 0→255     ↑   preset 12                  │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌─ 🔴 Atlas ──────────────────────────────────────────┐  │
│  │  [offline — cues will skip]                         │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                            │
│  ████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   │
│  [waveform — scrolls and zooms in sync with cue lanes]    │
│                                                            │
├────────────────────────────────────────────────────────────┤
│  Snap: [ 1/4 ▼ ]  Zoom: [–][+][Fit]  Theme: [Neon ▼]    │
└────────────────────────────────────────────────────────────┘
```

---

## Tech Stack

| Component | Technology |
|---|---|
| UI Framework | SwiftUI |
| Platform | macOS first, iOS later |
| Audio playback | AVFoundation |
| Waveform rendering | Custom SwiftUI Canvas or Metal |
| Nexus connection | JoebotSDK NexusClient |
| File format | .jbt (daw_setlist, daw_project types) |
| MIDI | CoreMIDI |
| Theme | JoebotSDK theme system |

---

## Cue Lanes

### Lane Behavior

Each connected Nexus device gets its own cue lane. Lanes are dynamically generated from capability discovery — no hardcoded device list.

Each lane header shows:
- 🟢 / 🔴 / 🟡 — device status from Nexus
- Device label
- Offline behavior indicator

### Lane Status States

| Status | Color | Meaning |
|---|---|---|
| 🟢 Online | Full color | Device connected, cues will fire |
| 🔴 Offline | Grayed, dimmed cues | Device disconnected |
| 🟡 Connecting | Yellow | Reconnecting |

### Per-Lane Offline Behavior

Each lane has a configurable offline behavior:

| Mode | Behavior |
|---|---|
| Skip | Cue fires but is silently ignored if device offline |
| Queue | Cue is held for up to N seconds, fires when device reconnects |
| Warn | Cue fires, alert shown in performance dashboard |

Queue timeout options: 5s, 30s, Until reconnect

### Lane Colors

Each device type has a default color:
- MTPX — Cyan `#00FFFF`
- DirtyMixer — Orange `#FF6600`
- Atlas/Extron — Green `#00FF88`
- Text Wall — Purple `#AA00FF`
- DAW/GlitchBoard — Yellow `#FFCC00`
- Unknown — Grey `#888888`

Colors are customizable per lane per setlist.

---

## Cue Types

### One-Shot Cue

Fires a single action at a specific bar/beat position.

```
●  skew B=31
```

### Range / Ramp Cue

Fires a continuous action from one position to another. Used for parameter interpolation.

```
●──────────────●
ramp 0→255
```

### Muted Cue

Cue exists but won't fire. Shown dimmed with strikethrough.

```
⊘  skew B=31
```

---

## Edit Modes

| Mode | Icon | Behavior |
|---|---|---|
| Edit | ✏️ | Click to add cues, drag to move individual cues |
| Select | ↔️ | Click-drag to select time ranges across lanes |
| Razor | 🔪 | Click on range cue to split at that point |
| Cursor | 👆 | Move playhead only, no accidental edits |

---

## Snap to Grid

```
Snap: [ Off | 1/1 | 1/2 | 1/4 | 1/8 | 1/16 | 1/32 ]
```

When snap is enabled, cues magnetically lock to the nearest grid division when placed or dragged.

At 140 BPM:
- 1/4 note = 428ms
- 1/8 note = 214ms
- 1/16 note = 107ms
- 1/32 note = 54ms

---

## Multi-Select and Range Operations

### Selection

In Select mode, click-drag horizontally across the timeline. Selection spans all lanes simultaneously. The selected time range is highlighted across every lane.

Per-lane deselection: Cmd+click a lane header to exclude that lane from the selection.

### Selection Action Bar

When a selection is active the bottom bar shows:

```
Selected: 6 cues across 3 devices  Bar 2 Beat 1 → Bar 4
[ Delete ] [ Copy ] [ Move ] [ Mute ] [ Loop region ] [ Add Pattern... ]
```

### Add Pattern to Selection

Right-click selection or click Add Pattern button:

```
┌─ Add Pattern to Selection ──────────────────────────────┐
│                                                         │
│  Range: Bar 2 Beat 1  →  Bar 6 Beat 4                 │
│  Lanes: MTPX #1, Dirty Mixer                           │
│                                                         │
│  Pattern Type:                                          │
│  ○ Every beat                                          │
│  ○ Every X beats  [ 2 ] (1-16)                        │
│  ○ Every division [ 1/8 ▼ ]                           │
│  ○ On beats       [ ☑1 ☐2 ☑3 ☐4 ]                   │
│  ○ Custom pattern [ 1 0 1 0 1 1 0 1 ] (step seq)     │
│                                                         │
│  Starting on beat [ 1 ▼ ] of bar [ 2 ▼ ]             │
│                                                         │
│  Cue to stamp:                                         │
│  [ Max Blue Separation 🔵 ▼ ] (from cue library)      │
│  or [ + New Cue... ]                                   │
│                                                         │
│  Preview: ● · ● · ● · ● · ● · ● · ● · ● ·           │
│                                                         │
│  [ Cancel ]                    [ Add Pattern ]         │
└─────────────────────────────────────────────────────────┘
```

### Pattern Types

**Every beat** — fires on every beat in the selection range.

**Every X beats (1-16)** — fires every N beats. Creates polyrhythms against the bar:
- Every 2 → half time feel
- Every 3 → 3 against 4 polyrhythm, shifts every bar
- Every 5 → repeats every 5 bars against 4/4 grid
- Every 7 → near-random feeling, fully deterministic and reproducible

**Every division** — fires on musical subdivisions (1/8, 1/16 etc). High density patterns for rapid-fire effects.

**On beats** — checkboxes for specific beats in each bar. Check 1 and 3 for a classic feel. Check 2 and 4 for backbeat. Any combination works.

**Custom step sequencer** — a binary pattern string. `1 0 1 0 1 1 0 1` means fire on steps 1, 3, 5, 6, 8 and skip the rest. Repeats across the entire selection range.

**Live preview** updates in real time showing exactly where cues will land:
```
Preview: ● · ● · ● · ● · ● · ● · ● · ● ·
```

### Humanize Timing

Available on any selection. Shifts cues slightly off the perfect grid by a random amount within a tolerance:

```
Humanize: Amount [ ████░░░ ] 30ms max
```

Makes automated sequences feel less robotic without losing the musical relationship to the grid.

---

## Cue Preview Tooltip

Hover over any cue to see a tooltip showing:
- Device name and status
- Exact action name
- Parameters with values
- What SIS or board command will actually fire
- Bar/beat position and timestamp in seconds

Example:
```
┌─ Cue Info ──────────────────────────────┐
│  🟢 MTPX Plus #1                        │
│  set_input_skew                         │
│  Input: 3  R: 0  G: 0  B: 31 (MAX)    │
│                                         │
│  SIS: 3*0*0*31*4Iseq↵                  │
│  → mtpx1.extron.video:23               │
│                                         │
│  Bar 4 Beat 1  |  01:23.428            │
└─────────────────────────────────────────┘
```

The SIS command visibility is especially useful for learning and debugging.

---

## Cue Library

A panel on the left side of GlitchBoard showing saved reusable cues.

Drag any library cue onto a compatible lane — it snaps to the grid at the drop position.

Library cues have:
- Name
- Icon/emoji
- Action
- Compatible device types
- Default parameters
- Tags for filtering

### Example Library Cues

```
🔵 Max Blue Separation    mtpx    set_input_skew B=31
📼 VHS Overdrive          mtpx    set_input_peaking 200
⚡ Chaos Mode             dirty   recall_preset 12
🌑 Safe State             mtpx    reset_skew + reset_peaking
🎯 Blackout               atlas   set_video_mute all
```

Library is saved in the .jbt setlist file under `global_cue_library`.

---

## Setlist Mode

GlitchBoard supports multiple songs in a single .jbt setlist file.

### Setlist Panel

```
┌─ Setlist ───────────────────────────┐
│  Show A — March 2026               │
│                                     │
│  ▶ 01  Opening         03:24       │
│    02  Drop Section    04:12       │
│    03  Breakdown       02:55       │
│    04  Finale          05:30       │
│                                     │
│  [ + Add Song ]  [ Reorder ]       │
└─────────────────────────────────────┘
```

Click a song to switch to its cue timeline. The waveform and lanes update instantly.

### Song Transitions

Between songs a transition fires cleanup cues automatically:

```json
{
  "transition": {
    "type": "immediate",
    "duration_bars": 0,
    "transition_cues": [
      {
        "action": "reset_skew",
        "device": "device.mtpx.1",
        "note": "Zero all RGB skew between songs"
      },
      {
        "action": "recall_preset",
        "device": "device.dirtymixer.1",
        "params": {"preset_id": 1},
        "note": "Return to default mix state"
      }
    ]
  }
}
```

Transition types:
- **Immediate** — cleanup fires, next song starts instantly
- **Gap** — cleanup fires, N second pause, next song starts
- **Crossfade** — audio fades over N bars while cleanup fires

---

## Performance Dashboard

A separate fullscreen window for live performance monitoring. Put on a second monitor or iPad via Observatory.

```
┌─ LIVE ─────────────────────────────────┐
│                                        │
│  ▶ Bar 12  Beat 3  00:47.2            │
│  Song: Drop Section  (2 of 4)         │
│                                        │
│  NEXT CUE in 1.2s:                    │
│  🟢 MTPX — set_input_skew B=31        │
│                                        │
│  LAST FIRED:                           │
│  🟢 DirtyMixer — preset 12  (0.8s ago)│
│  🟢 Atlas — recall_preset 3  (2.1s)   │
│                                        │
│  DEVICE STATUS:                        │
│  🟢 MTPX #1   🟢 Dirty   🔴 Atlas    │
│                                        │
│  NEXT SONG: Breakdown  (in 2:37)      │
│                                        │
└────────────────────────────────────────┘
```

Shows:
- Current bar, beat, timestamp
- Current song and position in setlist
- Next cue with countdown
- Last 2-3 fired cues with elapsed time
- Device health status
- Next song name and time remaining

---

## MIDI Controller Integration

GlitchBoard connects to MIDI devices via CoreMIDI. Tested with APC Mini Mk2.

### MIDI Mapping

Each button/fader on the APC Mini can be mapped to a Nexus action. The mapping UI uses capability discovery — available actions come from Nexus, not hardcoded lists.

```
┌─ MIDI Mapping ──────────────────────────────────────────┐
│  Device: APC Mini Mk2                                   │
│                                                         │
│  Button 1,1  →  [ recall_preset 12 on dirtymixer  ]   │
│  Button 1,2  →  [ set_input_skew B=31 on mtpx.1   ]   │
│  Button 1,3  →  [ reset_skew on mtpx.1             ]   │
│  Fader 1     →  [ set_channel_mix CH1 on dirtymixer]   │
│  Fader 2     →  [ set_input_peaking IN3 on mtpx.1  ]   │
│                                                         │
│  [ Learn Mode ]  — press button to assign             │
└─────────────────────────────────────────────────────────┘
```

**Learn mode** — click Learn, press a physical button/fader, GlitchBoard detects the MIDI message and assigns it.

Fader mapping includes range scaling — a 0-127 MIDI fader can map to any parameter range (0-255 for mix, 0-31 for skew, etc).

MIDI mappings are saved per setlist in the .jbt file.

---

## Undo / Redo

Full undo/redo stack. Cmd+Z / Cmd+Shift+Z.

Every operation that modifies cues is undoable:
- Add cue
- Delete cue(s)
- Move cue
- Copy/paste cues
- Add pattern
- Humanize
- Change cue parameters
- Mute/unmute

Undo history is per-song within the setlist. Switching songs preserves each song's undo stack independently.

---

## Zoom and Navigation

Waveform and cue lanes zoom and scroll together — always in sync.

**Zoom controls:**
- `[–]` / `[+]` buttons
- Pinch gesture on trackpad
- Scroll wheel with Cmd held
- `[Fit]` — zoom to show entire song

**Scroll:**
- Scroll wheel horizontal
- Two-finger swipe on trackpad

**Playback follow mode:**
- During playback the timeline scrolls to keep the playhead centered
- Toggle with a button — sometimes you want to scroll ahead while playing

**Zoom levels:**
- Zoomed out: see full song, bars visible
- Medium: see 32 bars, individual beats visible
- Zoomed in: see 4-8 bars, 1/16th note divisions visible
- Maximum zoom: individual samples visible (for precise placement)

---

## Capability Discovery Flow

On launch GlitchBoard queries Nexus for all connected devices and their capabilities:

1. GlitchBoard registers with Nexus as `glitchboard_v1`
2. Sends `capabilities.query` for all connected clients
3. Nexus returns capabilities for each device
4. GlitchBoard builds lane list from response
5. Cue editor action dropdowns populate from capability data
6. MIDI mapping action lists populate from capability data

When a new device connects mid-session a new lane appears automatically. When a device disconnects its lane grays out but cues are preserved.

---

## Timing Architecture

### Pre-scheduling

GlitchBoard uses look-ahead scheduling to avoid timing drift:

- Every 50ms the scheduler looks 200ms ahead
- Upcoming cues are pre-scheduled using AVAudioTime for precise delivery
- Cues are never fired reactively (which introduces latency)
- Result: sub-10ms timing accuracy

### BPM and Time Signature

V1 assumes constant BPM and 4/4 time signature throughout a song.

Future: variable tempo map, time signature changes.

### Beat Detection

V1 uses manual cue placement only. No automatic beat detection.

Future: optional automatic beat detection using Accelerate framework for audio analysis.

---

## .jbt File Format

### daw_setlist

```json
{
  "jbt_type": "daw_setlist",
  "version": "1.0",
  "created_at": "2026-03-14T00:00:00Z",
  "name": "Show A — March 2026",
  "payload": {
    "songs": [
      {
        "id": "song_001",
        "title": "Opening",
        "audio_path": "~/Music/opening.wav",
        "bpm": 140,
        "time_signature": "4/4",
        "cues": [ ... ],
        "transition": {
          "type": "immediate",
          "transition_cues": [ ... ]
        }
      }
    ],
    "global_cue_library": [
      {
        "id": "cue_lib_001",
        "name": "Max Blue Separation",
        "icon": "🔵",
        "action": "set_input_skew",
        "device_type": "mtpx",
        "params": {"red": 0, "green": 0, "blue": 31},
        "tags": ["signature", "glitch", "color"]
      }
    ],
    "device_lanes": [
      {
        "device_id": "device.mtpx.1",
        "label": "MTPX Plus #1",
        "color": "#00FFFF",
        "offline_behavior": "skip",
        "queue_timeout_seconds": 5
      },
      {
        "device_id": "device.dirtymixer.1",
        "label": "Dirty Mixer",
        "color": "#FF6600",
        "offline_behavior": "queue",
        "queue_timeout_seconds": 30
      }
    ],
    "midi_mappings": [
      {
        "midi_device": "APC Mini Mk2",
        "button": {"row": 1, "col": 1},
        "action": "recall_dirtymixer_preset",
        "params": {"preset_id": 12}
      }
    ]
  }
}
```

### Cue Object

```json
{
  "id": "cue_001",
  "type": "one_shot",
  "bar": 4,
  "beat": 1,
  "device_id": "device.mtpx.1",
  "action": "set_input_skew",
  "params": {"input": 3, "red": 0, "green": 0, "blue": 31},
  "muted": false,
  "label": "Max separation",
  "color": null
}
```

Range cue adds:
```json
{
  "type": "range",
  "end_bar": 8,
  "end_beat": 1,
  "interpolation": "linear",
  "start_params": {"mix": 0},
  "end_params": {"mix": 255}
}
```

---

## Build Priority

### Phase 1 — Basic Playback and Cues
1. SwiftUI shell with NexusStatusIndicator
2. Audio file loading and waveform display via AVFoundation
3. Bar/beat ruler synced to waveform
4. Single cue lane with manual cue placement
5. Snap to grid
6. Cue fires Nexus intent on playhead hit
7. Basic .jbt save/load

### Phase 2 — Full Lane System
8. Capability discovery → dynamic lane generation
9. Per-lane status dots from Nexus
10. Multiple lanes, per-lane colors
11. Range/ramp cues with interpolation
12. Cue preview tooltip
13. Undo/redo

### Phase 3 — Selection and Patterns
14. Select mode with range selection
15. Delete/copy/move/mute selection
16. Add pattern dialog
17. All pattern types including step sequencer
18. Humanize timing

### Phase 4 — Setlist and Performance
19. Setlist mode — multiple songs
20. Song transitions with cleanup cues
21. Cue library panel
22. Performance dashboard window
23. MIDI controller integration

### Phase 5 — Polish
24. All themes from JoebotSDK
25. Variable BPM/tempo map
26. Auto beat detection option
27. iOS companion

---

## First Session Prompt for Claude Code

> "I am building GlitchBoard, a native SwiftUI macOS DAW-style show control app that is part of the Joebot Ecosystem. It loads audio files, displays a waveform with a bar/beat grid, and lets you place cues on per-device lanes that fire through a WebSocket server called Nexus. Start by building Phase 1: the basic app shell with audio loading via AVFoundation, waveform display using SwiftUI Canvas, a bar/beat ruler that syncs with the waveform, and a single hardcoded cue lane where you can click to place one-shot cues that snap to a 1/4 note grid. Use the Joebot Classic dark theme — dark grey background, orange accents. Include NexusStatusIndicator from JoebotSDK in the toolbar. The full spec is at https://raw.githubusercontent.com/joebot94/docs/main/GlitchBoard_Spec.md"

---

## Related Documents

- `Nexus_Architecture.md` — Nexus server spec
- `JoebotSDK_Guide.md` — shared Swift toolkit
- `JBT_Format_Spec.md` — .jbt file format
- `Extron_SIS_Reference.md` — device command reference

---

*GlitchBoard — DAW-style show control for the Joebot Ecosystem*
*github.com/joebot94/daw-app*
*Document version 1.0 — March 2026*
*🦖*
