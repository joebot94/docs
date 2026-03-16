# Lyric App — Build Guide v2.0
> Joebot Ecosystem Lyric Authoring and Sync App
> GitHub: github.com/joebot94/lyric-app
> Document version 2.0 — March 2026
> Supersedes v1.0 — expanded scope to include full TextWall configuration authoring
> 🦖 Joebot Ecosystem

---

## What The Lyric App Is

The Lyric App is a native SwiftUI macOS authoring tool for syncing lyrics and text content
to an audio timeline. It is NOT just a playback tool — it is a performance design tool.

Every lyric cue carries not just text and timing but the complete TextWall display
configuration for that moment. Grid size, layout, mode, font, color — all baked in at
authoring time. The exported .jbt file is a complete performance script. Load it into
TextWall and it plays back perfectly every time, no configuration needed during performance.

It is also GlitchBoard's content source. Import the .jbt into GlitchBoard and every lyric
becomes a cue on the TextWall lane, editable alongside all other hardware cues on the
same timeline.

---

## Core Philosophy

- **Author once, play anywhere** — .jbt contains everything, no live configuration needed
- **What you see is what fires** — mini preview in every cue editor shows exactly how
  TextWall will render it
- **TextWall-aware** — queries TextWall capabilities from Nexus, shows only available
  layouts and modes
- **GlitchBoard compatible** — exported .jbt imports directly into GlitchBoard TextWall lane
- **Standalone** — works without Nexus for authoring, needs Nexus for live playback

---

## Layout — Main Window

```
┌─ Lyric App ──────────────────────────────────────────────────┐
│  LYRIC APP                             🟢 Nexus  ⚙️          │
│  [📂 Load Audio] [💾 Save JBT] [📤 Export] [▶ Preview]      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  BAR: 1    2    3    4    5    6    7    8    9    10        │
│  BEAT:1234 1234 1234 1234 1234 1234 1234 1234 1234 1234     │
│                                                              │
│  ████████████████████████████████████████████████████████   │
│  [waveform — click to place lyric dot]                      │
│                                                              │
│  ┌─ Lyric Lane ────────────────────────────────────────┐    │
│  │                                                     │    │
│  │  ◉           ◉         ◉              ◉            │    │
│  │  [3×3        [3×3      [2×2           [16×16       │    │
│  │  ctr row]    center]   corners]       scatter]     │    │
│  │  "welcome    "welcome  "welcome to    "welcome to  │    │
│  │   my son"    "         the machine"   the machine" │    │
│  │                                                     │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
├──────────────────────────────────────────────────────────────┤
│  ┌─ Lyric List ──────────────────────┐                      │
│  │  #   Time      Text          Mode │                      │
│  │  01  1:02.5   welcome my son  W  │                      │
│  │  02  1:05.2   welcome         W  │                      │
│  │  03  1:06.8   to the          W  │                      │
│  │  04  1:07.8   [clear]         —  │                      │
│  │  05  1:08.1   machine.....    L  │                      │
│  │  06  1:14.0   [clear]         —  │                      │
│  └───────────────────────────────────┘                      │
│  Snap: [1/4 ▼]  Zoom: [–][+][Fit]  Theme: [Joebot ▼]      │
└──────────────────────────────────────────────────────────────┘
```

---

## Lyric Cue Editor — The Heart of the App

When you click on the waveform or click an existing dot this popover appears.
This is where all the magic happens — text + timing + full TextWall config in one place.

```
┌─ Lyric Cue — 1:02.5 ────────────────────────────────────────┐
│                                                              │
│  TEXT                                                        │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ welcome, my son                                        │ │
│  └────────────────────────────────────────────────────────┘ │
│  Style: [ Verse ▼ ]   Hardware advance: [ ☑ Relay 2 ]      │
│                                                              │
│  TEXTWALL CONFIGURATION                                      │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Grid:   [ 3×3 ▼ ]                                  │    │
│  │  Layout: [ Center Row ▼ ]                           │    │
│  │  Mode:   [ Word ▼ ]                                 │    │
│  │  Font:   [ Helvetica Neue ▼ ]  Weight: [ Bold ▼ ]  │    │
│  │  Color:  [ ■ White ]  BG: [ ■ Black ]              │    │
│  │  Transition: [ Cut ▼ ]  Duration: [ — ]            │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  PREVIEW                                                     │
│  ┌──────────────────────────────────┐                       │
│  │  ┌───────┬───────┬───────┐      │                       │
│  │  │       │       │       │      │                       │
│  │  │  WEL  │  MY   │  SON  │      │                       │
│  │  │  COME │       │       │      │                       │
│  │  ├───────┼───────┼───────┤      │                       │
│  │  │       │       │       │      │                       │
│  │  │       │       │       │      │                       │
│  │  ├───────┼───────┼───────┤      │                       │
│  │  │       │       │       │      │                       │
│  │  │       │       │       │      │                       │
│  │  └───────┴───────┴───────┘      │                       │
│  └──────────────────────────────────┘                       │
│  Preview updates live as you change settings                │
│                                                              │
│  Duration: [ Until next cue ▼ ]                            │
│  Position: [ 1:02.5 ]  [ Snap to grid ▼ ]                 │
│                                                              │
│  [ Delete Cue ]  [ Cancel ]  [ Save Cue ]                  │
└──────────────────────────────────────────────────────────────┘
```

