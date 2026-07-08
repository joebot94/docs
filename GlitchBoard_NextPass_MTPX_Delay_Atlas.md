# GlitchBoard — Next Pass Spec
> MTPX scope controls, delay/offset planning, and Atlas/NAS next steps  
> Version: 0.4 planning draft  
> Date: 2026-07-08

---

## Context

GlitchBoard now has:

- dark broadcast-console mock GUI
- Edit/Live modes
- collapsible Rig/Devices sidebar
- compact timeline lane headers
- smart cue objects
- cue selection, hover, double-click, and right-click context menu
- device-aware Cue Edit Popovers
- right inspector sync
- TextWall, DirtyMixer, Matrix, Atlas, MGP, and MTPX mock editors
- MGP visual preset library and 2×2 preview cards
- MTPX channel grid, RGB skew sliders/steppers, quick presets, and generated command previews
- centralized theme colors in `Theme/Theme.swift`

This next pass should stay mock-first. Do not add real hardware sending yet.

---

# Pass Goal

Improve GlitchBoard in three focused areas:

1. Add MTPX apply scope controls: single channel, selected channels, all channels.
2. Add timing/delay offset planning UI so GlitchBoard can eventually compensate for device/capture delays.
3. Reframe Atlas as a future always-running NAS service / layout brain, without implementing the full service yet.

The goal is still to make the UI and data model understand the rig in human terms before real SIS/Nexus/backend sending is added.

---

# Hard Out of Scope

Do not implement yet:

- real audio playback
- waveform analysis
- real Nexus connection
- SIS sending
- telnet
- serial
- hardware discovery
- file save/load
- MIDI
- real device control
- actual NAS deployment
- real Atlas service
- automatic delay measurement from hardware

Everything in this pass is mock UI/data model only.

---

# 1. MTPX Apply Scope Controls

Improve the existing MTPX Plus editor.

The editor currently supports one selected input channel. Add an Apply Scope selector:

- Single Channel
- Selected Channels
- All Channels

The current single-channel behavior should remain unchanged.

---

## Single Channel Mode

Behavior:

- one selected input channel
- one RGB skew value set
- command preview shows one generated SIS preview line

Example:

```text
Input: 3
RGB: 0 / 0 / 31

Generated command:
3*0*0*31*4Iseq↵
```

Timeline labels:

```text
input 3 max blue
skew 0/0/31
rgb split
```

---

## Selected Channels Mode

Behavior:

- channel grid supports multi-select
- selected channels are highlighted
- RGB skew values apply to selected channels
- command preview shows one generated line per selected channel
- if many commands exist, show the first few lines and a total count

Add quick selection buttons:

- Clear
- Odd
- Even
- First 4
- Last 4
- All

Example:

```text
Selected channels: 1, 3, 5, 7
RGB: 0 / 15 / 31

Generated preview:
1*0*15*31*4Iseq↵
3*0*15*31*4Iseq↵
5*0*15*31*4Iseq↵
7*0*15*31*4Iseq↵
4 commands total
PREVIEW ONLY — NOT SENT
```

Timeline labels:

```text
ch 1,3,5,7 rgb split
4 ch skew 0/15/31
selected ch max blue
```

---

## All Channels Mode

Behavior:

- all 16 input channels are visually marked active
- RGB skew values apply to all channels
- command preview shows first few generated lines plus `16 commands total`

Example:

```text
Apply to: All 16 inputs
RGB: 0 / 0 / 31

Generated preview:
1*0*0*31*4Iseq↵
2*0*0*31*4Iseq↵
3*0*0*31*4Iseq↵
4*0*0*31*4Iseq↵
... 16 commands total
PREVIEW ONLY — NOT SENT
```

Timeline labels:

```text
all ch max blue
all ch rgb split
all ch skew 0/0/31
```

---

## Quick Presets Respect Scope

Existing quick presets should apply to the selected scope:

- Clean
- Max Red
- Max Green
- Max Blue
- RGB Split
- Random Same
- Random Each

Definitions:

- Clean = R/G/B all 0
- Max Red = R 31, G 0, B 0
- Max Green = R 0, G 31, B 0
- Max Blue = R 0, G 0, B 31
- RGB Split = R 0, G 15, B 31
- Random Same = one random RGB value applied to the whole scope
- Random Each = each channel in scope gets its own random RGB value

Random Each is mock-only for now. It should update preview/labels clearly.

---

# 2. Timing / Delay Offset Planning UI

GlitchBoard will eventually need to compensate for different delays across devices.

Examples:

- MGP preset recall may visually land later than a TextWall update.
- Some scalers/capture paths may introduce frame delay.
- VHS/capture/HDMI/analog chains may be offset from each other.
- Some devices may need commands sent early.

Add mock UI/data model support for delay offsets, but do not measure or apply real delays yet.

---

## Global Timing Offset

Add a global timing settings section, likely in a Settings/Timing mock panel or a compact global timing popover.

Fields:

- Global command offset in ms
- Global visual compensation in frames
- Assume FPS selector:
  - 29.97
  - 59.94
  - 60.00
- Dry-run timing preview toggle

Example:

```text
Global Timing
Command Offset: -40 ms
Visual Offset: 0 frames
FPS: 59.94
Dry-run Preview: On
```

Meaning:

- negative offset = send commands early
- positive offset = send commands late

No real sending happens in this pass.

