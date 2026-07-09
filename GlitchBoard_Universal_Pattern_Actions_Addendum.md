# GlitchBoard — Universal Pattern Actions Addendum
> Multi-action patterns, per-beat steps, random choices, anchor/return behavior, and pattern support for all capable devices  
> Version: 0.7 planning draft  
> Date: 2026-07-09

---

## Purpose

The first pattern/range pass proved that high-level range cues are the right abstraction.

Now GlitchBoard needs to generalize patterns beyond only:

- MGP Random Preset Pattern
- MTPX Skew Flash Pattern

The user wants pattern mode to eventually work for everything that makes sense:

- MGP preset patterns
- IPCP LED strobe/random/color patterns
- IPCP VHS/relay/wall mode patterns where useful
- TextWall lyric/message/grid patterns
- DirtyMixer preset/random patterns
- Matrix/route patterns if useful
- generic API/device patterns later

The key idea:

A pattern should be a rule over time, not a pile of duplicated cues.

Patterns should support per-beat actions, random pools, momentary flashes, and return-to-base behavior.

Example user intents:

```text
Bars 14–25, beat 3: IPCP LED random strobe/color pulse.
```

```text
Bars 3–25: MGP preset 48 on beat 1, random presets on beats 2/3/4.
```

```text
MGP holds preset 48 as the base state. On beat 3, quickly flash to preset 54, then return to 48.
```

```text
Every bar, beat 1 forces clean preset 48, other beats are random chaos, so the system always lands back on a known safe layout.
```

---

# Core Concept

Add a more universal pattern action system.

A pattern has:

- a lane/device
- a bar range
- one or more beat/action rules
- optional random choices
- optional momentary flash behavior
- optional return/base action
- occurrence preview
- advisory conflict warnings

Patterns remain one editable object unless explicitly baked/expanded later.

---

# 1. Pattern Types

The app should support these conceptual pattern kinds:

## Single-action repeating pattern

One action repeats over a range according to beat filter.

Example:

```text
IPCP LED Red on beat 3, bars 14–25
```

## Random-action repeating pattern

At each occurrence, pick one action/value from a pool.

Example:

```text
MGP random preset 48–71 on beat 3
```

```text
IPCP LED random color on beats 2+4
```

## Multi-beat step pattern

Different beats in each bar perform different actions.

Example:

```text
Beat 1: MGP preset 48
Beat 2: random 48–71
Beat 3: random 48–71
Beat 4: random 48–71
```

## Momentary flash pattern

Trigger an action, hold it briefly, then return.

Example:

```text
Beat 3: MGP preset 54 for 150ms, then return to preset 48
```

```text
Beat 3: IPCP LED white flash for 100ms, then return to previous/base color
```

## Anchor/base-return pattern

A known base action happens at a defined beat, and chaos/random/flash behavior occurs around it.

Example:

```text
Beat 1: base preset 48
Beat 2: random
Beat 3: flash 54 then return 48
Beat 4: random
```

---

# 2. Pattern Rule Model

A universal pattern can be represented as a list of rules.

Suggested model:

```text
PatternCue
  deviceId
  laneId
  startBar
  endBar
  rules: [PatternRule]
  previewMode
  warningState
```

Each rule:

```text
PatternRule
  ruleId
  name
  beatSelector
  actionMode
  actionPayload
  randomPool
  momentary
  holdMs
  returnBehavior
  enabled
```

Beat selector examples:

```text
Every beat
Beat 1
Beat 2
Beat 3
Beat 4
Beats 1+3
Beats 2+4
Custom list: 1,3
```

Action modes:

```text
Fixed Action
Random From Range
Random From List
Random From Tagged Pool
Step Sequence
Momentary Flash
Return/Base Action
```

Return behavior:

```text
No return
Return to previous value
Return to base action
Return to fixed action
Return to clean/default
```

---

# 3. Minimum Useful Next Implementation

Do not try to implement the entire universal system at once.

The next practical pass should add a limited but powerful system:

## A. MGP Multi-Beat Pattern

Support multiple beat rules in one MGP pattern.

Example default templates:

### MGP Anchor Random Pattern

```text
Bars 3–25
Beat 1: preset 48
Beats 2,3,4: random presets 48–71
Avoid immediate repeats: On
Allow unknown: Off
```

Label:

```text
MGP anchor 48 + rnd · bars 3–25
```

Preview:

```text
3.1 preset 48
3.2 random 48–71
3.3 random 48–71
3.4 random 48–71
4.1 preset 48
...
```

### MGP Flash Return Pattern

```text
Bars 3–25
Beat 1: preset 48
Beat 3: flash preset 54 for 150ms, then return preset 48
```

Label:

```text
MGP flash 54→48 · b3 · 150ms
```

Preview:

```text
3.1 preset 48
3.3 preset 54 → +150ms preset 48
4.1 preset 48
4.3 preset 54 → +150ms preset 48
```

