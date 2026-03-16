# GlitchBoard — Build Specification
> Joebot Ecosystem DAW-Style Show Control App
> GitHub: github.com/joebot94/daw-app
> Document version 1.3 — March 2026
> Changes: Added TextWall lane as first-class lane type, inline TextWall cue editor
> with mini preview popup, hover tooltips with SIS command preview, snake keyframe
> path editor, freeform cell picker for sequence/checkerboard cues, lyric_timeline.jbt
> import → auto-populate TextWall lane, duplicate cue smart stamp tool, color
> interpolation modes, operating mode awareness.
> 🦖 Joebot Ecosystem

---

## What GlitchBoard Is

GlitchBoard is a native SwiftUI macOS DAW-style show control application. It loads audio,
displays a waveform with a bar/beat grid, and lets you place cues on per-device lanes that
fire through Nexus to control hardware in sync with music.

It is the bridge between music and hardware. Load a song, place cues, hit play, and your
MTPX skew changes, dirty mixer presets recall, TextWall displays lyrics, and Atlas fires
routing commands — all synchronized to the beat.

---

## Core Philosophy

- **Reproducible** — every performance saved as .jbt setlist and replayed identically
- **Graceful** — offline devices don't kill the show, they skip or queue
- **Discoverable** — lanes and actions come from Nexus capability discovery, nothing hardcoded
- **Musical** — snap to grid, polyrhythms, step sequencer patterns, humanize
- **Live-friendly** — performance dashboard, APC Mini MIDI control, setlist mode
- **Visual** — tooltips show exactly what SIS command will fire before it happens

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
│  │  ●           ●──────────────●    ●                  │  │
│  │  skew B=31   ramp 0→255          preset 5           │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌─ 🟢 Dirty Mixer ────────────────────────────────────┐  │
│  │       ●──────────────●   ●                          │  │
│  │       ramp 0→255     ↑   preset 12                  │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌─ 🟢 TextWall ───────────────────────────────────────┐  │
│  │  ◉           ◉              ◉⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳       │  │
│  │  [3×3 ctr]   [2×2 corners]  [16×16 scatter 10Hz]   │  │
│  │  "welcome    "welcome to    "welcome to the         │  │
│  │   my son"     the machine"   machine" ×8inst        │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌─ 🔴 Atlas ──────────────────────────────────────────┐  │
│  │  [offline — cues will skip]                         │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                            │
│  ████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   │
│                                                            │
├────────────────────────────────────────────────────────────┤
│  Snap: [ 1/4 ▼ ]  Zoom: [–][+][Fit]  Theme: [Joebot ▼]  │
└────────────────────────────────────────────────────────────┘
```

---

## TextWall Lane

TextWall is a first-class lane type in GlitchBoard — not an afterthought.
Each TextWall cue carries both text content AND complete display configuration.

### TextWall Cue Dot Display

```
◉           ← filled circle, purple accent color
[3×3 ctr]   ← layout summary
"welcome    ← first 20 chars of text
 my son"
```

### TextWall Lane — Scatter Cue Visual

```
◉⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳
[16×16 scatter 10Hz]
"welcome to the machine" ×8inst
```

The ⟳ symbols extend to show the duration of the scatter cue.

### Import lyric_timeline.jbt

Drag a `lyric_timeline.jbt` file into GlitchBoard or use File → Import Lyrics.
Every lyric cue becomes a TextWall lane cue automatically.
All TextWall config from the Lyric App is preserved.
Every cue is fully editable after import.

```
Import:    lyric_timeline.jbt
           ↓
GlitchBoard creates TextWall lane
           ↓
Each lyric → one TextWall cue
Text + timing + grid + layout + mode all imported
           ↓
