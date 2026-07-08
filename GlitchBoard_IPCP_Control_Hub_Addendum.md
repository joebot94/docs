# GlitchBoard — IPCP Control Hub Addendum
> IPCP lane model, named sub-targets, expandable sublanes, IR/serial/relay cues, and optional macros  
> Version: 0.6 planning draft  
> Date: 2026-07-08

---

## Purpose

This document captures the IPCP / control processor concept for GlitchBoard.

The core problem:

An Extron IPCP or similar control processor can control many unrelated physical things: IR devices, serial devices, relays, digital outputs, RGB lights, VHS decks, projectors, lamps, scalers, etc. If every endpoint becomes a permanent top-level timeline lane, the timeline will become overcrowded fast.

The goal is to make IPCP control usable without adding 15+ permanent rows.

This is a future concept/spec. Do not implement during the current timeline authoring pass unless explicitly requested.

---

# Core Design Rule

IPCP should be represented as one top-level Control Hub lane with named sub-targets inside it.

Default view:

```text
IPCP 505 / Control Hub
```

Cues inside that lane target specific IPCP endpoints:

```text
IR2 — RGB Strip → Red
IR3 — VHS Deck → Stop
Relay1 — Lamp → Off
Serial1 — Scaler → Freeze
```

These are separate cues unless intentionally grouped into a macro.

---

# 1. One Top-Level IPCP Lane

Do not create a permanent top-level lane for every IPCP-controlled device by default.

Avoid this as the default timeline model:

```text
RGB Strip
VHS Deck
Projector
Lamp
Scaler Serial
Relay 1
Relay 2
IR Port 1
IR Port 2
IR Port 3
```

Instead use:

```text
IPCP 505 / Control Hub
```

The cue object stores which sub-target it controls.

---

# 2. Named Sub-Targets

The IPCP device should contain a list of named sub-targets.

Example model:

```text
IPCP 505
  IR Port 1 — Projector
  IR Port 2 — RGB Strip
  IR Port 3 — VHS Deck
  Serial Port 1 — Scaler
  Serial Port 2 — Matrix Utility
  Relay 1 — Lamp
  Relay 2 — Sign Light
  Digital Out 1 — Trigger
```

Each sub-target should have:

- unique ID
- display name
- target type
- physical port
- optional notes
- command library
- optional color/accent for timeline labels

Target types:

- IR
- Serial
- Relay
- Digital Output
- RGB Light
- Macro / Virtual Target

---

# 3. Collapsed vs Expanded IPCP Lane

The IPCP lane should support collapsed and expanded display modes.

## Collapsed

One clean lane with compact cue labels:

```text
IPCP 505: [RGB red]      [VHS stop]      [lamp off]      [SER freeze]
```

This keeps the timeline readable.

## Expanded

Temporary/detail view showing sub-target lanes:

```text
IPCP 505 / Control Hub
  IR2 — RGB Strip:       [red]        [blue]       [random]
  IR3 — VHS Deck:              [stop]       [record]
  Relay1 — Lamp:                         [off]
  Serial1 — Scaler:      [freeze]
```

Expanded sublanes are for precision editing. They should not become permanent top-level clutter unless the user explicitly pins them later.

---

# 4. IPCP Lane Filter Views

Add optional target filters to avoid the collapsed IPCP lane becoming a junk drawer.

Possible filters:

```text
[All Targets] [IR] [Serial] [Relays] [RGB] [VHS] [Lighting]
```

Filtering should change what is visible in the IPCP lane, not delete cues.

---

# 5. IPCP Cue Model

An IPCP cue is a normal timeline cue that targets an IPCP sub-target.

Fields:

- device: IPCP 505
- target ID
- target display name
- target type
- command ID
- command display name
- optional parameters
- optional color swatch
- optional command preview
- timing offset rules like any other device cue

Example cue:

```text
Device: IPCP 505
Target: IR2 — RGB Strip
Command: Red
Label: RGB red
```

Example cue:

```text
Device: IPCP 505
Target: IR3 — VHS Deck
Command: Stop
Label: VHS stop
```

These two cues are unrelated unless intentionally grouped by the user.

---

# 6. IPCP Cue Editor

The IPCP editor should be target-first.

