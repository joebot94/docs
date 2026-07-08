# GlitchBoard — Device Configuration / Settings Addendum
> Global config page, device connection settings, credentials, sockets/threads, IPCP config, and safe connection testing  
> Version: 0.6c planning draft  
> Date: 2026-07-08

---

## Purpose

GlitchBoard needs a real configuration/settings area before it can safely control hardware.

The app already has a strong timeline/control-surface concept, but real devices need configurable network addresses, ports, credentials, connection limits, transport types, and safety/dry-run settings.

This document captures the future config-page concept.

This is a future implementation spec. Do not implement during the current timeline authoring pass unless explicitly requested.

---

# Core Concept

GlitchBoard should have a dedicated Settings / Configuration page for the whole rig.

This page should let the user define:

- devices
- connection types
- IP addresses / hostnames / domains
- ports
- usernames/passwords or tokens when applicable
- serial ports and baud rates
- IPCP endpoints and sub-targets
- socket/thread/concurrency behavior
- timing offsets
- safety/dry-run behavior
- test connection actions

The timeline should use these configured devices instead of hardcoded assumptions.

---

# 1. Settings Page Structure

Suggested top-level settings sections:

```text
General
Rig / Devices
Connections
Credentials
Timing
IPCP Control Hub
Generic Devices
Safety / Dry Run
Advanced
```

Alternative sidebar-style layout:

```text
Settings
  General
  Devices
  Network
  Serial
  IPCP
  Timing
  Safety
  Advanced
```

The visual style should match GlitchBoard: dark technical control-panel UI, not a plain macOS preferences sheet if avoidable.

---

# 2. Device Configuration

Each device should have a configuration object.

Common fields:

- enabled / disabled
- display name
- device type
- connection type
- host / IP / domain
- port
- username, if applicable
- password / token, if applicable
- connection timeout
- command timeout
- retry count
- device timing offset
- dry-run only toggle
- notes

Example:

```text
Device: Matrix 12800
Type: Extron Matrix
Connection: Telnet / SIS
Host: mx.extron.video
IP: 10.0.0.12
Port: 23
Username: admin
Password: stored securely
Max sockets: 1
Command queue: serial
Dry-run only: Off
```

Example:

```text
Device: MGP 464 A
Type: Extron MGP
Connection: Telnet / SIS
Host: mgp1.extron.video
IP: 10.0.0.61
Port: 23
Auth: none
Max sockets: 1
Command queue: serial
Dry-run only: On until tested
```

---

# 3. Connection Types

Supported or future connection types:

- Mock only
- TCP raw socket
- Telnet / SIS
- Serial / RS-232
- HTTP
- WebSocket
- OSC, future
- MIDI, future
- IPCP transport
- Local app / helper service

Each connection type should expose only relevant fields.

Example:

## TCP / Telnet

- host
- port
- username/password if needed
- line ending
- login prompt behavior
- command timeout
- reconnect behavior

## Serial

- serial port path
- baud rate
- data bits
- parity
- stop bits
- flow control
- line ending

## HTTP

- base URL
- method
- headers
- token/auth

## IPCP

- IPCP device
- physical port/target
- command library

---

# 4. Credentials And Secrets

Passwords/tokens should not be casually stored in plain project files.

Future implementation should store secrets using macOS Keychain when possible.

For mock UI, show placeholders only.

Rules:

- hide passwords by default
- provide reveal button
- provide test connection without exposing password
- do not log passwords/tokens
- do not put secrets in Event Log
- do not export secrets into shared project files unless explicitly requested and encrypted/marked unsafe

Example UI:

```text
Username: admin
Password: ••••••••••  [Reveal] [Store in Keychain]
```

---

# 5. Sockets / Threads / Command Queues

Some devices should only use one connection/command stream at a time. Others may support multiple sockets or parallel requests.

The config page should expose this carefully.

Use user-facing terms first, advanced terms second.

Suggested UI label:

```text
Command Streams
```

Advanced tooltip:

```text
Number of simultaneous sockets/worker queues allowed for this device.
```

Fields:

- command mode:
  - Serial queue / one at a time
  - Parallel allowed
  - Rate limited
- max command streams / sockets
- minimum delay between commands
- max commands per second
- require ACK before next command
- reconnect on failure
- keep connection alive

Examples:

```text
MGP 464 A
Command mode: Serial queue
Max streams: 1
Min delay: 50 ms
Require ACK: On
```