## B. IPCP LED Pattern

Support pattern/range behavior for IPCP LED/RGB Strip targets.

Example templates:

### LED Random Color Beat Pattern

```text
Bars 14–25
Beat 3: random color from selected palette
Hold/return optional
```

Label:

```text
LED random color · bars 14–25 · b3
```

Preview:

```text
14.3 random color
15.3 random color
16.3 random color
...
```

### LED Strobe/Flash Pattern

```text
Bars 14–25
Beat 3: white flash 100ms then return previous/base/off
```

Label:

```text
LED flash white · b3 · 100ms
```

Preview:

```text
14.3 WHITE → +100ms return
15.3 WHITE → +100ms return
```

---

# 4. UI Concepts

## Pattern editor should gain a Rules section

Instead of only one beat filter/action payload, advanced patterns can show a rule list:

```text
RULES
1. Beat 1 — MGP preset 48
2. Beats 2,3,4 — Random MGP presets 48–71
3. Beat 3 — Flash preset 54 for 150ms → return 48
```

Controls:

- Add Rule
- Duplicate Rule
- Delete Rule
- Enable/disable rule
- Reorder rules if useful later

For the first pass, templates may be enough:

- MGP Simple Random
- MGP Anchor + Random
- MGP Flash Return
- LED Random Color
- LED Flash/Strobe

Do not overbuild a full node/rule editor yet if it risks destabilizing the app.

---

# 5. Pattern Template Picker

Right-click menu should eventually support:

## MGP lane

```text
Add MGP Pattern...
  Random Preset Pattern
  Anchor 48 + Random Other Beats
  Flash 54 → Return 48
```

## IPCP lane / RGB Strip target

```text
Add IPCP Pattern...
  RGB Strip Random Color Pattern
  RGB Strip Flash/Strobe Pattern
```

Other IPCP targets can remain single cues for now.

---

# 6. Random Choices

Random patterns should be explicit about the random pool.

MGP random sources:

- preset range
- preset list
- tagged preset pool

LED random sources:

- selected color palette
- all colors
- warm colors
- cold colors
- custom chosen swatches

Random behavior:

- avoid immediate repeats
- seeded preview optional later
- preview should show conceptual `random`, not fake exact choices unless seed support exists

---

# 7. Base / Return Behavior

Base/return behavior is essential.

Examples:

```text
Base: MGP preset 48
Flash: preset 54 for 150ms
Return: preset 48
```

```text
Base: LED Off
Flash: White for 100ms
Return: Off
```

```text
Base: Previous value
Flash: Red for 100ms
Return: Previous value
```

The UI should make this visible so the user understands what happens after a momentary action.

---

# 8. Timeline Visuals

Universal patterns should still render as ribbons.

Possible visual language:

```text
RND = random pattern
STEP = multi-rule/beat pattern
FLASH = momentary flash pattern
ANCHOR = base/return pattern
```

Labels:

```text
MGP anchor 48 + rnd · bars 3–25
MGP flash 54→48 · b3 · 150ms
LED rnd color · bars 14–25 · b3
LED strobe white · b3 · 100ms
```

Occurrence ticks may indicate rule types if practical:

- base/anchor tick
- random tick
- flash tick

If that is too much, keep ticks simple and show detail in inspector/editor.

---

# 9. Conflict Handling

Patterns can overlap single cues and stacks. This should warn, not block.

Same-device generated occurrences that collide with existing cues should show advisory warnings.

Within a multi-rule pattern, if two rules hit the same device at the same exact beat, order matters.

Example:

```text
Beat 3: random preset
Beat 3: flash preset 54 then return 48
```

This is valid only if the user intentionally ordered it. The preview should show rule order.

---

# 10. Do Not Bake By Default

Do not expand universal patterns into many visible cues by default.

A future feature can offer:

```text
Expand/Bake Pattern to Cues...
```

But it should remain disabled or marked `SOON` until intentionally built.

---

# 11. Acceptance Criteria For A Future Universal Pattern Pass

A future pass succeeds when:

- app compiles
- existing patterns still work
- MGP simple random pattern still works
- MTPX flash pattern still works
- IPCP lane still works
- Rig Config still works
- user can create an MGP Anchor + Random pattern
- user can create an MGP Flash Return pattern
- user can create an IPCP LED Random Color pattern
- user can create an IPCP LED Flash/Strobe pattern
- pattern editor shows a rule/template structure
- occurrence preview shows per-beat actions and returns
- pattern remains one object, not many cues
- overlap warnings remain advisory
- no real commands are sent
- no backend/Nexus integration is added yet

---

# Important

This is about making pattern mode universal and musically expressive.

Do not implement every possible device at once.

Start with:

```text
MGP multi-beat patterns
IPCP LED random/flash patterns
```

Then extend the same rule system later to TextWall, DirtyMixer, Matrix, and generic devices.
