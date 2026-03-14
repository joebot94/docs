# JBT Format Specification
> Joebot Ecosystem Shared File Format
> GitHub: github.com/joebot94/docs
> Document version 1.0 — March 2026

---

## What .jbt Is

`.jbt` is the shared file format for the entire Joebot studio ecosystem. It is a JSON-based container format where the `jbt_type` field identifies what kind of object the file contains.

Every Joebot app reads and writes `.jbt` files. Nexus uses `.jbt` for scene snapshots. Glitch Catalog stores sessions as `.jbt`. DirtyMixerApp saves presets as `.jbt`.

The format is intentionally simple — plain JSON, human readable, version tracked, extensible.

---

## Core Principles

- Every .jbt file is valid JSON
- Every .jbt file has a `jbt_type` field at the root
- Every .jbt file has a `version` field at the root
- Every .jbt file has a `created_at` timestamp at the root
- Apps only process types they understand — unknown types are ignored gracefully
- The format evolves via version field — old files remain readable

---

## Root Envelope

Every .jbt file regardless of type shares this root structure:

```json
{
  "jbt_type": "string",
  "version": "1.0",
  "created_at": "2026-03-13T00:00:00Z",
  "name": "Human readable name",
  "notes": "Optional notes",
  "payload": { ... }
}
```

| Field | Required | Description |
|---|---|---|
| `jbt_type` | Yes | Identifies the object type |
| `version` | Yes | Format version for this type |
| `created_at` | Yes | ISO 8601 timestamp |
| `name` | Recommended | Human readable name |
| `notes` | Optional | Free text notes |
| `payload` | Yes | Type-specific content |

---

## Known JBT Types

| jbt_type | App | Description |
|---|---|---|
| `dirtymixer_preset` | DirtyMixerApp | Stored board state for all 9 channels |
| `dirtymixer_timeline` | DirtyMixerApp | Keyframed automation over time |
| `dirtymixer_clip` | DirtyMixerApp | Reusable automation segment |
| `dirtymixer_project` | DirtyMixerApp | Full session — presets + timelines |
| `extron_snapshot` | Atlas | Extron device state snapshot |
| `glitch_session` | Glitch Catalog | Full studio session with scene snapshot |
| `nexus_scene` | Nexus | Scene definition — actions across devices |
| `nexus_pattern` | Nexus | Timed behavior pattern |
| `nexus_cue_bundle` | Nexus | Collection of cues for show playback |
| `textwall_state` | Text Wall | Text wall content and style state |

New types are added as new apps join the ecosystem. Existing apps ignore unknown types gracefully.

---

## Type Definitions

---

### dirtymixer_preset

A complete snapshot of all 9 dirty mixer channel states.

```json
{
  "jbt_type": "dirtymixer_preset",
  "version": "1.0",
  "created_at": "2026-03-13T00:00:00Z",
  "name": "Chaos Mode",
  "notes": "High energy all channels fighting",
  "payload": {
    "channels": [
      {
        "id": 1,
        "input_a_enabled": true,
        "input_b_enabled": true,
        "mix": 128
      },
      {
        "id": 2,
        "input_a_enabled": true,
        "input_b_enabled": false,
        "mix": 255
      }
    ],
    "transition": {
      "type": "linear",
      "duration_ms": 0
    }
  }
}
```

#### Channel Fields

| Field | Type | Description |
|---|---|---|
| `id` | int | Channel number 1–9 |
| `input_a_enabled` | bool | Input A active |
| `input_b_enabled` | bool | Input B active |
| `mix` | int | Mix value 0–255 (or 0–1023 for 10-bit) |

#### Transition Fields

| Field | Type | Description |
|---|---|---|
| `type` | string | `snap`, `linear`, `triangle`, `preset_default` |
| `duration_ms` | int | Transition duration in milliseconds |

---

### dirtymixer_timeline

Keyframed automation for dirty mixer channels over time.

```json
{
  "jbt_type": "dirtymixer_timeline",
  "version": "1.0",
  "created_at": "2026-03-13T00:00:00Z",
  "name": "10 Second Chaos Ramp",
  "payload": {
    "duration_ms": 10000,
    "tracks": [
      {
        "channel_ids": [1, 4, 6, 9],
        "parameter": "mix",
        "keyframes": [
          { "time_ms": 0, "value": 0 },
          { "time_ms": 5000, "value": 120 },
          { "time_ms": 10000, "value": 0 }
        ],
        "interpolation": "linear"
      },
      {
        "channel_ids": [7],
        "parameter": "mix",
        "mode": "random",
        "random_interval_ms": 500
      }
    ]
  }
}
```

#### Track Fields

| Field | Type | Description |
|---|---|---|
| `channel_ids` | array | Which channels this track controls |
| `parameter` | string | `mix`, `input_a_enabled`, `input_b_enabled` |
| `keyframes` | array | Time/value pairs |
| `interpolation` | string | `linear`, `step`, `triangle` |
| `mode` | string | Optional — `random` overrides keyframes |
| `random_interval_ms` | int | How often random value is chosen in random mode |

---

### dirtymixer_project

Full DirtyMixerApp session container.

```json
{
  "jbt_type": "dirtymixer_project",
  "version": "1.0",
  "created_at": "2026-03-13T00:00:00Z",
  "name": "Show A",
  "payload": {
    "presets": [ ... ],
    "timelines": [ ... ],
    "clips": [ ... ],
    "default_preset_id": "chaos_mode"
  }
}
```

---

### extron_snapshot

Extron device state snapshot for scene recall.

