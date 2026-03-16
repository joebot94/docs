# TextWall — Build Specification
> Joebot Ecosystem Text and Lyric Display App
> GitHub: github.com/joebot94/textglitch (rename to textwall)
> Document version 1.0 — March 2026
> 🦖 Joebot Ecosystem

---

## What TextWall Is

TextWall is the native SwiftUI macOS app that receives text content from Nexus and displays it across a configurable grid of cells. It is the visual voice of the performance — lyrics appear word by word, letters scatter across 256 squares, phrases pulse in and out on the beat.

It is not a dumb display. TextWall is a smart renderer. The Lyric App sends raw text. TextWall decides how to show it based on its current mode, layout, and configuration. Two performances of the same song can look completely different depending on how TextWall is configured.

TextWall output connects to the signal chain via scan converter — the app window is captured by a VSC unit, fed into the Matrix 12800, and routed wherever it needs to go. One app, one window, infinite routing possibilities.

---

## Core Philosophy

- **Text is a visual instrument** — not just information, a performance element
- **Smart renderer** — receives dumb text, applies intelligent display logic
- **Grid-native** — everything is cells, from 1x1 to 16x16
- **Beat-aware** — high frequency updates must be smooth, no lag at 20Hz
- **Nexus-driven** — all content arrives via Nexus, nothing is typed live
- **Graceful** — works perfectly with no Nexus connection, shows last known state

---

## Layout — Main Window

```
┌─ TextWall ──────────────────────────────────────────────────┐
│  TEXTWALL                            🟢 Nexus  ⚙️  [config] │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                                                     │   │
│  │  ┌───────┐  ┌───────┐  ┌───────┐                  │   │
│  │  │       │  │       │  │       │                  │   │
│  │  │  WEL  │  │  TO   │  │  THE  │                  │   │
│  │  │  COME │  │       │  │       │                  │   │
│  │  └───────┘  └───────┘  └───────┘                  │   │
│  │  ┌───────┐  ┌───────┐  ┌───────┐                  │   │
│  │  │       │  │       │  │       │                  │   │
│  │  │       │  │ MACH  │  │       │                  │   │
│  │  │       │  │  INE  │  │       │                  │   │
│  │  └───────┘  └───────┘  └───────┘                  │   │
│  │  ┌───────┐  ┌───────┐  ┌───────┐                  │   │
│  │  │       │  │       │  │       │                  │   │
│  │  │       │  │       │  │       │                  │   │
│  │  │       │  │       │  │       │                  │   │
│  │  └───────┘  └───────┘  └───────┘                  │   │
│  │                                                     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  Grid: [3×3 ▼]  Layout: [X Pattern ▼]  Mode: [Word ▼]     │
│  Current: "welcome to the machine"  Style: [Verse ▼]       │
│  Last update: 0.3s ago                                      │
└─────────────────────────────────────────────────────────────┘
```

---

## Configuration Panel

```
┌─ TextWall Config ───────────────────────────────────────────┐
│                                                             │
│  GRID                                                       │
│  Size: [ 1×1 | 2×2 | 3×3 | 4×4 | 8×8 | 16×16 | Custom ]  │
│  Custom: [ 5 ] cols  [ 3 ] rows                            │
│                                                             │
│  LAYOUT — which cells are active                           │
│  ○ All cells                                               │
│  ○ X pattern          ○ Center only                       │
│  ○ Top row            ○ Bottom row                        │
│  ○ Center row         ○ Corners only                      │
│  ○ Border cells       ○ Checkerboard A                    │
│  ● Custom             ○ Checkerboard B                    │
│                                                             │
│  Custom layout picker:                                      │
│  ┌───┬───┬───┐                                             │
│  │ ☑ │ ☐ │ ☑ │  ← click cells to toggle active/inactive  │
│  ├───┼───┼───┤                                             │
│  │ ☐ │ ☑ │ ☐ │                                             │
│  ├───┼───┼───┤                                             │
│  │ ☑ │ ☐ │ ☑ │                                             │
│  └───┴───┴───┘                                             │
│  [ Save Layout... ]  Name: [ X Pattern          ]          │
│                                                             │
│  DISPLAY MODE                                               │
│  ○ Line mode    — show full text in active cells           │
│  ○ Word mode    — one word per active cell                 │
│  ○ Letter mode  — one character per active cell            │
│  ● Scatter mode — N instances across random cells          │
│  ○ Reveal mode  — words appear over time (beat-synced)    │
│                                                             │
│  Scatter settings (when scatter mode active):              │
│  Instances per word: [ 5 ]  (1–32)                        │
│  Rotation speed: [ 10 ] Hz  (1–20)                        │
│  Words shown simultaneously: [ 4 ]                        │
│                                                             │
│  APPEARANCE                                                 │
│  Font: [ Helvetica Neue ▼ ]  Weight: [ Bold ▼ ]           │
│  Size: [ Auto-fit ▼ ]  (fits text to cell)                │
│  Color: [ ■ White  ]  Background: [ ■ Black ]              │
│  Padding: [ 8px ]                                          │
│                                                             │
│  TRANSITION                                                 │
│  Between updates: [ Cut | Fade ▼ ]  Duration: [ 200ms ]   │
│                                                             │
│  [ Cancel ]                         [ Apply ]              │
└─────────────────────────────────────────────────────────────┘
```