```text
Generic HTTP Device
Command mode: Parallel allowed
Max streams: 4
Rate limit: 20 commands/sec
```

```text
Matrix 12800
Command mode: Serial queue
Max streams: 1
Require ACK: On
```

---

# 6. IPCP Control Hub Configuration

The IPCP config section should define the IPCP itself and its named sub-targets.

## IPCP Device

Fields:

- host / IP / domain
- port
- username/password if applicable
- connection type
- command queue settings
- enabled/disabled
- test connection

## IPCP Sub-Targets

Example:

```text
IR Port 1 — Projector
IR Port 2 — RGB Strip
IR Port 3 — VHS Deck
IR Port 4 — Video Wall Controller
Serial Port 1 — Scaler
Relay 1 — Lamp
Digital Out 1 — Trigger
```

Each sub-target should have:

- name
- port/type
- command library
- optional color/accent
- optional linked device identity
- notes

For the video wall controller, the user-facing device can be separate while using IPCP as transport:

```text
Device: Video Wall Controller
Transport: IPCP IR
IPCP Target: IR Port 4
Commands: 2×2, 2×3, 3×2, 3×3, 3×4, 4×4
```

---

# 7. Generic Device Configuration

For a possible public/generic version of GlitchBoard, users should be able to create their own generic command devices.

Generic device fields:

- device name
- connection type
- host/port or serial settings
- command list
- command parameters
- command preview
- timing offset
- queue/concurrency settings

Example:

```text
Device: Projector
Connection: TCP
Host: 192.168.1.50
Port: 23
Commands:
- Power On
- Power Off
- HDMI 1
- HDMI 2
```

Example:

```text
Device: RGB Controller
Connection: Serial
Port: /dev/ttyUSB0
Baud: 9600
Commands:
- Red
- Blue
- Off
- Random
```

---

# 8. Connection Testing

Every configured device should support mock-safe connection tests.

Buttons:

- Test Connection
- Send Safe Query
- Dry-Run Command Preview
- Reset Connection

Event Log examples:

```text
CONFIG Testing connection to MGP 464 A at 10.0.0.61:23
CONFIG MGP 464 A connection OK
WARNING Matrix 12800 login failed — credentials rejected
```

Never log passwords.

---

# 9. Safety / Dry Run

Global safety settings:

- global dry-run mode
- require arm before real sending
- panic / emergency black behavior
- confirm dangerous actions
- block power-off commands unless armed
- log all outbound commands
- show command preview before enabling real send

Device-level safety:

- dry-run only
- allow real sending
- block dangerous commands
- require confirmation

Example:

```text
Global Hardware Mode:
[Mock Only] [Dry Run] [Armed Real Send]
```

Default should be Mock Only or Dry Run.

---

# 10. Project vs Rig Configuration

Separate project/show data from rig hardware configuration.

## Project file should contain:

- cues
- patterns
- stacks
- timeline settings
- device references by ID
- mode profile references

## Rig config should contain:

- actual IP addresses
- hostnames
- ports
- credentials references
- serial port names
- physical IPCP mappings
- command libraries
- timing offsets

Reason:

A show/project should be portable without leaking passwords or hardcoded rack details.

---

# 11. UI Placement

Possible places for Settings:

- top bar gear icon
- app menu: GlitchBoard → Settings
- left rail button: CONFIG
- Live Mode status pill click

Config should probably open as a large modal/window or a dedicated page, not a tiny popover.

Suggested title:

```text
Rig Configuration
```

or:

```text
Device Config
```

---

# 12. Acceptance Criteria For A Future Config Pass

A future config pass succeeds when:

- app has a visible Settings / Rig Configuration entry point
- devices can show/edit host/IP/domain and port
- credentials are represented safely in mock UI
- connection type changes visible fields
- command stream/socket settings exist in Advanced section
- IPCP config supports named sub-targets
- video wall controller can reference IPCP IR as transport
- generic device config exists or is stubbed
- Test Connection buttons exist as mock/safe actions
- Event Log records config tests without exposing secrets
- project data is conceptually separate from rig config
- no real hardware commands are sent unless a later real-backend pass explicitly enables it

---

# Important

Do not scatter connection settings across random cue editors.

Real hardware connection details belong in one coherent Rig Configuration area.

Cue editors should reference configured devices and commands, not ask for IPs/passwords directly.