### Scatter Mode Editor

When mode is set to Scatter additional controls appear:

```
│  Mode:   [ Scatter ▼ ]                                      │
│                                                              │
│  ┌─ Scatter Settings ──────────────────────────────────┐    │
│  │  Instances per word: [ 8  ] (1–32)                  │    │
│  │  Speed:              [ 10 ] Hz (1–20)               │    │
│  │  Words shown:        [ 4  ] simultaneously          │    │
│  │                                                      │    │
│  │  ⚠️ 15Hz+ may cause issues for photosensitive users │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
│  PREVIEW  [ ▶ Animate ]                                     │
│  ┌──────────────────────────────────┐                       │
│  │  · · W · · T · ·                │                       │
│  │  M · · · W · · T                │                       │
│  │  · · T · · M · ·                │                       │  
│  │  W · · M · · · T                │                       │
│  │  · T · · W · M ·                │                       │
│  └──────────────────────────────────┘                       │
│  [ ▶ Animate ] shows scatter randomizing in real time       │
```

### Reveal Mode Editor

When mode is set to Reveal a scrubber appears in the preview:

```
│  Mode:   [ Reveal ▼ ]                                       │
│                                                              │
│  ┌─ Reveal Settings ───────────────────────────────────┐    │
│  │  Direction: [ Left → Right ▼ ]                      │    │
│  │  Duration:  [ 4.0s ]  (matches cue length)          │    │
│  │  Timing:    [ Even spread ▼ ]                       │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
│  PREVIEW                                                     │
│  ┌──────────────────────────────────┐                       │
│  │  ┌──┬──┬──┬──┐                  │                       │
│  │  │WE│TO│TH│MA│                  │                       │
│  │  └──┴──┴──┴──┘                  │                       │
│  │  ──────●──────────────── 1.2s   │                       │
│  │  [◀◀] scrub to preview frame    │                       │
│  └──────────────────────────────────┘                       │
```

---

## Lyric List Panel

The bottom panel shows all lyrics as a scrollable list.
Click any row to jump to that cue in the waveform and open the editor.

```
┌─ Lyrics (24 cues) ──────────────────────────────────────────┐
│  #    Time      Grid   Layout      Mode    Text             │
│  ────────────────────────────────────────────────────────   │
│  01   1:02.5   3×3    Center Row  Word    welcome, my son  │
│  02   1:05.2   3×3    Center      Word    welcome          │
│  03   1:06.8   3×3    Center Row  Word    to the           │
│  04   1:07.8   —      —           CLEAR   [clear]          │
│  05   1:08.1   3×3    Center Row  Letter  machine.....     │
│  06   1:14.0   —      —           CLEAR   [clear]          │
│  07   1:18.7   3×3    Center      Line    where have you   │
│  ...                                                        │
│                                                             │
│  [ + Add ]  [ Import from text ]  [ Export as text ]       │
└─────────────────────────────────────────────────────────────┘
```

**Mode abbreviations in list:**
- W — Word
- L — Letter  
- S — Scatter
- R — Reveal
- Li — Line
- C — Checkerboard
- — — Clear event

---

## Import from Text File

Paste or load a plain text file. App auto-spaces lyrics evenly across
the song duration. You then drag dots to fine-tune timing.

```
┌─ Import Lyrics ─────────────────────────────────────────────┐
│                                                             │
│  Paste lyrics (one line per cue):                          │
│  ┌───────────────────────────────────────────────────────┐ │
│  │ welcome, my son                                       │ │
│  │ welcome                                               │ │
│  │ to the                                                │ │
│  │ machine.....                                          │ │
│  │ where have you been?                                  │ │
│  └───────────────────────────────────────────────────────┘ │
│                                                             │
│  Spacing: [ Even across song ▼ ]                           │
│  Default grid: [ 3×3 ▼ ]                                   │
│  Default layout: [ Center Row ▼ ]                          │
│  Default mode: [ Word ▼ ]                                  │
│                                                             │
│  Creates 5 cues spaced evenly — adjust timing after import │
│                                                             │
│  [ Cancel ]                    [ Import ]                  │
└─────────────────────────────────────────────────────────────┘
```