---

## Display Modes

### Line Mode
The full text string displays in every active cell simultaneously.
Good for titles, important single phrases, simple lyric display.

```
┌───────────┐  ┌───────────┐  ┌───────────┐
│           │  │           │  │           │
│  welcome  │  │  welcome  │  │  welcome  │
│  to the   │  │  to the   │  │  to the   │
│  machine  │  │  machine  │  │  machine  │
│           │  │           │  │           │
└───────────┘  └───────────┘  └───────────┘
```

---

### Word Mode
Text splits into words. One word per active cell in order.
Surplus cells are blank. Surplus words wrap or are truncated.

```
Text: "welcome to the machine"
Active cells: 4 (corners of 3×3)

┌───────┐  ┌───────┐  ┌───────┐
│  WEL  │  │       │  │  TO   │
│  COME │  │       │  │       │
└───────┘  └───────┘  └───────┘
┌───────┐  ┌───────┐  ┌───────┐
│       │  │       │  │       │
│       │  │       │  │       │
└───────┘  └───────┘  └───────┘
┌───────┐  ┌───────┐  ┌───────┐
│  THE  │  │       │  │ MACH  │
│       │  │       │  │  INE  │
└───────┘  └───────┘  └───────┘
```

---

### Letter Mode
Text splits into individual characters. One character per active cell.
Punctuation included or stripped (configurable).

```
Text: "MACHINE"
Active cells: 7+ (center row of 16×16 for example)

┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐
│M │ │A │ │C │ │H │ │I │ │N │ │E │
└──┘ └──┘ └──┘ └──┘ └──┘ └──┘ └──┘
```

---

### Scatter Mode ⭐
The signature TextWall mode. Takes one or more words, places N instances
of each across randomly selected cells, rotates positions at a set Hz.

At 10Hz the words appear to swarm across the wall. At 20Hz they become
visual texture — still readable as a word shape but no longer readable
as language. The word becomes a pattern.

```
Text: "welcome to the machine"
Instances per word: 8  |  Grid: 16×16  |  Speed: 10Hz
Active: all 256 cells

Every 100ms, 32 cells chosen randomly display the four words:
8× WELCOME — 8× TO — 8× THE — 8× MACHINE
224 cells blank

Frame 1:                    Frame 2 (100ms later):
·  ·  W  ·  ·  T  ·  ·    ·  M  ·  ·  W  ·  ·  ·
·  M  ·  ·  W  ·  ·  T    ·  ·  T  ·  ·  M  ·  W
T  ·  ·  M  ·  ·  W  ·    W  ·  ·  ·  T  ·  ·  M
·  ·  T  ·  M  ·  ·  W    ·  ·  M  W  ·  ·  T  ·
(etc across 16 rows)       (new random positions)
```

**Closing shot mode** — maximum instances, maximum speed:
```
Text: "welcome to the machine"
Instances per word: 8 each = 32 total lit cells
Grid: 16×16 = 256 cells
Speed: 10Hz
Duration: until lyric_clear

The entire wall consumed by the phrase. Swarming. Alive.
```

---

### Reveal Mode
Words appear one at a time over a set duration. Beat-synced or time-based.
Each word fades or cuts in at its scheduled moment.

