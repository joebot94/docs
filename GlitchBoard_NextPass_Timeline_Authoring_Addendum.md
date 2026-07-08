# GlitchBoard — Timeline Authoring Addendum
> Edge cases and logic clarifications for `GlitchBoard_NextPass_Timeline_Authoring.md`  
> Version: 0.5a  
> Date: 2026-07-08

---

## Purpose

Use this addendum alongside:

`GlitchBoard_NextPass_Timeline_Authoring.md`

The original timeline authoring spec is still the source of the requested feature pass. This addendum tightens ambiguous behavior so the implementation does not make assumptions that become annoying later.

Preserve the existing constraints:

- mock UI/data-model only
- no real SIS sending
- no real hardware control
- no Nexus connection
- no file save/load
- preserve the current dark broadcast-console style

---

# 1. Zoom Anchor Rules

The original spec says zoom should center around cursor/playhead when possible. This needs to be explicit because those are different anchors.

## Required behavior

- Trackpad pinch / magnification gesture zoom anchors to the current mouse cursor location inside the timeline.
- Command-scroll zoom anchors to the current mouse cursor location inside the timeline.
- The UI zoom slider anchors to the current playhead position.

## Reason

If the user pinches over Bar 12 while the playhead is parked at Bar 2, the timeline should not jump back to Bar 2. Gesture zoom should preserve the user's local focus point under the cursor.

## Acceptance

- pinch/gesture zoom does not yank the visible timeline to the playhead
- slider zoom keeps the playhead visually stable when possible
- zoom slider remains synced with gesture zoom

---

# 2. Cue Collision Behavior

When adding a cue via lane `+` or right-click Add Cue Here, the new cue may land exactly where another cue already starts on the same lane.

## Required behavior

If a newly created cue would have the exact same lane and start tick/time as an existing cue:

1. Do not overwrite the existing cue.
2. Create the new cue anyway.
3. Visually stack or slightly offset the cue so both are selectable.
4. Log a warning.

Example log:

```text
WARNING New MTPX cue overlaps existing cue at Bar 4 Beat 1 — stacked visually
```

## Future option

A later pass may add collision policies such as:

- nudge by 1 tick
- replace existing cue
- group cues
- allow same-frame bundles

Do not build those policies in this pass.

## Acceptance

- creating overlapping cues does not delete old cues
- overlapping cues remain selectable
- Event Log warns about exact-time collisions

---

# 3. Keyboard Paste Behavior

The original spec covers right-click Paste Cue Here. Standard Mac paste should also have predictable behavior.

## Required behavior

When the user presses Cmd+V and a copied cue exists:

- paste the cue at the current playhead position
- paste it onto the originally copied cue's lane/device
- preserve the cue's settings
- update its bar/beat/time to the playhead
- select the pasted cue
- open Cue Edit Popover only if currently in Edit Mode
- update inspector
- log the paste

Example:

```text
CUE Pasted copied MGP cue to original MGP lane at playhead Bar 6 Beat 1
```

## If the original lane/device no longer exists

- do not paste
- show/log a warning

Example:

```text
WARNING Cannot paste cue — original MGP lane unavailable
```

## Acceptance

- Cmd+V works consistently
- Cmd+V does not require the mouse to be over the timeline
- pasted cue lands at playhead on original lane/device

---

# 4. Edit Mode vs Live Mode Popover Behavior

Creating a cue should not always auto-open an editor popover, because Live Mode should stay performance-safe and visually uncluttered.

## Required behavior

When creating a new cue:

- In Edit Mode:
  - select the new cue
  - open Cue Edit Popover immediately
  - update inspector

- In Live Mode:
  - create/select the new cue if creation is allowed
  - do not auto-open Cue Edit Popover
  - log that the cue was created without opening the editor because Live Mode is active

Example log:

```text
CUE Created MGP cue at playhead — editor not opened in Live Mode
```

## Optional behavior

In Live Mode, cue creation may be disabled entirely if that feels safer. If disabled, log a warning instead:

```text
WARNING Cue creation is disabled in Live Mode
```

For this pass, either approach is acceptable, but do not obscure Live Mode with automatic popovers.

## Acceptance

- Edit Mode creation opens popover
- Live Mode creation does not auto-open popover
- Live Mode cockpit remains unobstructed

---

# 5. Scrubbing Does Not Execute Cues

Dragging or clicking the playhead updates the timeline preview, but it should not simulate real cue execution state.

## Required behavior

Scrubbing the playhead:

- updates mock timecode/bar/beat
- updates playhead position
- updates current/next cue preview
- does not fire mock cues
- does not apply cue effects to persistent mock device state
- does not attempt to undo cues when scrubbing backward

## Reason

Scrubbing backward across a DirtyMixer cue should not try to reverse a previous preset. This pass is timeline preview only.

## Acceptance

- scrub updates static schedule preview only
- no mock hardware state changes happen during scrub
- Event Log does not claim cues fired during scrub

---

# 6. Final Implementation Reminder

Implement this addendum as clarification to the original timeline authoring spec.

Do not broaden the pass.

The goal remains:

- collapsible right inspector
- timeline zoom
- playhead click/drag scrubbing
- lane selection
- add cue from right-click empty lane
- lane `+` cue creation
- same-device paste
- predictable edge behavior

Keep everything mock-only.