---

## Per-Device Timing Offset

Each device should eventually have its own timing offset.

Add mock fields to device settings/status detail:

- Device command offset in ms
- Device visual delay in frames
- Last measured delay placeholder
- Measurement status:
  - Unknown
  - Manual
  - Estimated
  - Measured

Example devices:

```text
MGP 464 A
Command offset: -80 ms
Visual delay: 2 frames
Measurement: Manual

TextWall
Command offset: -20 ms
Visual delay: 0 frames
Measurement: Estimated

MTPX Plus #1
Command offset: -10 ms
Visual delay: 0 frames
Measurement: Unknown
```

---

## Per-Cue Timing Offset

Each cue should be able to override timing if needed.

Add mock cue-level timing fields in the inspector/popover advanced section:

- Use device default timing: on/off
- Cue offset in ms
- Fire early/late indicator

Example:

```text
Timing
Use device default: On
Device offset: -80 ms
Cue override: 0 ms
Effective fire offset: -80 ms
```

---

## Effective Fire Time Preview

For selected cues, show timing information somewhere compact in the inspector:

```text
Cue time: Bar 4 Beat 1 / 00:05.142
Device offset: -80 ms
Global offset: -40 ms
Effective mock fire time: 00:05.022
```

This is display-only for now.

---

## Delay Measurement Future Placeholder

Add a disabled/future section for delay measurement:

```text
Delay Calibration
[Measure Device Delay]  SOON
[Tap Visual Hit]        SOON
[Import Capture Test]   SOON
```

This should make it clear that delay measurement is planned but not implemented.

Possible later concept:

- send visual flash cue
- capture/observe when it appears
- compare intended cue time to observed time
- store device/path delay
- use offset compensation during playback

Do not implement that yet.

---

# 3. Atlas Rework Planning / NAS Service Placeholder

Atlas needs to be rethought as a persistent layout/routing brain.

The likely future direction:

- Atlas runs as a small service on the NAS or always-on machine.
- GlitchBoard talks to Atlas over HTTP/WebSocket/local network.
- Atlas tracks layout modes, device roles, MGP ownership of cells, routing presets, and hardware capabilities.
- GlitchBoard remains the timeline/live cockpit.

Do not implement the real service in this pass.

---

## Atlas Device Panel Update

Atlas currently appears offline. Keep it offline/mock, but make its role clearer.

Update Atlas status details to say something like:

```text
Atlas
Status: Offline / Not Connected
Role: Layout brain / routing coordinator
Future host: NAS service
Connection: not configured
```

---

## Atlas Future Settings Placeholder

Add a disabled/mock Atlas settings section:

Fields:

- Hostname/IP: `atlas.local` or NAS hostname placeholder
- Port: placeholder
- Connection mode:
  - Local app
  - NAS service
  - Manual/offline
- Protocol:
  - HTTP
  - WebSocket
- Start with system: future placeholder
- Health check: disabled placeholder

Example:

```text
Atlas Service
Host: atlas.local
Mode: NAS Service
Protocol: WebSocket
Status: Not Connected
[Check Connection] SOON
```

---

## Atlas Layout Brain Placeholder

Add a mock/future explanation panel:

Atlas will eventually answer questions like:

- Which MGP owns each grid cell?
- Is the current wall mode 2×2, 2×3, 2×4, 3×2, or 4×2?
- Which MGP preset ranges are safe for the current layout?
- Which routes need to be active for this cue?
- What should panic restore do?

Do not implement logic yet, just document/represent the role in the UI.

---

# 4. Event Log

Add Event Log messages for new mock interactions.

Examples:

```text
[00:00.000] MTPX     Scope changed to All Channels
[00:00.000] MTPX     Selected channels 1,3,5,7
[00:00.000] MTPX     Applied Max Blue to all 16 channels
[00:00.000] TIMING   Global command offset set to -40 ms
[00:00.000] TIMING   MGP 464 A device offset set to -80 ms
[00:00.000] ATLAS    Atlas role set to NAS Service placeholder
```

Avoid log spam when dragging sliders. Log on commit/release or Apply where practical.

---

# 5. Acceptance Criteria

This pass succeeds when:

- app compiles
- app launches cleanly
- existing Edit/Live behavior still works
- existing MGP visual editor still works
- existing TextWall/DirtyMixer/Matrix/Atlas popovers are not broken
- MTPX editor has Apply Scope selector
- Single Channel mode preserves existing behavior
- Selected Channels mode supports multi-select channel grid
- Selected Channels mode has quick selection buttons
- All Channels mode visually selects/applies to all 16 inputs
- MTPX command preview generates multiple mock command lines when needed
- MTPX quick presets respect scope
- Random Same and Random Each exist as mock UI behaviors
- MTPX timeline labels regenerate compactly for multi/all-channel cues
- global timing offset mock UI exists
- per-device timing offset mock fields exist
- per-cue effective fire time preview exists
- delay measurement placeholders exist but are disabled/future
- Atlas UI is reframed as future NAS layout/routing service
- Atlas remains offline/mock unless explicitly changed
- no real hardware/audio/Nexus/SIS sending is added
- dark broadcast-console visual style is preserved

---

# Important

This is still a mock UI/data-model pass.

Do not overbuild backend architecture yet.

The goal is to make GlitchBoard understand channel scope, timing offsets, and Atlas's future role before adding real hardware control.
