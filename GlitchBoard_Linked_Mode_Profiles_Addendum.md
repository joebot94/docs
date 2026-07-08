# GlitchBoard — Linked Mode Profiles Addendum
> Device pairing, video wall controller lane, mode profile linking, and coordinated layout switching  
> Version: 0.6b planning draft  
> Date: 2026-07-08

---

## Purpose

This document captures an important future GlitchBoard concept:

Some devices should be controllable individually, but also optionally linked together through named layout/mode profiles.

Example use case:

- The video wall controller is IR-controlled through the IPCP.
- The MGP changes to a 2×2 layout preset.
- The video wall controller should also switch to its matching 2×2 mode.
- TextWall should optionally change to a matching 2×2 grid/mode.
- Atlas should eventually understand the global layout and routing role.

The goal is not to hardwire devices together. The goal is to allow optional configuration-based linking.

This is a future concept/spec. Do not implement during the current timeline authoring pass unless explicitly requested.

---

# Core Concept

GlitchBoard should support optional linked mode profiles.

A mode profile is a named layout/state that can coordinate multiple device actions.

Examples:

```text
2×2 Clean
2×3 Wide
3×2 Tall
3×3 Grid
3×4 Wall
4×4 Full Wall
```

Each profile can define recommended or linked actions for multiple devices.

Example:

```text
Mode Profile: 2×2 Clean

MGP 464 A:
- recall preset 48

Video Wall Controller:
- IPCP target: IR4 — Video Wall Controller
- command: 2×2 Mode

TextWall:
- grid: 2×2
- layout: All Cells

Matrix / Atlas:
- route group: clean 2×2
```

---

# 1. Video Wall Controller As A Device

The video wall controller should appear as a device/lane in GlitchBoard, even if it is physically triggered through the IPCP.

Reason:

The user thinks of it as a video wall controller, not just as `IPCP IR Port 4`.

Recommended model:

```text
Device: Video Wall Controller
Transport: IPCP IR
Physical path: IPCP 505 / IR Port 4
Command library: 2×2, 2×3, 3×2, 3×3, 3×4, 4×4, etc.
```

This lets the timeline show:

```text
Video Wall: [2×2 mode] [3×2 mode] [4×4 mode]
```

instead of only:

```text
IPCP: [IR4 cmd 003]
```

The IPCP remains the control transport, but the user-facing device is the video wall controller.

---

# 2. Device Pairing / Linked Actions

GlitchBoard should allow optional device pairing/linking.

Examples:

```text
When MGP switches to 2×2 preset group,
optionally also trigger Video Wall Controller 2×2 mode.
```

```text
When global wall mode becomes 3×2,
optionally update TextWall grid to 3×2.
```

```text
When Atlas layout mode changes to 3×4,
optionally cue Matrix routing, MGP presets, Video Wall mode, and TextWall grid.
```

This should be configuration-driven, not hardcoded.

---

# 3. Link Profiles

Add a future concept called `Link Profile` or `Mode Profile`.

A profile maps a high-level mode to per-device actions.

Example:

```text
Profile: 3×2 Glitch Wall

MGP 464 A:
- layout mode: 3×2 Dual MGP
- preset pool: 3×2 safe/random

Video Wall Controller:
- command: 3×2 Mode
- transport: IPCP IR4

TextWall:
- grid: 3×2
- layout: All Cells

Matrix 12800:
- route group: 3×2 wall inputs

Atlas:
- active layout: 3×2
```

---

# 4. Linked Action Cue

Eventually, a cue can be a normal single-device cue or a linked mode cue.

## Single-device cue

```text
MGP preset 52
```

Only the MGP changes.

## Linked mode cue

```text
Set Wall Mode: 3×2 Glitch Wall
```

This generates/coordinated child actions:

```text
MGP: set 3×2 preset pool
Video Wall: IPCP IR command 3×2
TextWall: grid 3×2
Matrix: route 3×2 group
Atlas: active layout 3×2
```

Visually, linked mode cues can appear as:

```text
MODE 3×2
```

or:

```text
WALL 3×2
```

The inspector/popover should show the child device actions and whether each is enabled.

---

# 5. Linking Should Be Optional Per Cue

Do not automatically force linked behavior.

For an MGP cue, the editor could eventually show:

```text
Linked mode actions:
[ ] Also switch Video Wall Controller
[ ] Also update TextWall grid
[ ] Also update Atlas layout
[ ] Also apply Matrix route group

Mode profile: 2×2 Clean
```

Or a simpler toggle:

```text
Use linked mode profile: Off / On
Profile: 2×2 Clean
```

Default should be Off until the user configures profiles.

---

# 6. Atlas Role

Atlas is the natural future owner of layout intelligence.

Atlas should eventually know:

- current wall mode
- which MGP owns which cells
- which video wall controller mode matches the active layout
- which TextWall grid matches the active layout
- which Matrix route group is required
- which preset pools are compatible with the active layout
- panic/restore behavior for the current layout

GlitchBoard remains the timeline/cockpit. Atlas can become the layout brain.

---

# 7. Video Wall Controller Command Library

The video wall controller should have a named command library.

Example commands:

```text
2×2 Mode
2×3 Mode
3×2 Mode
3×3 Mode
3×4 Mode
4×3 Mode
4×4 Mode
Single Fullscreen
Rotate / Flip / Mirror if supported
Reset / Default
```

The physical transport can be:

```text
IPCP 505 → IR Port X
```

The timeline/editor should show the semantic command name, not the raw IR code.

---

# 8. Visual UI Ideas

## Device lane

```text
Video Wall Controller: [2×2]      [3×2]       [4×4]
```

## Linked mode cue

```text
Mode Profile: 3×2 Glitch Wall

Enabled linked actions:
✓ MGP 464 A preset pool 3×2 safe
✓ Video Wall Controller 3×2 via IPCP IR4
✓ TextWall grid 3×2
✓ Atlas active layout 3×2
□ Matrix route group 3×2
```

## Inspector warning

If a linked device is offline/unconfigured:

```text
WARNING: Video Wall Controller transport not configured — linked IR command will skip
```

---

# 9. Relationship To Stacks And Patterns

Linked mode cues may behave like generated stacks under the hood.

Example:

```text
At Bar 8 Beat 1: Set Wall Mode 3×2
```

Generated/coordinated child actions:

```text
MGP preset/layout action
Video Wall IR command
TextWall grid action
Atlas layout state
Matrix route action
```

Do not expand these into separate visible cues by default.

Show them as one linked mode cue with child actions in the inspector/popover.

A pattern/range cue could also trigger a linked mode profile later, but this is advanced and should not be implemented until the core profile system exists.

---

# 10. Acceptance Criteria For A Future Linked Mode Pass

A future linked mode/profile pass succeeds when:

- video wall controller can be represented as a device even if transported through IPCP IR
- device has semantic command names like 2×2 / 3×2 / 4×4
- mode profiles can map one high-level wall mode to several device actions
- linked actions are optional and configuration-driven
- an MGP cue can optionally reference a mode profile
- TextWall can optionally follow mode profile grid/layout
- Atlas can be shown as future layout brain for profiles
- linked mode cue shows child actions in inspector/popover
- offline/unconfigured linked devices warn but do not break the cue
- no real hardware commands are sent during mock implementation

---

# Important

This feature should not make every cue secretly trigger everything.

Linking must be explicit, visible, and optional.

The user should always be able to see:

- what high-level mode is being requested
- which devices are linked
- which linked actions will run
- which linked actions are disabled/offline/unconfigured

Core rule:

```text
Devices stay independent by default.
Mode profiles coordinate them only when explicitly enabled.
```
