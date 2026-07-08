# GlitchBoard — Pattern / Range Cues Addendum
> Repeating cue logic, bar/beat filters, range blocks, and momentary actions  
> Version: 0.5c  
> Date: 2026-07-08

---

## Purpose

Use this addendum alongside:

- `GlitchBoard_NextPass_Timeline_Authoring.md`
- `GlitchBoard_NextPass_Timeline_Authoring_Addendum.md`
- `GlitchBoard_Timeline_Stacked_Commands_Addendum.md`

This addendum captures a major GlitchBoard concept:

Not every repeated command should become dozens of separate duplicated cue blocks.

GlitchBoard needs a way to represent repeated/ranged logic as one editable object.

This pass remains mock UI/data-model only.

Do not implement real hardware sending, SIS, telnet, serial, Nexus, file save/load, audio, or MIDI.

---

# Core Idea

A normal cue is a single event.

A stacked command bundle is multiple commands at the same exact time.

A pattern/range cue is one logical object that generates repeated scheduled moments over a time range.

Examples:

```text
Bars 2–15, beats 1 and 3: trigger MGP random preset
Bars 3–25, beat 3 only: flash MTPX skew for 500 ms then return clean
Bars 4–16, beats 2 and 4: TextWall scatter pulse
Every bar from 8–32: DirtyMixer random preset on beat 1
```

These should not clutter the timeline as dozens of independent cues unless the user explicitly asks to expand/bake them later.

---

# 1. New Cue Type: Pattern Cue / Range Cue

Add a mock cue type/category:

- `single` / normal cue
- `stack` / command bundle
- `pattern` / range cue

A pattern cue has:

- device/lane
- start bar
- end bar
- beat filter
- optional subdivision filter
- action payload
- repeat mode
- optional momentary return behavior
- generated occurrence preview

---

# 2. Visual Timeline Representation

A pattern cue should display as one long block/ribbon spanning its bar range.

It should not appear as dozens of separate duplicated cue blocks by default.

Visual style:

- long translucent block across the lane
- clear start and end handles/edges
- small tick marks or pulse dots inside the block showing generated occurrences
- compact label
- pattern badge/icon

Example labels:

```text
PATTERN bars 2–15 beat 1,3
MGP random 48–71
```

```text
RANGE bars 3–25 beat 3
MTPX flash 500ms
```

Compact timeline labels:

```text
MGP rnd · bars 2–15 · b1,b3
MTPX flash · bars 3–25 · b3 · 500ms
Text scatter · b2,b4
```

---

# 3. Pattern Cue Editor

Add a Pattern Cue editor section or popover for pattern cues.

Fields:

## Range

- Start bar
- End bar
- Optional start beat
- Optional end beat

## Beat Filter

Support:

- Every beat
- Beat 1
- Beat 2
- Beat 3
- Beat 4
- Beats 1 and 3
- Beats 2 and 4
- Custom beat list

Custom beat examples:

```text
1,3
2,4
1,2,4
```

## Subdivision Filter, future/optional

Visible but can be disabled or simple for now:

- whole beats only
- 1/2 beat
- 1/4 beat
- custom tick grid

For this pass, whole beats are enough.

## Action Payload

Use the existing device-specific editor logic where possible.

Pattern cues should still know their device:

- MGP pattern uses MGP preset/random/tag-pool controls
- MTPX pattern uses MTPX scope/skew controls
- TextWall pattern uses TextWall mode/layout controls
- DirtyMixer pattern uses DirtyMixer preset/random controls
- Matrix pattern can be supported later or simple route repeats

---

# 4. Occurrence Preview

The editor should show a generated occurrence preview.

Example:

```text
Generated occurrences: 28
Bars 2–15, beats 1 and 3

2.1, 2.3, 3.1, 3.3, 4.1, 4.3 … 15.1, 15.3
```

Do not create 28 independent cues in the timeline.

This is preview only.

---

# 5. Random Logic Inside Pattern Cues

Pattern cues should support random logic without expanding into separate cue blocks.

Examples:

## MGP random preset pattern

```text
Bars 3–25
Beat 3 only
Action: MGP Random From Range 48–71
Avoid immediate repeats: On
Allow unknown presets: Off
```

Label:

```text
MGP rnd 48–71 · bars 3–25 · b3
```

## DirtyMixer random pattern