Suggested UI:

```text
Target Type:
[IR] [Serial] [Relay] [Digital Out] [RGB]

Target:
IR2 — RGB Strip

Command:
[Red] [Green] [Blue] [Purple] [White] [Off] [Random]
```

For VHS:

```text
Target Type:
IR

Target:
IR3 — VHS Deck

Command:
[Play] [Stop] [Record] [Pause] [Rewind] [FF]
```

For serial:

```text
Target Type:
Serial

Target:
Serial1 — Scaler

Command:
[Freeze] [Unfreeze] [Input 1] [Input 2] [Power]
```

For relay:

```text
Target Type:
Relay

Target:
Relay1 — Lamp

Command:
[On] [Off] [Toggle]
```

---

# 7. RGB / Color Swatch Commands

For RGB light targets, color commands should render as actual color swatches.

Example:

```text
[■ Red] [■ Orange] [■ Yellow] [■ Green] [■ Cyan] [■ Blue] [■ Purple] [□ White] [Off] [Random]
```

Timeline cue labels should remain compact but visually distinct:

```text
RGB red
RGB blue
RGB rnd
RGB off
```

If possible, the cue block accent color should correspond to the selected RGB color.

Random color can use a multicolor/rainbow accent or a `RND` badge.

---

# 8. Compact Timeline Labels

IPCP labels should be short.

Examples:

```text
RGB red
RGB blue
RGB rnd
VHS stop
VHS rec
SER freeze
Relay off
Lamp on
```

The inspector/popover can show the full target and command details.

---

# 9. IPCP Macros Are Optional, Not Default

Macros should exist only when the user intentionally wants a chained multi-step sequence.

Normal IPCP actions remain separate cues.

Example normal unrelated cues:

```text
Bar 4 Beat 1: IR2 RGB Strip → Red
Bar 6 Beat 3: IR3 VHS Deck → Stop
```

These should not automatically become one macro.

Example macro cue:

```text
Tape Start Ritual
0.0s VHS Record
0.2s RGB Red
1.0s TextWall REC
```

Macro cue timeline label:

```text
IPCP macro · 3 steps · 1.0s
```

Macros can be a future feature. The first IPCP pass should focus on separate single-action IPCP cues with named targets.

---

# 10. Interaction With Stacks and Patterns

IPCP cues should follow the same core timeline object rules as other devices.

## Single Cue

One IPCP sub-target command at one time.

```text
IR3 VHS Stop
```

## Stack / Bundle

Multiple IPCP commands at the same exact time on the IPCP lane.

Example:

```text
Bar 8 Beat 1 — IPCP stack
- RGB Strip → Red
- Relay1 Lamp → Off
- VHS Deck → Stop
```

This is allowed and should be visible as a stack/bundle.

## Pattern / Range Cue

A repeated IPCP action over a range.

Example:

```text
Bars 4–16, beats 2 and 4:
RGB Strip random color
```

This should be represented as one pattern/range cue, not dozens of duplicated cues.

---

# 11. Future Backend Notes

Later, IPCP cue execution may map to:

- IR command send
- serial command send
- relay on/off/toggle
- digital output pulse
- HTTP/ethernet command if supported

But this document is only about UI/data-model structure.

Do not add real command sending yet.

---

# 12. Acceptance Criteria For A Future IPCP UI Pass

A future IPCP pass succeeds when:

- IPCP appears as one top-level Control Hub lane
- IPCP has named sub-targets
- collapsed lane shows compact mixed IPCP cues
- expanded mode shows sub-target lanes
- IPCP cue editor supports target type, target, command
- RGB targets show color swatches
- VHS/IR targets show command buttons like Play/Stop/Record
- serial targets show named command buttons
- relay targets show On/Off/Toggle
- cue labels stay compact
- IPCP actions remain separate unless intentionally grouped into a macro
- stacks work on the IPCP lane
- patterns can eventually target IPCP sub-targets
- no real IPCP hardware commands are sent

---

# Important

The IPCP should feel like a control hub, not a pile of extra lanes.

Default:

```text
One IPCP lane
many named sub-targets
expandable sublanes when needed
optional macros later
```

This keeps the timeline clean while still letting GlitchBoard control lots of physical things.