```json
{
  "jbt_type": "extron_snapshot",
  "version": "1.0",
  "created_at": "2026-03-13T00:00:00Z",
  "name": "3x3 Wall State",
  "payload": {
    "devices": [
      {
        "device_id": "device.matrix.main",
        "action": "recall_preset",
        "params": { "preset": 5 }
      },
      {
        "device_id": "device.dms.main",
        "action": "recall_preset",
        "params": { "preset": 3 }
      },
      {
        "device_id": "device.mgp.1",
        "action": "recall_preset",
        "params": { "preset": 2 }
      }
    ]
  }
}
```

---

### glitch_session

Full studio session — the primary Glitch Catalog record type. Contains metadata, scene snapshot, media references, and notes.

```json
{
  "jbt_type": "glitch_session",
  "version": "1.0",
  "created_at": "2026-02-25T00:00:00Z",
  "name": "First Glitch Session",
  "notes": "Autocreated example. Edit me.",
  "payload": {
    "date": "2025-02-25",
    "location": "Studio A",
    "tags": ["Datamosh", "RGB Skew"],
    "scene_snapshot": {
      "dirtymixer_preset": { ... },
      "extron_snapshot": { ... },
      "textwall_state": { ... }
    },
    "analog_masters": [
      {
        "id": "VH01S1",
        "label": "VHS — Joebot Glitches VH01S1 — 1",
        "format": "VHS",
        "tape_number": 1
      }
    ],
    "gear_chain": [
      { "label": "Extron" },
      { "label": "Panasonic WJ-AVE5" }
    ],
    "digital_captures": [
      {
        "filename": "20250225022171993.mp4",
        "duration_s": 7.2,
        "resolution": "1920x1080",
        "codec": "hevc",
        "tags": ["capture"]
      }
    ]
  }
}
```

---

### nexus_scene

A named collection of actions across devices, fired together by Nexus.

```json
{
  "jbt_type": "nexus_scene",
  "version": "1.0",
  "created_at": "2026-03-13T00:00:00Z",
  "name": "Video Wall 3x3",
  "payload": {
    "scene_id": "scene.video_wall.3x3",
    "actions": [
      {
        "device_id": "device.matrix.main",
        "action": "recall_preset",
        "params": { "preset": 5 }
      },
      {
        "device_id": "device.dms.main",
        "action": "recall_preset",
        "params": { "preset": 3 }
      },
      {
        "client_id": "dirtymixer_v1",
        "action": "recall_dirtymixer_preset",
        "params": { "preset_id": 12 }
      }
    ]
  }
}
```

---

### nexus_pattern

A timed behavior template executed by Nexus.

```json
{
  "jbt_type": "nexus_pattern",
  "version": "1.0",
  "created_at": "2026-03-13T00:00:00Z",
  "name": "Random Chaos Beats 1 and 3",
  "payload": {
    "pattern_id": "pattern.random_chaos.beats_1_3",
    "trigger": "beat",
    "trigger_beats": [1, 3],
    "duration_bars": 4,
    "targets": [
      {
        "device_id": "device.matrix.main",
        "action": "recall_preset",
        "params": { "preset_range": [1, 8] }
      }
    ]
  }
}
```

---

### textwall_state

Text wall app content and style state for scene recall.

```json
{
  "jbt_type": "textwall_state",
  "version": "1.0",
  "created_at": "2026-03-13T00:00:00Z",
  "name": "Fullscreen White",
  "payload": {
    "content": "...",
    "style": {
      "font": "...",
      "size": 120,
      "color": "#FFFFFF",
      "background": "#000000",
      "alignment": "center"
    },
    "display_mode": "fullscreen"
  }
}
```

---

## Versioning

Each type has its own version field independent of other types. Version follows semver lite:

- `1.0` — initial stable definition
- `1.1` — backwards compatible additions
- `2.0` — breaking changes

Apps should handle unknown fields gracefully — ignore rather than error. Apps should check version field and warn if they encounter a version higher than they support.

---

## File Naming Conventions

Suggested naming patterns:

| Type | Naming Pattern |
|---|---|
| `dirtymixer_preset` | `preset_{name}.jbt` |
| `dirtymixer_timeline` | `timeline_{name}.jbt` |
| `dirtymixer_project` | `project_{name}.jbt` |
| `glitch_session` | `{date}_{name}.jbt` |
| `nexus_scene` | `scene_{id}.jbt` |
| `nexus_pattern` | `pattern_{id}.jbt` |

---

## File Storage Locations

| App | Default Path |
|---|---|
| Nexus scenes | `~/.nexus/jbt/scenes/` |
| Nexus patterns | `~/.nexus/jbt/patterns/` |
| DirtyMixerApp presets | `~/JBT/dirtymixer/presets/` |
| DirtyMixerApp projects | `~/JBT/dirtymixer/projects/` |
| Glitch Catalog sessions | `~/JBT/sessions/` |

---

## Parsing Guidelines

Any app parsing .jbt files should:

1. Read `jbt_type` first — if unknown, skip gracefully
2. Check `version` — warn if higher than supported
3. Parse `payload` according to type schema
4. Ignore unknown fields rather than erroring
5. Never modify a .jbt file without updating `created_at` or adding a `modified_at` field

---

## Related Documents

- `Nexus_Architecture.md` — Nexus server spec
- `DirtyMixerApp_BuildGuide.md` — DirtyMixerApp spec
- `Ecosystem_Overview.md` — full ecosystem map (TODO)

---

*JBT Format Specification*
*github.com/joebot94/docs*
*Document version 1.0 — March 2026*