```text
Bars 8–16
Beats 2 and 4
Action: DirtyMixer Random
Pool: harsh
Rate: per occurrence
```

Label:

```text
Dirty rnd harsh · b2,b4
```

Important: random decisions are mock/logical only in this pass. Do not create actual scheduled real command sends.

---

# 6. Momentary / Flash Pattern Cues

Some actions need to happen briefly and then return.

Example request:

```text
Bars 3–25, beat 3:
make the MTPX skew flash for 500 ms and then go back
```

Support a mock momentary mode:

- Momentary: on/off
- Hold duration in ms
- Return behavior:
  - return to previous value
  - return to clean/default
  - return to specified value

Example MTPX pattern:

```text
Range: Bars 3–25
Beat Filter: Beat 3
Action: MTPX skew 0/0/31
Momentary: On
Hold: 500 ms
Return: Clean 0/0/0
```

Timeline label:

```text
MTPX flash 0/0/31 · 500ms · bars 3–25 · b3
```

Occurrence preview should show paired events conceptually:

```text
3.3 ON → +500ms OFF
4.3 ON → +500ms OFF
5.3 ON → +500ms OFF
...
```

Do not create separate visible OFF cues by default. Keep this as one pattern object.

---

# 7. Drag-to-Create Pattern Ranges

Eventually, the user should be able to drag across a lane to create a range cue.

For this pass, implement if practical or add placeholder UI.

Possible interaction:

- click and drag empty lane horizontally to select a range
- release shows context menu:
  - Create Pattern Cue From Range…
  - Create MGP Random Pattern…
  - Create MTPX Flash Pattern…
  - Cancel

If drag-to-create is too much for this pass, add a right-click option:

```text
Add Pattern Cue Here…
```

Then the editor lets the user set start/end bar manually.

---

# 8. Pattern Cue vs Duplicated Cues

Default behavior:

- Keep repeated logic as one pattern cue object.
- Do not automatically generate dozens of normal cues.

Future optional feature:

- `Expand/Bake Pattern to Cues…`

This should be visible as disabled/soon or omitted for now.

Reason:

Patterns keep the timeline readable and avoid accidental stack chaos.

---

# 9. Stacking Interaction

Pattern cues can overlap single cues or stacks.

For this pass:

- allow overlap
- warn if pattern generated occurrences collide with same-device single cues/stacks
- do not block
- do not auto-expand

Example warning:

```text
WARNING Pattern overlaps 2 existing MTPX cues in this range
```

If a pattern occurrence lands at the same time as a stack, it should be understood as contributing to that scheduled moment later, but do not build the real scheduler yet.

---

# 10. Suggested Initial Pattern Types

Implement only a small useful set if full generic pattern logic is too much.

Minimum useful pattern types:

## MGP Random Preset Pattern

- range bars
- beat filter
- random range/list/tag pool
- avoid repeats
- occurrence preview

## MTPX Skew Flash Pattern

- range bars
- beat filter
- MTPX scope/skew values
- momentary hold ms
- return clean/default
- occurrence preview

These two alone prove the concept.

---

# 11. Event Log

Log pattern actions without spam.

Examples:

```text
PATTERN Created MGP random pattern bars 3–25 beat 3
PATTERN Updated beat filter to 1,3 — 28 occurrences
PATTERN MTPX flash hold set to 500 ms
WARNING Pattern overlaps 2 existing MTPX cues
```

Do not log every generated occurrence during editing.

---

# 12. Acceptance Criteria

This pattern/range cue concept is successful when:

- app compiles
- app launches cleanly
- existing single cue editing still works
- existing stack handling still works
- pattern cues can exist as one object over a bar range
- pattern cues visually span a range on the timeline
- pattern cues show generated occurrence markers or preview
- editor supports start bar/end bar
- editor supports beat filters such as beat 3 and beats 1+3 / 2+4
- MGP random pattern can be represented mock-only
- MTPX momentary skew flash pattern can be represented mock-only
- occurrence preview updates when range/beat/action changes
- pattern overlap warnings are shown/logged but do not block
- no real hardware/SIS commands are sent

---

# Important

Do not turn this into the full playback scheduler yet.

This is a UI/data-model concept pass.

The key design principle:

Repeated logic should stay editable as one high-level pattern object unless the user explicitly expands it later.