```
Text: "welcome to the machine"
Duration: 4 seconds  |  Direction: left→right then top→bottom
Active: 2×2 corners

t=0.0s:   t=1.0s:   t=2.0s:   t=3.0s:
┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐
│WE│ │  │ │WE│ │TO│ │WE│ │TO│ │WE│ │TO│
└──┘ └──┘ └──┘ └──┘ └──┘ └──┘ └──┘ └──┘
┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐
│  │ │  │ │  │ │  │ │TH│ │  │ │TH│ │MA│
└──┘ └──┘ └──┘ └──┘ └──┘ └──┘ └──┘ └──┘
```

Gilmour drag effect: set duration to match how long he holds the syllable.
The word populates as he sings it.

---

### Checkerboard Scatter Mode
Two word sets alternate between checkerboard cell groups A and B on each beat.
Creates a strobe effect where the text itself pulses.

```
Beat 1 — Group A cells lit:    Beat 2 — Group B cells lit:
┌──┬──┬──┐                     ┌──┬──┬──┐
│WE│  │TO│                     │  │MA│  │
├──┼──┼──┤                     ├──┼──┼──┤
│  │TH│  │                     │WE│  │TO│
├──┼──┼──┤                     ├──┼──┼──┤
│MA│  │IN│                     │  │CH│  │
└──┴──┴──┘                     └──┴──┴──┘
```

---

## Nexus Message Types

### Incoming (Nexus → TextWall)

#### lyric_update
Primary content update. Arrives from Lyric App via Nexus.

```json
{
  "type": "lyric_update",
  "source": "lyric_app",
  "payload": {
    "text": "welcome to the machine",
    "style": "verse",
    "bar": 4,
    "beat": 1,
    "timestamp": 62.5,
    "index": 8,
    "total": 24
  }
}
```

TextWall renders this according to its current mode and layout.

#### lyric_clear
Clears all cell content. Blank wall.

```json
{
  "type": "lyric_clear",
  "source": "lyric_app",
  "payload": {}
}
```

#### textwall_config
Sent by GlitchBoard or any controller to change TextWall's mode mid-performance.
This is how GlitchBoard switches from verse mode to scatter mode on the closing line.

```json
{
  "type": "intent",
  "targets": ["textwall_v1"],
  "payload": {
    "action": "set_config",
    "params": {
      "grid_size": "4x4",
      "layout": "all_cells",
      "mode": "scatter",
      "scatter_instances": 8,
      "scatter_hz": 10
    }
  }
}
```

#### textwall_set_cell
Direct cell control — set specific cell content without changing mode.
Used by GlitchBoard for precise cue placement.

```json
{
  "type": "intent",
  "targets": ["textwall_v1"],
  "payload": {
    "action": "set_cell",
    "params": {
      "row": 1,
      "col": 1,
      "text": "WELCOME",
      "style": "chorus"
    }
  }
}
```

#### textwall_recall_layout
Recall a saved named layout configuration.

```json
{
  "type": "intent",
  "targets": ["textwall_v1"],
  "payload": {
    "action": "recall_layout",
    "params": {
      "layout_name": "verse_center_row"
    }
  }
}
```

---

### Outgoing (TextWall → Nexus)

#### Registration

```json
{
  "type": "register",
  "payload": {
    "client_id": "textwall_v1",
    "client_type": "display",
    "version": "1.0.0",
    "capabilities": {
      "grid_sizes": ["1x1","2x2","3x3","4x4","8x8","16x16","custom"],
      "modes": ["line","word","letter","scatter","reveal","checkerboard_scatter"],
      "max_hz": 20,
      "max_instances": 32,
      "layouts": ["all","x_pattern","center","corners","top_row","bottom_row",
                  "center_row","border","checkerboard_a","checkerboard_b","custom"],
      "saved_layouts": ["verse_center_row","chorus_corners","scatter_full"]
    }
  }
}
```

GlitchBoard reads this and builds its TextWall lane cue editor dynamically.
No hardcoding anywhere.

#### State Update
Sent whenever config changes so Observatory and GlitchBoard stay in sync.

```json
{
  "type": "state_update",
  "payload": {
    "client_id": "textwall_v1",
    "state": {
      "grid_size": "3x3",
      "mode": "word",
      "layout": "x_pattern",
      "current_text": "welcome to the machine",
      "scatter_hz": 0,
      "scatter_instances": 0
    }
  }
}
```

---

## Saved Layout .jbt Format