---

## .jbt File Format — lyric_timeline v2.0

Expanded from v1.0 to include full TextWall configuration per cue.

```json
{
  "jbt_type": "lyric_timeline",
  "version": "2.0",
  "created_at": "2026-03-15T00:00:00Z",
  "name": "Welcome to the Machine",
  "payload": {
    "audio_file": "~/Music/welcome_to_the_machine.wav",
    "audio_duration_s": 482.0,
    "bpm": 72,
    "time_signature": "4/4",
    "hardware_config": {
      "enabled": true,
      "device_id": "device.ipcp505.1",
      "action": "pulse_relay",
      "relay": 2,
      "pulse_duration_ms": 100
    },
    "lyrics": [
      {
        "id": "lyric_001",
        "time": 62.5,
        "bar": 8,
        "beat": 1,
        "text": "welcome, my son",
        "style": "verse",
        "hardware_advance": true,
        "textwall": {
          "grid_size": "3x3",
          "layout": "center_row",
          "mode": "word",
          "font": "Helvetica Neue",
          "weight": "bold",
          "color": "#FFFFFF",
          "background": "#000000",
          "transition": "cut"
        }
      },
      {
        "id": "lyric_002",
        "time": 65.2,
        "bar": 8,
        "beat": 3,
        "text": "welcome",
        "style": "verse",
        "hardware_advance": false,
        "textwall": {
          "grid_size": "3x3",
          "layout": "center_only",
          "mode": "word",
          "font": "Helvetica Neue",
          "weight": "bold",
          "color": "#FFFFFF",
          "background": "#000000",
          "transition": "cut"
        }
      },
      {
        "id": "lyric_clear_001",
        "time": 67.8,
        "type": "clear",
        "text": null,
        "textwall": null
      },
      {
        "id": "lyric_003",
        "time": 68.1,
        "bar": 9,
        "beat": 1,
        "text": "machine.....",
        "style": "verse",
        "hardware_advance": true,
        "textwall": {
          "grid_size": "3x3",
          "layout": "center_row",
          "mode": "letter",
          "font": "Helvetica Neue",
          "weight": "bold",
          "color": "#FFFFFF",
          "background": "#000000",
          "transition": "cut"
        }
      },
      {
        "id": "lyric_scatter_final",
        "time": 421.0,
        "bar": 112,
        "beat": 1,
        "text": "welcome to the machine",
        "style": "chorus",
        "hardware_advance": false,
        "textwall": {
          "grid_size": "16x16",
          "layout": "all_cells",
          "mode": "scatter",
          "scatter_instances": 8,
          "scatter_hz": 10,
          "font": "Helvetica Neue",
          "weight": "bold",
          "color": "#FFFFFF",
          "background": "#000000",
          "transition": "cut"
        }
      }
    ]
  }
}
```

---

## Nexus Playback Messages

When playing back the Lyric App fires TWO messages per cue simultaneously:

### 1. lyric_update (content)
```json
{
  "type": "lyric_update",
  "source": "lyric_app",
  "payload": {
    "text": "welcome, my son",
    "style": "verse",
    "index": 0,
    "total": 24
  }
}
```

### 2. textwall_config (display instructions)
```json
{
  "type": "intent",
  "source": "lyric_app",
  "targets": ["textwall_v1"],
  "payload": {
    "action": "set_config",
    "params": {
      "grid_size": "3x3",
      "layout": "center_row",
      "mode": "word",
      "color": "#FFFFFF",
      "background": "#000000"
    }
  }
}
```

TextWall receives both, switches config AND displays text simultaneously.
One cue, one moment, perfect sync.

---

## GlitchBoard Import

When imported into GlitchBoard each lyric cue becomes a TextWall lane cue:

```
┌─ 🟢 TextWall ───────────────────────────────────────────┐
│                                                         │
│  ◉           ◉      ◉         ◉              ◉⟳⟳⟳⟳  │
│  [3×3        [3×3   [CLEAR]   [2×2           [16×16   │
│  ctr row     ctr             corners]        scatter] │
│  word]       word]                                     │
│  "welcome    "wel"           "welcome to    "welcome  │
│   my son"                    the machine"   to the    │
│                                              machine"  │
└─────────────────────────────────────────────────────────┘
```

Each cue is editable inside GlitchBoard — change timing, change TextWall config,
add hardware cues on other lanes at the same moment. Full integration.

---

## Minimal Viable Version — Tonight's Build

For the deadline the bare minimum that works:

