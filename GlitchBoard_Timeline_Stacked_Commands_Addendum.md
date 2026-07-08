# GlitchBoard — Stacked Commands Addendum
> Same-time cue stacks, command bundles, and graceful conflict handling  
> Version: 0.5b  
> Date: 2026-07-08

---

## Purpose

Use this addendum alongside:

- `GlitchBoard_NextPass_Timeline_Authoring.md`
- `GlitchBoard_NextPass_Timeline_Authoring_Addendum.md`

The timeline authoring pass must support stacked commands gracefully. This matters because GlitchBoard will often need multiple commands at the same musical moment, including multiple commands for the same device.

This pass remains mock UI/data-model only.

Do not implement real hardware sending, SIS, telnet, serial, Nexus, file save/load, audio, or MIDI.

---

# Core Rule

GlitchBoard must allow multiple cues at the same time on the same lane/device.

Do not assume one cue per device per timestamp.

Same-time commands should become an intentional timeline feature, not an error condition.

---

# Terminology

## Cue

A single logical action on the timeline.

Examples:

- MTPX input 3 max blue
- MGP preset 52
- TextWall display `WELCOME`
- Matrix route 1→4

## Stack

Two or more cues that share the same lane/device and exact start tick/time.

Example:

```text
Bar 4 Beat 1 — MTPX Plus #1
- input 1 max blue
- input 2 clean
- input 3 rgb split
```

## Command Bundle

A stack treated as one visual timeline object with multiple child cues/actions inside it.

A bundle may eventually send multiple commands in sequence, but for now it is mock/display only.

---

# 1. Do Not Auto-Nudge Stacked Commands

The previous addendum allowed stacking or slight visual offset. Clarification:

For this app, exact-time stacking should be allowed and preserved.

Do not automatically nudge commands forward by default.

Reason:

GlitchBoard often needs multiple changes to happen on the same beat/frame, especially during performance cues.

---

# 2. Visual Handling of Stacks

When two or more cues share the exact same device/lane and start time:

- keep them at the same logical time
- visually represent them as a stack/bundle
- show a small badge with the stack count
- allow the stack to be selected
- allow individual child cues to be inspected/edited

Example visual labels:

```text
STACK ×3
MTPX bundle
```

or compact:

```text
3 cmds
```

The stack should be visually distinct from a single cue but not ugly.

---

# 3. Stack Selection Behavior

Clicking a stacked cue object should select the stack.

The right inspector should show a Stack/Bundle summary:

- device/lane
- bar/beat/time
- number of child cues
- list of child cue labels
- effective timing offset
- mock execution order

Example:

```text
MTPX Plus #1 — Stack at Bar 4 Beat 1
3 commands

1. input 1 max blue
2. input 2 clean
3. input 3 rgb split

Execution order: top to bottom
PREVIEW ONLY — NOT SENT
```

---

# 4. Stack Popover Behavior

Double-clicking a stack should open a Stack Edit Popover.

The Stack Edit Popover should allow:

- viewing child cues
- selecting a child cue for editing
- reordering child cues using up/down controls or drag if easy
- adding another child cue to the stack
- deleting a child cue
- opening the normal device-specific editor for a child cue

For this pass, if full nested editing is too much, implement a simpler version:

- show child list
- select child cue
- button: `Edit Selected Child…`
- button: `Add Child Command…`
- button: `Remove Child`

---

# 5. Stack Context Menu

Right-clicking a stack should show:

- Edit Stack…
- Add Command to Stack…
- Ungroup Stack
- Duplicate Stack
- Copy Stack
- Delete Stack

For this pass, these can be mock behavior, but they should update data/log where practical.

---

# 6. Creating Cues at an Existing Timestamp

When the user creates a cue at a time where one or more cues already exist on the same device/lane:

- do not overwrite
- do not auto-nudge
- add the new cue to the stack
- if the existing cue was a single cue, convert the visual representation to a stack/bundle
- select the newly updated stack or the new child cue
- log the stack creation/update

Example log:

```text
STACK Added MTPX cue to existing stack at Bar 4 Beat 1 — stack now has 3 commands
```

---

# 7. Copy/Paste Stacks

Copying a stack should copy all child cues.

Pasting a stack should:

- paste all child cues at the target time
- preserve child cue order
- preserve same-device requirement for now
- show/log a warning if pasted into a different device lane

Example log:

```text
STACK Pasted MTPX stack with 3 commands at Bar 6 Beat 1
WARNING Cannot paste MTPX stack into TextWall lane
```

---

# 8. Execution Order Model

Even though no real commands are sent yet, the mock data model should record stack order.

Each stack should have a stable ordered child list.

Execution order default:

1. order children by explicit stack order
2. if no explicit order exists, order by creation order

The UI should display:

```text
Execution order: top to bottom
```

This matters later when real hardware sending is added.

---

# 9. Conflict Warning Model

Stacking is allowed, but some stacks may contain potentially conflicting commands.

For this pass, add simple mock warnings only.

Examples:

## Same device, same parameter conflict

Two MTPX child cues both change the same input/channel at the same time.

Example:

```text
input 3 max blue
input 3 clean
```

Show warning:

```text
WARNING: two commands modify input 3 at the same time
```

## Same MGP, multiple preset recalls

Two MGP preset recalls at the same exact time may conflict.

Show warning:

```text
WARNING: multiple MGP preset recalls at same time — order matters
```

Do not block the user. Just warn.

---

# 10. Stack Badge / Timeline Display Examples

MTPX stack:

```text
MTPX stack ×3
input 1 max blue
```

MGP stack:

```text
MGP stack ×2
preset 52 + random
```

TextWall stack:

```text
Text stack ×2
clear + welcome
```

Timeline should stay compact. Use tooltip/inspector/popover for detail.

---

# 11. Acceptance Criteria

This stacked command handling is successful when:

- app compiles
- app launches cleanly
- existing timeline authoring behavior still works
- creating a cue at the exact same lane/time does not overwrite existing cues
- stacked cues preserve exact same timestamp
- stacked cues show a visible stack/count badge
- clicking a stack selects it
- inspector shows stack summary and child cue list
- stack child cues preserve explicit or creation order
- stack creation/update logs clearly
- copy/paste stack works for same device lane
- cross-device stack paste is disabled/warning-only
- simple conflict warnings appear for obvious same-parameter conflicts
- no real hardware/SIS commands are sent

---

# Important

Do not turn this into a full backend scheduler yet.

This is a timeline UI/data-model feature only.

The goal is to make same-time multi-command moments visible, editable, and safe to reason about before real command execution exists.