```json
{
  "jbt_type": "textwall_layout",
  "version": "1.0",
  "created_at": "2026-03-15T00:00:00Z",
  "name": "Welcome to the Machine — Closing Shot",
  "payload": {
    "grid_size": "16x16",
    "active_cells": "all",
    "mode": "scatter",
    "scatter_instances": 8,
    "scatter_hz": 10,
    "font": "Helvetica Neue",
    "weight": "bold",
    "color": "#FFFFFF",
    "background": "#000000"
  }
}
```

Stored at: `~/JBT/textwall/layouts/`

---

## Welcome to the Machine — Performance Map

This is the reference example. Full song broken into TextWall configurations.

```
TIME      LYRIC                        MODE         LAYOUT          NOTES
────────────────────────────────────────────────────────────────────────────
1:02.5    welcome, my son              line         center row      soft open
1:05.2    welcome                      word         center only     one word
1:06.8    to the                       word         center row      two cells
1:07.8    [clear]                      —            —               beat pause
1:08.1    machine.....                 letter       center row      M-A-C-H-I-N-E
1:14.0    [clear]                      —            —               
1:18.7    where have you been?         line         center          question
1:22.1    it's alright                 word         top row         
1:24.0    we know                      word         bottom row      
1:25.5    where you've been            word         all corners     
────────────────────────────────────────────────────────────────────────────
[machine noise section — the stereo whoosh]
────────────────────────────────────────────────────────────────────────────
2:10.0    [scatter begins — low speed] scatter      all 3x3         hz=2
2:14.0    [ramp up scatter speed]      scatter      all 3x3         hz=5
2:18.0    [full scatter]               scatter      all 3x3         hz=10
────────────────────────────────────────────────────────────────────────────
[final verse builds...]
────────────────────────────────────────────────────────────────────────────
FINAL     welcome to the machine       scatter      16x16 all       hz=10
          ← 4 words × 8 instances = 32 lit cells across 256         instances=8
          ← positions randomize every 100ms
          ← wall consumed by the phrase
          ← hold until end of song
          ← lyric_clear on silence
```

---

## Cell Architecture

### Why 16×16 at 20Hz is doable

256 cells each containing a short string. SwiftUI LazyVGrid with fixed cell size.
On each scatter tick only the cells that changed need to redraw — not all 256.

Key architecture decision: **cell state is a flat array, not a nested grid.**

```swift
// Fast — index math is trivial
@Published var cells: [CellState] = Array(
    repeating: CellState(), 
    count: gridSize.rows * gridSize.cols
)

// Cell index from row/col
func index(row: Int, col: Int) -> Int {
    return row * gridSize.cols + col
}

// Scatter tick — only update changed indices
func scatterTick() {
    let newPositions = selectRandomCells(count: totalInstances)
    var newCells = blankCells()
    for (word, positions) in wordPositions(newPositions) {
        for pos in positions {
            newCells[pos].text = word
        }
    }
    cells = newCells  // single @Published update → one redraw
}
```

Timer fires at scatter Hz. One array replacement per tick. SwiftUI diffs and redraws only changed cells. At 20Hz this is 20 array replacements per second — trivial.

### Cell State

```swift
struct CellState: Identifiable, Equatable {
    let id: Int
    var text: String = ""
    var style: TextStyle = .default
    var isActive: Bool = true
}
```

---

## Observatory Card

```
┌─────────────────────────────────┐
│ 🟢  TextWall                    │
│                                 │
│ Grid: 3×3  Mode: scatter        │
│ Speed: 10Hz  Instances: 8       │
│                                 │
│ "welcome to the machine"        │
│                                 │
│ [ Open TextWall ]               │
└─────────────────────────────────┘
```

---

## Repo Structure

```
textglitch/  (rename to textwall eventually)
├── TextWallApp.swift
├── ContentView.swift
├── Views/
│   ├── CellGridView.swift         ← the main display grid
│   ├── CellView.swift             ← individual cell
│   ├── ConfigPanel.swift          ← settings sheet
│   └── LayoutPicker.swift         ← visual cell toggle grid
├── Models/
│   ├── CellState.swift
│   ├── GridConfig.swift
│   ├── DisplayMode.swift
│   └── SavedLayout.swift
├── Engine/
│   ├── ScatterEngine.swift        ← handles scatter/strobe tick
│   ├── RevealEngine.swift         ← handles timed word reveal
│   └── TextSplitter.swift         ← word/letter splitting logic
└── Services/
    └── NexusHandler.swift         ← all incoming message handling
```

---

## Build Priority

