# Session Replay — Feature Spec
> Glitch Catalog + Nexus
> GitHub: github.com/joebot94/docs
> Document version 1.0 — March 2026

---

## What It Is

Session Replay is a timeline scrubber built into Glitch Catalog that lets you scrub through a recorded session and see exactly what every device was doing at any point in time. Drag the slider to any moment and DirtyMixerApp updates to show the exact state at that timestamp. Hit Replay to Hardware and Nexus sends those states to the actual physical board.

Nobody in the glitch art world has this. This is the ultimate preservation and reproducibility feature.

---

## The UI

```
┌─ Session Replay ─────────────────────────────────┐
│                                                  │
│  Basement Burn-In — 2026-03-14                  │
│                                                  │
│  ████████████░░░░░░░░░░░░░░░░░░░░░░░  47:23    │
│  00:00                                1:42:15   │
│                                                  │
│  [◀◀] [▶] [▶▶]   Speed: [1x ▼]               │
│                                                  │
│  Current state at this moment:                  │
│  CH1: 30   CH2: 128  CH3: 194                  │
│  CH4: 236  CH5: 147  CH6: 36                   │
│  CH7: 158  CH8: 70   CH9: 156                  │
│                                                  │
│  Last event: CH3 mix changed to 194             │
│  Next event: Snapshot taken (in 14s)            │
│                                                  │
│  [ Replay to Hardware ]  [ Export This Moment ] │
└──────────────────────────────────────────────────┘
```

---

## How It Works

### Scrubbing

The event log stored in the .jbt session file contains every state change with precise timestamps. The scrubber reconstructs state at any point by:

1. Starting from initial state at session start
2. Applying all events up to the scrubber position in order
3. Displaying the resulting state in the UI

Drag slider left or right → UI updates to show exact device states at that moment. No network calls needed for scrubbing — it's all computed locally from the event log.

### Replay to Hardware

When the user hits Replay to Hardware:

1. Glitch Catalog sends current scrubber state to Nexus as a `scene_recall` message
2. Nexus fans out the state to all relevant connected apps
3. DirtyMixerApp receives its channel states and updates the physical board
4. All other connected apps restore their states simultaneously
5. Confirmation shown — "State restored to 47:23"

### Live Playback

Hit the play button and Nexus replays the event log in real time:

1. Glitch Catalog sends events to Nexus respecting original timestamps
2. Nexus fans each event out to connected apps as it would have originally
3. DirtyMixerApp sliders move in real time as they did during the original session
4. Everything plays back exactly as it happened

Speed control:
- 0.5x — slow motion playback
- 1x — real time
- 2x — double speed
- 4x — fast forward

### Export This Moment

Capture the current scrubber position as a new named preset:
- Creates a `dirtymixer_preset` .jbt from current state
- Adds it to the session's preset list
- Can be recalled instantly later

---

## The Event Log Format

Stored inside the glitch_session .jbt file:

```json
{
  "jbt_type": "glitch_session",
  "name": "Basement Burn-In",
  "presets": [ ... ],
  "event_log": {
    "session_id": "session_001",
    "session_name": "Basement Burn-In",
    "started_at": "2026-03-14T00:32:00.000Z",
    "stopped_at": "2026-03-14T02:15:00.000Z",
    "duration_seconds": 6180,
    "events": [
      {
        "timestamp": "2026-03-14T00:32:01.234Z",
        "relative_ms": 1234,
        "type": "state_update",
        "source": "dirtymixer_v1",
        "summary": "CH1 mix changed to 30",
        "payload": {
          "channels": [
            {"id": 1, "input_a": true, "input_b": true, "mix": 30}
          ]
        }
      },
      {
        "timestamp": "2026-03-14T00:32:15.123Z",
        "relative_ms": 15123,
        "type": "scene_save",
        "source": "glitch_catalog",
        "summary": "Snapshot taken — Opening Look",
        "payload": { ... }
      }
    ]
  }
}
```

The `relative_ms` field is milliseconds since session start — makes scrubbing math simple.

---

## Nexus Side

Nexus needs a `replay_session` function that:

1. Receives an event log
2. Iterates through events in order
3. Respects `relative_ms` timestamps — waits the correct amount of time between events
4. Sends each event's payload to the relevant connected clients
5. Supports speed multiplier — divide wait times by speed factor
6. Supports pause and seek — cancel current replay, jump to position, resume

```python
async def replay_session(event_log: dict, speed: float = 1.0):
    events = event_log["events"]
    start_time = asyncio.get_event_loop().time()
    
    for event in events:
        target_time = event["relative_ms"] / 1000.0 / speed
        current_time = asyncio.get_event_loop().time() - start_time
        wait_time = target_time - current_time
        
        if wait_time > 0:
            await asyncio.sleep(wait_time)
        
        await dispatch_event(event)
```

---

## Glitch Catalog Integration

The Replay panel lives in Glitch Catalog as:
- A sheet that opens when you click a session's event log
- Or a dedicated Replay tab in the session detail view

**Replay button appears on any session that has an event log attached.**

Sessions without event logs show a grayed out Replay button with tooltip:
"No event log — start recording during your next session to enable replay"

---

## Why This Is Special

Most glitch artists have zero ability to recreate a specific look. It happens, they capture it on video, and it's gone forever. Even with presets you only capture discrete snapshots.

Session Replay captures the entire arc of a session — every parameter change, every moment. You can:

- Find the exact moment that incredible look happened
- See every setting that created it
- Replay it to hardware instantly
- Capture that moment as a named preset
- Share the event log with someone else so they can replay your session on their hardware

That's not just preservation. That's a completely different relationship with the medium.

---

## Build Priority

This feature depends on:
1. ✅ Nexus event logging — log all state changes with timestamps
2. ✅ Glitch Catalog recording — start/stop recording, embed log in .jbt
3. 🔲 Replay scrubber UI — timeline slider, state reconstruction
4. 🔲 Replay to Hardware — send scrubber state to Nexus
5. 🔲 Live playback — real time event replay with speed control
6. 🔲 Export This Moment — capture scrubber position as preset

Items 1 and 2 are being built now.
Items 3-6 are the next major feature milestone.

---

## Related Documents

- `Nexus_Architecture.md` — Nexus server spec
- `JBT_Format_Spec.md` — .jbt file format
- `Observatory_BuildGuide.md` — Observatory spec

---

*Session Replay — The ultimate glitch art preservation feature*
*github.com/joebot94/docs*
*Document version 1.0 — March 2026*