### Phase 0 — Deadline Build (tonight)
A minimal Swift app with NO waveform. Just a list editor.

```
┌─ Lyric App — Quick Edit ─────────────────────────────────┐
│  [📂 Load Audio (optional)] [💾 Save JBT] [▶ Preview]   │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Song: Welcome to the Machine  Duration: 8:02           │
│                                                          │
│  ┌────┬──────────┬──────────────────────┬─────────────┐ │
│  │ #  │  Time    │  Text                │  Config     │ │
│  ├────┼──────────┼──────────────────────┼─────────────┤ │
│  │ 01 │ 1:02.5   │ welcome, my son      │ 3×3 ctr W   │ │
│  │ 02 │ 1:05.2   │ welcome              │ 3×3 ctr W   │ │
│  │ 03 │ 1:06.8   │ to the               │ 3×3 ctr W   │ │
│  │ 04 │ 1:07.8   │ [clear]              │ —           │ │
│  │ 05 │ 1:08.1   │ machine.....         │ 3×3 ctr L   │ │
│  └────┴──────────┴──────────────────────┴─────────────┘ │
│  [ + Add Row ]  [ Delete ]                              │
│                                                          │
│  Click any row to edit text, time, and TextWall config  │
├──────────────────────────────────────────────────────────┤
│  Quick config (applies to selected row):                │
│  Grid: [3×3 ▼]  Layout: [Center Row ▼]  Mode: [Word ▼] │
│                                                          │
│  Mini preview:                                          │
│  ┌───┬───┬───┐                                         │
│  │WEL│MY │SON│                                         │
│  └───┴───┴───┘                                         │
└──────────────────────────────────────────────────────────┘
```

No waveform needed. Type timestamps manually or paste from a text file.
Each row has a quick config selector. Mini preview updates live.
Export `.jbt`. Done.

This is buildable in one Codex session tonight.

---

## Build Priority

### Phase 0 — Deadline Build (TONIGHT)
1. SwiftUI list-based lyric editor — no waveform
2. Add/edit/delete rows with timestamp, text, TextWall config
3. Quick config selector per row — grid, layout, mode
4. Mini preview updates live per selected row
5. Export `lyric_timeline.jbt` v2.0
6. NexusStatusIndicator in toolbar
7. Connect to Nexus, fire lyric_update + textwall_config on playback

### Phase 1 — Proper Timeline
8. Load audio via AVFoundation
9. Waveform display via SwiftUI Canvas
10. Bar/beat ruler synced to waveform
11. Click waveform to place lyric dots
12. Drag dots to adjust timing
13. Snap to grid

### Phase 2 — Full Cue Editor
14. Full popover cue editor replacing quick config
15. Scatter settings panel
16. Reveal settings with scrubber
17. Hardware advance per cue
18. Import lyrics from text file with auto-spacing

### Phase 3 — GlitchBoard Integration
19. GlitchBoard imports lyric_timeline.jbt
20. Each lyric becomes TextWall lane cue
21. Editable inside GlitchBoard timeline

### Phase 4 — Polish
22. All themes from JoebotSDK
23. iOS companion for live lyric editing
24. Shared setlist mode with GlitchBoard

---

## Tonight's Codex Session Prompt

> "I am building the Lyric App, part of the Joebot Ecosystem. Tonight I need Phase 0 — a bare bones SwiftUI macOS app with NO waveform. Just a list-based editor where I can add rows with a timestamp, text, and TextWall configuration (grid size, layout, mode). Each row should have a mini preview grid that updates live showing how TextWall will render the text. The app should export a lyric_timeline.jbt file in v2.0 format where each lyric cue includes the full TextWall config. Include NexusStatusIndicator in toolbar. On playback fire both lyric_update and textwall_config intents to Nexus simultaneously for each cue. Use Joebot Classic dark theme — dark grey background, orange accents. Read the full spec at https://raw.githubusercontent.com/joebot94/docs/main/LyricApp_BuildGuide.md and the TextWall spec at https://raw.githubusercontent.com/joebot94/docs/main/TextWall_BuildGuide.md"

---

## Related Documents

- `Nexus_Architecture.md` — Nexus server spec
- `TextWall_BuildGuide.md` — TextWall app spec
- `GlitchBoard_Spec.md` — GlitchBoard DAW spec
- `JBT_Format_Spec.md` — .jbt file format
- `JoebotSDK_Guide.md` — shared Swift toolkit

---

*Lyric App — Performance lyric authoring for the Joebot Ecosystem*
*github.com/joebot94/lyric-app*
*Document version 2.0 — March 2026*
*Expanded from v1.0 — full TextWall configuration authoring per cue*
*🦖*