### Phase 1 — Basic Display and Nexus
1. SwiftUI shell with NexusStatusIndicator — Joebot Classic theme
2. 3×3 cell grid — hardcoded size to start
3. Connect to Nexus as `textwall_v1`
4. Receive `lyric_update` — display text in all cells (line mode)
5. Receive `lyric_clear` — blank all cells
6. Register capabilities with Nexus
7. Shows in Observatory ✅

### Phase 2 — Grid Sizes and Layouts
8. Grid size selector — 1×1 through 16×16
9. Layout system — active/inactive cells
10. Built-in layouts — X pattern, corners, rows, border
11. Custom layout picker — click cells to toggle
12. Save/load layouts as .jbt

### Phase 3 — Display Modes
13. Word mode — split text across active cells
14. Letter mode — split characters across cells
15. Reveal mode — words appear over time
16. Config panel — full settings UI

### Phase 4 — Scatter Mode ⭐
17. ScatterEngine — random position selection
18. Timer-driven scatter tick at configurable Hz
19. Instance count control
20. Scatter ramp — speed changes via `textwall_config` intent
21. Multi-word scatter — all four words simultaneously

### Phase 5 — GlitchBoard Integration
22. Receive `set_config` intents from GlitchBoard
23. Receive `set_cell` direct cell control
24. Receive `recall_layout` named layout switching
25. State updates to Nexus on every config change

### Phase 6 — Polish
26. Font and color customization
27. Transition effects between updates (cut, fade, flash)
28. iOS companion — same display on iPad/iPhone
29. Checkerboard scatter mode

---

## Lyric Player Script (Deadline Hack)

For the Welcome to the Machine prompt deadline — a Python script that
fires pre-timed lyrics through Nexus while audio plays separately.
No waveform editor needed. Gets TextWall displaying lyrics tonight.

```
nexus/tools/lyric_player.py
```

Lyric file format (simple text, easy to edit):

```
# welcome_to_the_machine.lyrics
# BPM: ~72  Key: E minor  Vibe: dystopian

1:02.5  welcome, my son
1:05.2  welcome
1:06.8  to the
1:07.8  [clear]
1:08.1  machine.....
1:14.0  [clear]
1:18.7  where have you been?
1:22.1  it's alright
1:24.0  we know
1:25.5  where you've been
```

Script behavior:
- Reads the file
- Plays audio via system (or just counts time if no audio arg)
- Fires `lyric_update` to Nexus at each timestamp
- Fires `lyric_clear` on [clear] lines
- Ctrl+C to stop, fires `lyric_clear` on exit

```bash
python lyric_player.py welcome_to_the_machine.lyrics --audio ~/Music/welcome.wav
```

This is the bridge until the proper Lyric App is built.

---

## First Session Prompt for Claude Code

> "I am building TextWall, a native SwiftUI macOS display app that is part of the Joebot Ecosystem. It connects to a WebSocket server called Nexus and receives lyric_update messages that it displays across a configurable grid of cells. Start with Phase 1: build a 3×3 cell grid where each cell displays text, connect to Nexus via JoebotSDK NexusClient, register as textwall_v1 with capabilities, receive lyric_update messages and display the text in all cells (line mode), and receive lyric_clear to blank the grid. Use the Joebot Classic dark theme — full black background, white text in cells, orange accent for the toolbar. The cell grid should fill the entire window. NexusStatusIndicator in the toolbar. Read the full spec at https://raw.githubusercontent.com/joebot94/docs/main/TextWall_BuildGuide.md and the architecture at https://raw.githubusercontent.com/joebot94/docs/main/Nexus_Architecture.md"

Also build this Python script in the same session:
> "Also build nexus/tools/lyric_player.py — a script that reads a timestamped lyrics text file and fires lyric_update WebSocket messages to Nexus at the correct times. Format is MM:SS.s followed by the lyric text, or [clear] to send lyric_clear. Takes the lyrics file as argument and an optional --audio flag. Print each lyric to console as it fires."

---

## Related Documents

- `Nexus_Architecture.md` — Nexus server spec
- `JBT_Format_Spec.md` — .jbt file format
- `LyricApp_BuildGuide.md` — Lyric App spec
- `GlitchBoard_Spec.md` — GlitchBoard DAW spec
- `JoebotSDK_Guide.md` — shared Swift toolkit

---

*TextWall — Text and lyric display for the Joebot Ecosystem*
*github.com/joebot94/textglitch*
*Document version 1.0 — March 2026*
*🦖*
