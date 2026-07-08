# GlitchBoard — Next Pass Spec
> Timeline authoring, zoom, cue creation, and collapsible inspector  
> Version: 0.5 planning draft  
> Date: 2026-07-08

---

## Context

GlitchBoard now has:

- dark broadcast-console UI
- Edit and Live modes
- collapsible left Rig/Devices rail
- smart cue objects
- cue selection, hover, double-click, and right-click context menu
- device-aware Cue Edit Popovers
- right inspector sync
- TextWall, DirtyMixer, Matrix, Atlas, MGP, and MTPX mock editors
- MGP visual preset library/editor
- MTPX Single / Selected / All channel scope editor
- global/per-device/per-cue timing mock UI
- Atlas future NAS layout-brain placeholder
- centralized theme colors

Preserve all of that.

This pass is about timeline authoring and workspace usability.

---

# Hard Out of Scope

Do not implement:

- real hardware
- SIS sending
- telnet
- serial
- Nexus connection
- real audio
- waveform analysis
- file save/load
- MIDI
- hardware discovery
- real Atlas service

Keep this mock UI/data-model only.

---

# Pass Goal

Make the timeline feel like an editable authoring surface, not just a display.

Add:

1. right inspector collapse/expand
2. pinch/trackpad or command-scroll zoom support
3. click/drag playhead scrubbing
4. lane selection
5. cue creation by right-clicking empty timeline space
6. lane `+` buttons to add cues at playhead
7. same-device cue copy/paste into timeline

---

# 1. Collapsible Right Inspector

The right inspector should be collapsible, similar to the left device sidebar.

Reason:
The floating Cue Edit Popover is now strong enough for fast editing, so the user should be able to collapse the inspector and reclaim horizontal timeline space.

Requirements:

- add collapse/expand button on the right inspector header
- expanded width stays similar to current
- collapsed width target: 42–64 px
- collapsed rail should show:
  - tiny label/icon: `INSPECT`
  - selected cue/device status dot if applicable
  - maybe current selected device initials
- expanding restores full inspector
- state should animate smoothly if possible
- timeline should expand into the freed space

Do not remove the inspector. It remains the persistent detail view.

Event log:

```text
UI Right inspector collapsed
UI Right inspector expanded
```

Acceptance:

- right inspector collapses
- timeline becomes wider
- expanding restores inspector state
- selected cue data is preserved

---

# 2. Timeline Zoom

Improve timeline zoom behavior.

Current zoom slider should remain, but add trackpad/mouse gesture support if practical.

Requirements:

- pinch-to-zoom on the timeline if SwiftUI/AppKit support is practical
- command-scroll zoom fallback if pinch is awkward
- zoom should center around cursor/playhead when possible
- zoom slider updates when gesture zoom changes
- zoom changes should not break cue positions
- bar/beat ruler should redraw at the new zoom level
- cue blocks should scale horizontally

If true pinch is difficult in pure SwiftUI, bridge to NSView/AppKit for magnification gesture or implement command-scroll zoom as a fallback.

Acceptance:

- user can zoom timeline with trackpad/pinch or command-scroll
- zoom slider stays in sync
- cue positions remain correct
- ruler spacing updates cleanly

---

# 3. Playhead Click + Drag / Timeline Scrubbing

The orange playhead should be interactive.

---

## Click Timeline / Ruler

Clicking empty timeline space or the ruler should:

- move the playhead to clicked position
- update top timecode/bar/beat
- update current mock transport time
- respect Snap if enabled
- free-position if Snap is disabled

Single click should not create a cue. Single click only moves/selects.

---

## Drag Playhead

Dragging the orange playhead should:

- scrub mock time continuously
- update timecode/bar/beat live
- update current/next cue preview in Live Mode
- respect Snap if enabled
- not spam the Event Log while dragging

Log only on drag end:

```text
TRANSPORT Scrubbed playhead to Bar 5 Beat 1 / 00:07.032
```

Acceptance:

- click moves playhead
- drag scrubs
- Snap affects placement
- no log spam while dragging

---

# 4. Lane Selection

Clicking a lane header should select that lane/device.

Requirements:

- selected lane header gets subtle highlight
- sidebar device selection syncs with lane selection
- inspector shows device/lane summary if no cue is selected
- selected lane is used by `Add Cue at Playhead`

Acceptance:

- lane click selects lane
- selected lane visually obvious
- inspector/sidebar sync

---

# 5. Add Cue by Right-Clicking Empty Timeline Space

Right-clicking empty space in a timeline lane should show an Add Cue context menu.

This is different from right-clicking an existing cue.

Existing cue right-click menu should remain:

- Edit Cue…
- Duplicate
- Mute/Unmute
- Copy
- Delete

Empty lane right-click menu should include device-specific cue creation.

---

## MTPX Plus #1 Lane

Menu items:

- Add MTPX Skew Cue Here…
- Add MTPX Clean Cue Here…
- Add MTPX RGB Split Cue Here…
- Paste Cue Here, if valid

---

## MGP 464 A Lane

Menu items:

- Add MGP Preset Cue Here…
- Add MGP Random Range Cue Here…
- Paste Cue Here, if valid

---

## TextWall Lane

Menu items:

- Add TextWall Cue Here…
- Add TextWall Clear Cue Here…
- Paste Cue Here, if valid

---

## DirtyMixer Lane

Menu items:

- Add DirtyMixer Preset Cue Here…
- Add DirtyMixer Random Cue Here…
- Paste Cue Here, if valid

---

## Matrix 12800 Lane

Menu items:

- Add Matrix Route Cue Here…
- Paste Cue Here, if valid

---

## Atlas Lane

Atlas is offline/mock.

Atlas cue creation should be disabled or show warning:

- Atlas offline — cue will skip

---

## Cue Creation Behavior

When creating a cue:

- calculate bar/beat/time from click location
- respect Snap setting
- create a real mock cue object
- select the new cue
- open Cue Edit Popover immediately
- update right inspector
- write Event Log entry

Example log:

```text
CUE Created MGP preset cue at Bar 4 Beat 2
EDIT Opened Cue Edit Popover for new MGP cue
```

---

# 6. Lane Header `+` Button

Each compact timeline lane header should have a small `+` button.

Clicking `+` should:

- create a default cue for that lane at the current playhead position
- select the new cue
- open the Cue Edit Popover
- update inspector
- write Event Log entry

---

## Default Cue Values

### MTPX

- action: `set_input_skew`
- scope: Single Channel
- input: 1
- RGB: 0/0/0
- label: `skew 0/0/0`

### MGP

- action: `recall_preset`
- layout mode: 2×2 Single MGP
- preset source: Single Preset
- preset: 48
- label: `preset 48`

### TextWall

- action: `display_text`
- text: `NEW TEXT`
- grid: 2×2
- layout: All Cells
- mode: Word
- label: `text "NEW TEXT"`

### DirtyMixer

- action: preset
- channel: 1
- preset: 1
- label: `preset 1`

### Matrix

- action: route
- input: 1
- output: 1
- signal: All
- label: `route 1→1`

### Atlas

- disabled/warning because Atlas is offline/mock

---

# 7. Copy / Paste Cue Here

Improve existing copy behavior if needed.

Requirements:

- Copy from existing cue context menu stores a mock copied cue
- right-click empty timeline space enables Paste Cue Here if valid
- same-device paste is allowed
- pasted cue keeps original settings but uses new bar/beat/time
- cross-device paste can be disabled for now with a warning

Example log:

```text
CUE Pasted MGP cue at Bar 6 Beat 1
WARNING Cannot paste MGP cue into TextWall lane
```

---

# 8. Add Cue at Playhead Control

If easy, add a small top/timeline control:

```text
[+ Cue at Playhead]
```

Behavior:

- adds a default cue to the currently selected lane at the current playhead
- disabled if no lane is selected
- if disabled and clicked, show/log a subtle warning

This is optional if lane `+` buttons are already clear and useful.

---

# 9. Event Log

Add event log entries for:

- right inspector collapsed/expanded
- timeline zoom changed, but do not spam during continuous zoom
- playhead scrub ended
- lane selected
- cue created
- cue created from lane `+`
- cue pasted
- cue creation blocked due to Atlas offline
- paste blocked due to device mismatch

Avoid log spam during dragging/zooming.

---

# 10. Acceptance Criteria

This pass succeeds when:

- app compiles
- app launches cleanly
- existing Edit/Live behavior still works
- existing cue editing popovers still work
- existing MGP/MTPX editors still work
- existing timing mock UI still works
- existing Atlas placeholder still works
- left sidebar collapse still works
- right inspector can collapse/expand
- timeline expands when inspector is collapsed
- pinch/gesture or command-scroll zoom works
- zoom slider stays in sync
- click ruler/timeline moves playhead
- dragging playhead scrubs mock time
- Snap affects playhead placement
- clicking lane header selects lane/device
- each lane has a small `+` button
- lane `+` creates default cue at playhead
- right-clicking empty lane space shows Add Cue menu
- adding a cue creates a mock cue object at clicked location
- new cue is selected automatically
- Cue Edit Popover opens automatically
- inspector updates to new cue
- Paste Cue Here works for same-device cues
- Atlas cue creation is disabled/warning-only while Atlas is offline
- no real hardware/SIS/Nexus/audio/file saving is added

---

# Important

This pass is about timeline authoring and workspace usability.

Do not overbuild backend architecture.

Do not redesign the app.

The goal is to make the timeline feel touchable: zoom, scrub, select lanes, add cues, and collapse panels to make room.