You can edit any cue — import is a starting point not a lock
```

---

## Cue Editor — Side Panel

When you click a cue the right side panel shows the editor.
The editor content changes based on cue type and device.

### MTPX Cue Editor

```
┌─ Cue Editor — MTPX Plus #1 ────────────────────────────┐
│  Bar 4  Beat 1  |  01:23.428                           │
│  Device: 🟢 MTPX Plus #1                               │
│                                                         │
│  Action: [ set_input_skew ▼ ]                          │
│                                                         │
│  Input:  [ 3  ] (1–16)                                 │
│  Red:    [ 0  ] (0–31)  ░░░░░░░░░░░░░░░░░░░░░░░░      │
│  Green:  [ 0  ] (0–31)  ░░░░░░░░░░░░░░░░░░░░░░░░      │
│  Blue:   [ 31 ] (0–31)  ████████████████████████       │
│                                                         │
│  SIS preview:                                           │
│  3*0*0*31*4Iseq↵                                       │
│  → mtpx1.extron.video:23                               │
│                                                         │
│  Offline behavior: [ Skip ▼ ]                          │
│                                                         │
│  [ 🗑 Delete ]  [ ⎘ Duplicate... ]  [ ✓ Save ]         │
└─────────────────────────────────────────────────────────┘
```

### TextWall Cue Editor

```
┌─ Cue Editor — TextWall ────────────────────────────────┐
│  Bar 8  Beat 1  |  02:17.143                           │
│  Device: 🟢 TextWall                                   │
│                                                         │
│  Text:                                                  │
│  ┌─────────────────────────────────────────────────┐   │
│  │ welcome to the machine                          │   │
│  └─────────────────────────────────────────────────┘   │
│  Style: [ Verse ▼ ]   Hardware advance: [ ☑ Relay 2 ] │
│                                                         │
│  DISPLAY CONFIGURATION                                  │
│  Grid:   [ 2×2 ▼ ]                                     │
│  Layout: [ All Cells ▼ ]                               │
│  Mode:   [ Word ▼ ]                                    │
│  Font:   [ Helvetica Neue ▼ ]  Weight: [ Bold ▼ ]     │
│  Color:  [ ■ White  ]  BG: [ ■ Black ]                │
│                                                         │
│  PREVIEW                         [ ▶ Animate ]         │
│  ┌──────────┬──────────┐                               │
│  │          │          │                               │
│  │ WELCOME  │  TO      │                               │
│  │          │          │                               │
│  ├──────────┼──────────┤                               │
│  │          │          │                               │
│  │  THE     │ MACHINE  │                               │
│  │          │          │                               │
│  └──────────┴──────────┘                               │
│  Preview updates live as you change settings           │
│                                                         │
│  Duration: [ Until next cue ▼ ]                       │
│  Offline behavior: [ Skip ▼ ]                         │
│                                                         │
│  [ 🗑 Delete ]  [ ⎘ Duplicate... ]  [ ✓ Save ]         │
└─────────────────────────────────────────────────────────┘
```

### TextWall Scatter Editor (mode = scatter)

When scatter mode is selected additional controls appear below the preview:

```
│  Mode:   [ Scatter ▼ ]                                  │
│                                                         │
│  ┌─ Scatter Settings ─────────────────────────────┐    │
│  │  Instances per word: [ 8  ] (1–32)             │    │
│  │  Speed:              [ 10 ] Hz (1–20)           │    │
│  │  Words simultaneously: [ 4 ]                   │    │
│  │  ⚠️ 15Hz+ may affect photosensitive users      │    │
│  └────────────────────────────────────────────────┘    │
│                                                         │
│  PREVIEW  [ ▶ Animate ]                                │
│  ┌──────────────────────────────┐                      │
│  │  · · W · · T · ·            │                      │
│  │  M · · · W · · T            │                      │
│  │  · · T · · M · ·            │                      │
│  └──────────────────────────────┘                      │
│  Animates at configured Hz — shows real scatter motion │
```

### TextWall Reveal Editor (mode = reveal)

```
│  Mode:   [ Reveal ▼ ]                                   │
│                                                         │
│  ┌─ Reveal Settings ──────────────────────────────┐    │
│  │  Direction: [ Left → Right ▼ ]                 │    │
│  │  Duration:  [ 4.0s ] (matches cue length)      │    │
│  │  Timing:    [ Even spread ▼ ]                  │    │
│  └────────────────────────────────────────────────┘    │
│                                                         │
│  PREVIEW + SCRUBBER                                    │
│  ┌──┬──┬──┬──┐                                        │
│  │WE│TO│TH│MA│                                        │
│  └──┴──┴──┴──┘                                        │
│  ────────●──────────────────── 1.2s                   │
│  [◀◀] drag scrubber to preview frame by frame [▶▶]   │
```

---

## Hover Tooltips

Every cue shows a tooltip on hover. Tooltip content is device-specific.

### MTPX Tooltip

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

### TextWall Tooltip

```
┌─ Cue Info ───────────────────────────────────────┐
│  🟢 TextWall                                     │
│  "welcome to the machine"                        │
│                                                  │
│  Grid: 2×2   Layout: All   Mode: Word           │
│                                                  │
│  ┌──────────┬──────────┐                        │
│  │ WELCOME  │  TO      │   ← live mini preview  │
│  ├──────────┼──────────┤                        │
│  │  THE     │ MACHINE  │                        │
│  └──────────┴──────────┘                        │
│                                                  │
│  Bar 8 Beat 1  |  02:17.143                     │
└──────────────────────────────────────────────────┘
```

The TextWall tooltip shows the actual mini grid preview inline.
For scatter mode it animates in the tooltip at the configured Hz.

### IPCP Relay Tooltip

```
┌─ Cue Info ──────────────────────────────┐
│  🟢 IPCP 505 #1                         │
│  pulse_relay                            │
│  Relay: 1                               │
│                                         │
│  HTTP: GET /W=1R01                      │
│  → ipcp505-1.extron.video              │
│                                         │
│  Bar 4 Beat 1  |  01:23.428            │
└─────────────────────────────────────────┘
```

---

## Duplicate Cue — Smart Stamp Tool

Right-click any cue or click Duplicate in the cue editor:

```
┌─ Duplicate Cue ───────────────────────────────────┐
│                                                   │
│  Duplicating: MTPX skew B=31 at Bar 4 Beat 1     │
│                                                   │
│  PLACE ON:                                        │
│  ○ Specific positions                             │
│    Bars: [ 1, 4        ]  Beats: [ 1      ]      │
│    Bars: [ 3           ]  Beats: [ 1, 3   ]      │
│    [ + Add row ]                                  │
│                                                   │
│  ○ Every X bars      [ 2 ]  starting [ 1 ]      │
│  ○ Every X beats     [ 4 ]  starting [ 1 ]      │
│  ○ Every division    [ 1/8 ▼ ]                   │
│  ○ On beats only     [ ☑1 ☐2 ☑3 ☐4 ]           │
│  ○ Custom pattern    [ 1 0 1 0 1 1 0 1 ]        │
│                                                   │
│  RANGE:                                          │
│  From: [ Bar 1  ]  To: [ Bar 32 ]  [ End ▼ ]   │
│                                                   │
│  LANES:                                          │
│  ☑ MTPX Plus #1  ☑ DirtyMixer  ☐ Atlas         │
│                                                   │
│  Preview:                                        │
│  Bar:  1    2    3    4    5    6    7    8      │
│        ●         ●    ●         ●    ●          │
│                                                   │
│  Creates 12 cues across 32 bars                 │
│                                                   │
│  [ Cancel ]              [ Stamp Cues ]          │
└───────────────────────────────────────────────────┘
```

---

## Snake / Sequence Cue — Keyframe Path Editor

When placing a Sequence/Snake cue a path editor popup appears:

```
┌─ Sequence Path Editor ─────────────────────────────────┐
│                                                        │
│  Pattern: [ Snake ▼ ]  or  [ Auto ▼ ]                │
│                                                        │
│  MANUAL KEYFRAMES:                                     │
│  ┌───┬───┬───┐                                        │
│  │ 1 │ 2 │ 3 │  ← click cells to set waypoint order  │
│  ├───┼───┼───┤                                        │
│  │ 6 │ 5 │ 4 │                                        │
│  ├───┼───┼───┤                                        │
│  │ 7 │ 8 │ 9 │                                        │
│  └───┴───┴───┘                                        │
│  Path: R1C1→R1C2→R1C3→R2C3→R2C2→R2C1→R3C1→R3C2→R3C3 │
│                                                        │
│  AUTO MODE:  [ ☑ Enable auto-path ]                   │
│  Style: [ Boustrophedon ▼ ]                           │
│  (auto calculates optimal snake through all cells)    │
│                                                        │
│  TIMING:                                              │
│  Duration per cell: [ 1 beat ▼ ]                     │
│  Overlap: [ None ▼ ]                                  │
│  Reverse: [ ☐ Play backwards ]                        │
│                                                        │
│  Action per cell: [ recall_preset ▼ ]                 │
│  Preset list: [ 1, 3, 5, 7, 2, 4, 6, 8, 9 ]         │
│  (each cell in sequence fires a DIFFERENT preset)     │
│                                                        │
│  [ Cancel ]                    [ Apply ]              │
└────────────────────────────────────────────────────────┘
```

---

## Freeform Cell Picker

Available for Checkerboard, Sequence, and TextWall cues.
Every layout is ultimately a saved freeform selection.

```
┌─ Cell Layout Picker ───────────────────────────────────┐
│                                                        │
│  Grid: [ 3×3 ▼ ]                                      │
│                                                        │
│  ┌───┬───┬───┐                                        │
│  │ ☑ │ ☐ │ ☑ │  ← tap any cell to toggle             │
│  ├───┼───┼───┤                                        │
│  │ ☐ │ ☑ │ ☐ │                                        │
│  ├───┼───┼───┤                                        │
│  │ ☑ │ ☐ │ ☑ │                                        │
│  └───┴───┴───┘                                        │
│                                                        │
│  Presets: [ X Pattern ] [ Corners ] [ Star 5×5 ]     │
│           [ Center Row ] [ All ] [ Border ] [ + ]    │
│                                                        │
│  Active cells: 5 of 9                                 │
│                                                        │
│  [ Save as... ]  Name: [ My Custom Layout ]           │
│                                                        │
│  [ Cancel ]                    [ Apply ]              │
└────────────────────────────────────────────────────────┘
```

**Built-in layouts:**

| Layout ID | Description |
|---|---|
| `all_cells` | Every cell active |
| `x_pattern` | Diagonal cross — corners + center |
| `center_only` | Single center cell |
| `corners` | Four corner cells |
| `top_row` | First row only |
| `bottom_row` | Last row only |
| `center_row` | Middle row only |
| `border` | Outer edge cells only |
| `checkerboard_a` | Alternating pattern — set A |
| `checkerboard_b` | Alternating pattern — set B |
| `star_5x5` | 5-point star on 5×5 grid |
| `star_7x7` | 5-point star on 7×7 grid |
| `custom` | User-defined freeform |

---

## Cue Types

### One-Shot Cue
Fires a single action at a specific bar/beat.
```
●  skew B=31
```

### Range / Ramp Cue
Interpolates a parameter from start to end position.
```
●──────────────●
ramp 0→255
```

### Muted Cue
Exists but won't fire. Shown dimmed with strikethrough.
```
⊘  skew B=31
```

### Repeat / Sprinkler Cue
Fires every N divisions with reset.
```
⟳──────────────────────────────⟳
sprinkler  1/8 note  B=31 → reset
```

### Strobe Cue
Alternates two states on beat division.
```
▓░▓░▓░▓░▓░▓░▓░▓░
strobe  1/8  blank↔restore
```

### Ramp Strobe Cue
Strobe with frequency build-up.
```
▓░▓░░░░▓░░░░░░░▓░░░░░░░░░░░▓
ramp strobe  1Hz → 15Hz  exponential
⚠️ Hard cap at 15Hz recommended (photosensitive safety)
```

### Checkerboard Cue
Alternating cell groups on beat.
```
▦──────────────────────────────▦
checkerboard  1/4  A↔B  [freeform picker]
```

### Sequence / Snake Cue
Fires targets in defined order.
```
🐍──────────────────────────────🐍
snake  R1C1→R1C2→R1C3→R2C3→R2C2→R2C1
```

### Random Cue
Fires random action from defined pool.
```
🎲──────────────────────────────🎲
random  1/4  presets 1–12
```

### TextWall Cue (one-shot)
Fires text + TextWall config simultaneously.
```
◉  [3×3 center row word] "welcome my son"
```

### TextWall Scatter Cue (range)
Scatter mode over a time range.
```
◉⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳⟳◉
[16×16 scatter 10Hz] "welcome to the machine" ×8
```

---

## Multi-Select and Range Operations

In Select mode, click-drag horizontally across the timeline.
Selection spans all lanes simultaneously.

```
┌─ GlitchBoard ──────────────────────────────────────────────┐
│  Mode: [ ✏️ Edit | ↔️ Select | 🔪 Razor | 👆 Cursor ]     │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ┌─ 🟢 MTPX Plus #1 ───────────────────────────────────┐  │
│  │  ●    [●▓▓▓▓▓▓▓▓▓▓▓●]         ●                    │  │
│  │       ← selected range →                            │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌─ 🟢 TextWall ───────────────────────────────────────┐  │
│  │       [◉▓▓▓▓▓▓▓▓▓▓◉▓▓▓▓▓▓▓▓▓▓]                    │  │
│  │       ← same range, lyric cues selected →           │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌─ 🟢 Dirty Mixer ────────────────────────────────────┐  │
│  │              [▓▓▓▓▓▓▓▓▓▓●▓▓▓▓▓▓]                   │  │
│  │              ← same time range →                    │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                            │
├────────────────────────────────────────────────────────────┤
│  Selected: 6 cues across 3 devices  Bar 2 Beat 1 → Bar 4  │
│  [ Delete ] [ Copy ] [ Move ] [ Mute ] [ Loop region ]    │
│  [ Add Pattern... ]                                        │
└────────────────────────────────────────────────────────────┘
```

### Add Pattern Dialog

```
┌─ Add Pattern to Selection ──────────────────────────────┐
│                                                         │
│  Range: Bar 2 Beat 1  →  Bar 6 Beat 4                 │
│  Lanes: MTPX #1, Dirty Mixer, TextWall                 │
│                                                         │
│  Pattern Type:                                          │
│  ○ Every beat                                          │
│  ○ Every X beats  [ 2 ] (1–16)                        │
│  ○ Every division [ 1/8 ▼ ]                           │
│  ○ On beats       [ ☑1 ☐2 ☑3 ☐4 ]                   │
│  ● Custom pattern [ 1 0 1 0 1 1 0 1 ] (step seq)     │
│                                                         │
│  Starting on beat [ 1 ▼ ] of bar [ 2 ▼ ]             │
│                                                         │
│  Cue to stamp:                                         │
│  [ Max Blue Separation 🔵 ▼ ] (from cue library)      │
│                                                         │
│  Preview: ● · ● · ● · ● · ● · ● · ● · ● ·           │
│                                                         │
│  [ Cancel ]                    [ Add Pattern ]         │
└─────────────────────────────────────────────────────────┘
```

---

## Performance Dashboard

```
┌─ LIVE ─────────────────────────────────┐
│                                        │
│  ▶ Bar 12  Beat 3  00:47.2            │
│  Song: Welcome to the Machine (1 of 1)│
│                                        │
│  NEXT CUE in 1.2s:                    │
│  🟢 TextWall — scatter 16×16 10Hz     │
│  "welcome to the machine" ×8          │
│                                        │
│  LAST FIRED:                           │
│  🟢 MTPX — skew B=31  (0.8s ago)     │
│  🟢 DirtyMixer — preset 12  (2.1s)   │
│  🟢 TextWall — word 2×2  (3.4s)      │
│                                        │
│  DEVICE STATUS:                        │
│  🟢 MTPX #1   🟢 Dirty   🟢 TextWall │
│  🔴 Atlas                              │
│                                        │
└────────────────────────────────────────┘
```

---

## Welcome to the Machine — Reference Performance Map

This is the worked example for the first real performance capture.

```
TIME      LANE       CUE TYPE    CONFIG                    TEXT
──────────────────────────────────────────────────────────────────────────────
1:02.5    TextWall   one-shot    3×3 center_row word       "welcome, my son"
1:05.2    TextWall   one-shot    3×3 center word           "welcome"
1:06.8    TextWall   one-shot    3×3 center_row word       "to the"
1:07.8    TextWall   clear       —                         [clear]
1:08.1    TextWall   one-shot    3×3 center_row letter     "machine....."
1:14.0    TextWall   clear       —                         [clear]
──────────────────────────────────────────────────────────────────────────────
[machine noise / stereo pan section]
──────────────────────────────────────────────────────────────────────────────
~2:10     MTPX       ramp cue    pre-peaking 0→200         ← sync to whoosh
~2:10     TextWall   scatter     3×3 all hz=2              "machine"
~2:14     TextWall   config      3×3 all hz=5              (ramp up scatter)
~2:18     TextWall   config      3×3 all hz=10             (full scatter)
──────────────────────────────────────────────────────────────────────────────
[star layout moment — "you dreamed of being a big star"]
──────────────────────────────────────────────────────────────────────────────
?:??.?    TextWall   one-shot    5×5 star_5x5 word         "you dreamed of
                                                            being a big star"
──────────────────────────────────────────────────────────────────────────────
[final verse builds...]
──────────────────────────────────────────────────────────────────────────────
FINAL     TextWall   scatter     16×16 all hz=10 ×8inst   "welcome to
                                                            the machine"
          ← 4 words × 8 instances = 32 lit cells across 256
          ← positions randomize every 100ms
          ← wall consumed by the phrase
          ← hold until end of song → clear on silence
```

---

## Build Priority

### Phase 1 — Basic Playback and Cues ✅ COMPLETE
### Phase 2 — Full Lane System ✅ COMPLETE

### Phase 3 — Selection, Patterns, and Advanced Cue Types
17. Select mode with range selection across all lanes
18. Delete/copy/move/mute selection
19. Add pattern dialog — all pattern types including step sequencer
20. Humanize timing
21. Repeat/Sprinkler cue type
22. Strobe cue type
23. Ramp Strobe cue type (with 15Hz safety warning)
24. Checkerboard cue type with freeform cell picker
25. Sequence/Snake cue type with keyframe path editor
26. Random cue type
27. Hover tooltips on all cues — SIS command preview
28. Duplicate cue smart stamp tool
29. **TextWall lane as first-class lane type**
30. **TextWall cue editor with mini preview**
31. **Import lyric_timeline.jbt → auto-populate TextWall lane**

### Phase 4 — Wall Animation and Atlas Integration
32. Sequence cue Atlas awareness — which MGP owns which cell
33. Checkerboard auto-calculation from Atlas grid mode
34. Wall animation preview before committing
35. Atlas preset integration as single cue

### Phase 5 — Setlist and Performance
36. Setlist mode — multiple songs in one .jbt file
37. Song transitions with cleanup cues
38. Performance dashboard window
39. MIDI controller integration — APC Mini Mk2

### Phase 6 — Polish
40. All themes from JoebotSDK
41. Variable BPM/tempo map
42. iOS companion app

---

## First Session Prompt for Claude Code (Phase 3)

> "I am continuing development of GlitchBoard, a native SwiftUI macOS DAW-style show control app. Phases 1 and 2 are complete (as of commit 33a77ae). Start Phase 3 by implementing: (1) Select mode with click-drag range selection spanning all lanes simultaneously, with a selection action bar showing Delete/Copy/Move/Mute/Loop/Add Pattern buttons. (2) The Add Pattern dialog with all pattern types including step sequencer. (3) The TextWall lane as a first-class lane type — it should display lyric cues with mini grid previews, and when you click a TextWall cue the right panel shows the TextWall cue editor with grid picker, layout picker, mode selector, and live mini preview. Read the full spec at https://raw.githubusercontent.com/joebot94/docs/main/GlitchBoard_Spec.md before starting."

---

## Related Documents

- `Nexus_Architecture.md` — Nexus server spec
- `JoebotSDK_Guide.md` — shared Swift toolkit
- `JBT_Format_Spec.md` — .jbt file format
- `TextWall_BuildGuide.md` — TextWall display app spec
- `LyricApp_BuildGuide.md` — Lyric App authoring spec
- `Extron_SIS_Reference.md` — device command reference

---

*GlitchBoard — DAW-style show control for the Joebot Ecosystem*
*github.com/joebot94/daw-app*
*Document version 1.3 — March 2026*
*🦖*
