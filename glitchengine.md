# GlitchBoard — Engine Specification (Swift)

> Joebot Ecosystem — The Runtime Layer
> Companion to `GlitchBoard_Spec.md` (v1.3)
> Document version 0.1 — June 2026
> Scope: the clock, the cue→event compiler, the scheduler, and the output path.
> This document defines the **engine only**. The UI spec stays as-is.
> 🦖 Joebot Ecosystem

-----

## Why This Document Exists

`GlitchBoard_Spec.md` is a complete and excellent specification of the **authoring UI** —
every cue editor, tooltip, dialog, and lane is drawn out. What it does **not** define is the
part that actually fires cues in time with music: the engine.

Every “partially working” build to date has been a build where the UI was right and the
agent was left free to invent the timing layer — so it grabbed a naive timer that drifts.
Python couldn’t hold timing stable. This spec locks the engine so that never happens again.

**Rule for any agent building from this:** the engine defined here is the foundation. Build
it first, prove it with the diagnostic harness in §11, and do **not** start any Phase 3 UI
work until the harness passes its acceptance criteria.

-----

## 1. Five Principles (non-negotiable)

1. **The audio render clock is the only source of truth.** Song position is always *read*
   from the audio engine. It is never tracked by a separate timer.
1. **Never accumulate time. Always sample absolute position.** Drift is caused by adding up
   intervals. We never add. Each tick re-reads “where are we now” from the audio clock. This
   makes cumulative drift mathematically impossible.
1. **Canonical time is the sample frame (`Int64`).** Bars, beats, and seconds are *derived
   views*. All internal scheduling math is integer frame math — exact, no float drift.
1. **Every cue compiles down to one flat primitive: `ScheduledEvent`.** A one-shot, a ramp,
   a 10Hz scatter, a sprinkler — all expand at load time into a sorted list of timed events.
   The runtime knows exactly one concept.
1. **The clock never blocks on I/O.** Firing an event hands it to an output queue and returns
   immediately. Network/serial latency to Extron hardware never stalls the scheduler.

If a build violates any of these five, it’s wrong regardless of how good it looks.

-----

## 2. Canonical Time Model

Everything internal is measured in **sample frames** at the loaded song’s sample rate
(`sampleRate`, typically 44100 or 48000).

```
seconds  = frames / sampleRate
frames   = Int64(seconds * sampleRate)
```

### Beat grid → frames (resolved at compile time)

Cues are authored in bar/beat. The compiler resolves them to absolute frame positions once,
at load. The runtime never sees a bar or a beat.

```
framesPerBeat = (60.0 / bpm) * sampleRate
frame(forBar bar: Int, beat: Double) =
    gridOriginFrame + Int64((Double((bar - 1)) * beatsPerBar + (beat - 1)) * framesPerBeat)
```

For variable tempo (Phase 6), replace the constant `framesPerBeat` with a precomputed
**tempo map**: an array of `(beatIndex, frame)` anchors. The resolver becomes a lookup +
interpolation. The runtime is unaffected — it still only sees frames.

-----

## 3. Data Model — `Cue` vs `ScheduledEvent`

This split is the heart of the engine. It is the thing that has been missing.

### `Cue` — the authored object (what the UI edits)

Rich, type-specific, lives on the timeline. This is what your existing spec already describes
visually. One representative shape:

```swift
struct Cue: Identifiable, Codable {
    let id: UUID
    var lane: DeviceRef          // which device/lane
    var startBeat: BeatPosition  // authored position (bar/beat)
    var kind: CueKind            // .oneShot, .ramp, .strobe, .scatter, .snake, .random, ...
    var params: CueParams        // type-specific payload
    var isMuted: Bool
    var offlineBehavior: OfflineBehavior  // .skip / .queue
}
```

### `ScheduledEvent` — the compiled primitive (what the runtime runs)

Flat. Dumb. Sorted by frame. The runtime deals **only** in these.

```swift
struct ScheduledEvent {
    let frame: AVAudioFramePosition   // absolute song frame — canonical time
    let device: DeviceRef
    let action: DeviceAction          // already resolved to a concrete command intent
    let coalesceKey: CoalesceKey?     // for catch-up dedup (see §7)
    let sourceCueID: UUID             // for diagnostics / "last fired" UI
}
```

### The Compiler

```swift
protocol CueCompiler {
    /// Expand all authored cues into a flat, sorted [ScheduledEvent].
    /// Called once on load, and again on any edit (then swapped atomically — §9).
    func compile(_ cues: [Cue], grid: BeatGrid) -> [ScheduledEvent]
}
```

Expansion rules per cue kind:

|Cue kind         |Expands to                                                               |
|-----------------|-------------------------------------------------------------------------|
|One-shot         |1 event at its frame                                                     |
|Ramp 0→255       |N stepped events at `rampStepRate` (default 60/sec) across the span      |
|Sprinkler 1/8    |1 event per 1/8 division across the span                                 |
|Strobe           |alternating A/B events at the division rate                              |
|Ramp strobe      |events at increasing frequency, **hard-capped at 15 Hz** (photosensitive)|
|Checkerboard     |A/B group events at the division rate                                    |
|Snake / sequence |1 event per cell, in path order, spaced by per-cell duration             |
|Random           |1 event per division; the *value* is drawn at compile time + a seed      |
|TextWall one-shot|1 event carrying text + display config                                   |
|TextWall scatter |1 event per (instance × frame) at the configured Hz across the span      |


> **Why pre-expand ramps into steps instead of a live generator?** Uniformity. One runtime
> concept (timed events) is easier to make correct than two (events + tickers). 60 steps/sec
> is plenty smooth for hardware params and trivially cheap. Optimize to a generator later
> *only* if profiling demands it.

> **Random with a seed:** draw values at compile time from a seeded RNG stored on the cue.
> This makes performances **reproducible** (a Core Philosophy requirement) — the same setlist
> replays identically, including its “random” hits.

-----

## 4. The Clock

`AVAudioEngine` + `AVAudioPlayerNode`. Song position is derived from the render clock.

```swift
final class AudioClock {
    private let engine = AVAudioEngine()
    private let player = AVAudioPlayerNode()
    private var file: AVAudioFile!
    private var seekStartFrame: AVAudioFramePosition = 0   // offset after a seek
    private(set) var sampleRate: Double = 48000

    func load(url: URL) throws {
        file = try AVAudioFile(forReading: url)
        sampleRate = file.processingFormat.sampleRate
        engine.attach(player)
        engine.connect(player, to: engine.mainMixerNode, format: file.processingFormat)
        try engine.start()
    }

    func play(fromFrame start: AVAudioFramePosition = 0) {
        player.stop()
        seekStartFrame = start
        let count = AVAudioFrameCount(file.length - start)
        player.scheduleSegment(file, startingFrame: start, frameCount: count, at: nil)
        player.play()
    }

    /// THE source of truth. Absolute song position in frames, read from the render clock.
    func currentSongFrame() -> AVAudioFramePosition? {
        guard let nodeTime = player.lastRenderTime,
              let playerTime = player.playerTime(forNodeTime: nodeTime) else { return nil }
        return seekStartFrame + playerTime.sampleTime
    }

    func pause() { player.pause() }
    func stop()  { player.stop() }
}
```

That `currentSongFrame()` is the entire anti-drift strategy. Nothing else tracks time.

-----

## 5. The Scheduler (tick loop)

A dedicated high-priority timer wakes ~every 5 ms, asks the clock for the current frame, and
fires every event between the last frame and now. It is a **pump**, not a clock — its wake-up
jitter doesn’t matter because it always re-reads absolute position.

```swift
final class Scheduler {
    private let clock: AudioClock
    private let dispatcher: CueDispatcher
    private let queue = DispatchQueue(label: "glitchboard.scheduler", qos: .userInteractive)
    private var timer: DispatchSourceTimer?

    private var events: [ScheduledEvent] = []     // immutable snapshot during playback
    private var cursor = 0                         // index of next un-fired event
    private var lastFrame: AVAudioFramePosition = 0

    let positionFrame = ManagedAtomic<Int64>(0)    // read by UI for playhead

    func loadEvents(_ compiled: [ScheduledEvent]) { events = compiled; cursor = 0 }

    func start() {
        let t = DispatchSource.makeTimerSource(queue: queue)
        t.schedule(deadline: .now(), repeating: .milliseconds(5), leeway: .milliseconds(1))
        t.setEventHandler { [weak self] in self?.tick() }
        timer = t; t.resume()
    }

    private func tick() {
        guard let now = clock.currentSongFrame() else { return }
        positionFrame.store(Int64(now), ordering: .relaxed)

        // Fire everything that has come due since last tick. Binary-search the cursor on seek.
        while cursor < events.count && events[cursor].frame <= now {
            dispatcher.enqueue(events[cursor])   // hands off, returns immediately
            cursor += 1
        }
        lastFrame = now
    }
}
```

### Granularity & accuracy

A 5 ms tick means worst-case dispatch granularity of ~5 ms, plus whatever the output path
adds (§6). For a 10 Hz scatter (100 ms spacing) that’s 5% jitter; for on-beat hits at 140 BPM
(~428 ms/beat) it’s ~1%. Both are well below human rhythmic perception (~20–30 ms). Tighten
the tick to 2–3 ms if you ever need sub-frame transient precision; it costs almost nothing.

> **Optional: lookahead scheduling.** If you later want sub-tick precision, switch to the
> classic lookahead model — each tick, schedule events in `[now, now + 100ms]` with explicit
> target timestamps the output layer honors. Not needed for v1; the simple due-firing model
> above is robust and easy to reason about. Don’t add it until you’ve proven you need it.

-----

## 6. The Output Path (never blocks the clock)

Events leave the scheduler through `CueDispatcher`. The dispatcher pushes onto **per-device**
queues; worker tasks drain them and do the actual Nexus / SIS / HTTP I/O off the timing thread.

```swift
protocol CueDispatcher {
    func enqueue(_ event: ScheduledEvent)
}
```

Requirements for the real dispatcher:

- **Per-device serial queue.** Commands to one device are ordered; different devices run in
  parallel. A slow MTPX never holds up the DirtyMixer.
- **Rate limiting (token bucket per device).** Hardware has a ceiling on commands/sec. The
  bucket caps throughput so a 10 Hz scatter can’t flood a device that only accepts, say, 8/sec.
- **Coalescing on `coalesceKey`.** If the playhead jumps (UI hitch, GC) and several events for
  the same device parameter are now overdue, fire only the **latest** value, drop the
  intermediates. This is what keeps “I fell behind” from turning into “every command fires in
  a delayed burst.” Critical for ramps especially.
- **Output latency compensation (optional).** If a device has a known fixed dispatch latency
  `L`, the compiler can shift its events earlier by `L` frames so they *land* on time.
- **Offline handling.** Honor `Cue.offlineBehavior`: `.skip` drops the event and logs it;
  `.queue` holds it for when the device returns. Offline devices never throw or stall (Graceful
  philosophy).

```swift
// The proving-ground dispatcher: logs target vs actual dispatch time. No hardware.
final class LoggingDispatcher: CueDispatcher {
    func enqueue(_ e: ScheduledEvent) {
        let actual = clock.currentSongFrame() ?? e.frame
        let deltaMs = Double(actual - e.frame) / clock.sampleRate * 1000
        log("FIRE cue=\(e.sourceCueID) dev=\(e.device) target=\(e.frame) Δ=\(deltaMs)ms")
    }
}
```

Inject the dispatcher (`LoggingDispatcher` for tests, `NexusDispatcher` for real). The engine
is hardware-agnostic and fully testable without a single Extron box plugged in.

-----

## 7. Catch-Up & Missed-Event Policy

When the playhead advances past multiple overdue events in one tick, behavior depends on event
character — define this explicitly, because ambiguity here is what makes a build “feel broken”:

|Event character                                 |Policy                                      |
|------------------------------------------------|--------------------------------------------|
|State-setting (skew value, preset, ramp step)   |Coalesce by `coalesceKey` → fire latest only|
|Idempotent fire (textwall display)              |Coalesce → fire latest only                 |
|Discrete percussive (strobe edge, sprinkler hit)|If <1 division late: fire. If more: skip.   |

`coalesceKey` is typically `(device, parameterID)`. Ramp steps for the same param share a key,
so a stall makes the value jump to “now” instead of replaying the whole ramp late.

-----

## 8. Transport

Each transport action maps cleanly onto the model:

- **Play** — `clock.play(fromFrame:)`; binary-search `cursor` to first event `>= startFrame`.
- **Pause** — `clock.pause()`; timer keeps running but `currentSongFrame` stops advancing, so
  nothing new fires. (Or suspend the timer; either is fine.)
- **Stop** — `clock.stop()`; reset `cursor = 0`, `lastFrame = 0`.
- **Seek** — `clock.play(fromFrame: target)`; binary-search `cursor`; reset any stateful cue
  runtime (none, if everything is pre-expanded — another payoff of §3).
- **Loop region** — when `currentSongFrame` wraps the loop end, re-seek to loop start and
  binary-search the cursor back. Because events are immutable and stateless, looping is just
  “move the cursor.”

-----

## 9. Threading & Concurrency

|Thread / context                    |Owns               |Rule                                       |
|------------------------------------|-------------------|-------------------------------------------|
|Audio render thread                 |`AVAudioEngine`    |Untouchable. We only *read* its clock.     |
|Scheduler queue (`.userInteractive`)|tick loop, cursor  |Never does I/O. Never touches UIKit/AppKit.|
|Output workers (per device)         |network/serial I/O |Drains queues; latency lives here.         |
|Main thread                         |UI, playhead render|Reads `positionFrame` atomic. No logic.    |

**Editing during playback:** never mutate the live `events` array. The compiler builds a new
array; swap it under a lock held only for the pointer swap (microseconds), then re-seek the
cursor. The tick loop reads an immutable snapshot, lock-free in the hot path.

```swift
private let swapLock = OSAllocatedUnfairLock()
func replaceEvents(_ new: [ScheduledEvent], at frame: AVAudioFramePosition) {
    swapLock.withLock {
        events = new
        cursor = new.partitionPoint { $0.frame < frame }  // binary search
    }
}
```

Use Swift’s `Synchronization` / `Atomics` for `positionFrame`. Keep the hot path allocation-free.

-----

## 10. Module / Type Layout

A suggested file structure so the engine is a clean, testable Swift package separate from UI:

```
GlitchBoardEngine/            (Swift package — no AppKit/SwiftUI imports)
├── Time/
│   ├── BeatGrid.swift        // bpm, tempo map, bar/beat ⇄ frame
│   └── FramePosition.swift
├── Model/
│   ├── Cue.swift             // authored object
│   ├── ScheduledEvent.swift  // compiled primitive
│   └── DeviceAction.swift
├── Compile/
│   └── CueCompiler.swift     // Cue[] → ScheduledEvent[]  (one expander per CueKind)
├── Clock/
│   └── AudioClock.swift      // AVAudioEngine wrapper, currentSongFrame()
├── Schedule/
│   └── Scheduler.swift       // the 5ms tick loop
├── Output/
│   ├── CueDispatcher.swift   // protocol
│   ├── LoggingDispatcher.swift
│   └── (NexusDispatcher.swift lives in the app, conforms to the protocol)
└── Diagnostics/
    └── TimingHarness.swift   // §11
```

The app target imports `GlitchBoardEngine` and provides `NexusDispatcher`. The engine has zero
UI dependencies, so it builds and tests headless.

-----

## 11. Diagnostics & Acceptance Criteria — **build and pass this first**

This is the “prove the engine with ugly output” step. Before any Phase 3 UI, build a headless
harness:

**The click-track test.** Generate a cue list that fires one event on every beat of a known
song for its full length. Run with `LoggingDispatcher`. Record `Δ = actualFrame − targetFrame`
for every event.

### Acceptance criteria (definition of done for the engine)

1. **Dispatch jitter:** 95th-percentile `|Δ|` ≤ 5 ms; max `|Δ|` ≤ 10 ms under normal load.
1. **Zero cumulative drift:** the mean `Δ` of the *last* 100 events equals the mean of the
   *first* 100 (within noise). Event #5000 is as accurate as event #1.
1. **Burst integrity:** a sustained 10 events/sec/device cue list fires every event, in order,
   with no queue overflow and no clock stall.
1. **Seek correctness:** seek to an arbitrary frame; the next event that fires is the correct
   one for that position.
1. **Catch-up correctness:** inject a 200 ms artificial stall; verify state events coalesce to
   latest and discrete events follow the §7 policy — no delayed burst.
1. **Reproducibility:** running the same seeded setlist twice produces identical fire logs.

Ship a `Timing Diagnostics` panel in the app (even hidden behind a debug flag) that shows live
`Δ` per fired cue. You’ll want it forever — it’s how you’ll know instantly if a future change
hurts timing.

-----

## 12. How This Slots Into `GlitchBoard_Spec.md`

- The UI spec is unchanged. Cue editors still produce `Cue` objects.
- Add one line to Core Philosophy: **“Sample-accurate — the audio render clock is the only
  clock; cues never drift.”**
- Phases 1–2 are marked ✅ COMPLETE. Re-audit them against §11: if the existing playback
  can’t pass the click-track test, those phases aren’t actually done — the engine is the real
  Phase 1 and should be rebuilt to this spec before Phase 3 features land on top of it.
- `.jbt` setlists already store cues; no format change needed. The compiler is a load-time
  step between “parse `.jbt`” and “press play.”

-----

## 13. Build Order for the Agent

> **Phase 0 — Engine (do this before anything else):**
> 
> 1. `AudioClock` with `currentSongFrame()`.
> 1. `ScheduledEvent` + `CueCompiler` for one-shot and ramp only.
> 1. `Scheduler` 5 ms tick loop.
> 1. `LoggingDispatcher` + `TimingHarness` click-track test.
> 1. **Pass all §11 acceptance criteria.** Do not proceed until green.
> 1. Add remaining cue-kind expanders (strobe, sprinkler, scatter, snake, random) one at a
>    time, re-running the harness after each.
> 1. `NexusDispatcher` with per-device queues, rate limiting, coalescing.
> 
> Only then return to `GlitchBoard_Spec.md` Phase 3 and build the UI on top of an engine that
> is already proven correct.

### Codex / Claude Code kickoff prompt

> “I’m building the GlitchBoard engine: the audio-synchronized cue runtime for a SwiftUI macOS
> show-control app. Build it as a headless Swift package `GlitchBoardEngine` with no UI
> dependencies, following `GlitchBoard_Engine_Spec.md`. Start with Phase 0 only: `AudioClock`
> (AVAudioEngine + AVAudioPlayerNode, song position read from the render clock — never a
> separate timer), `ScheduledEvent` + a `CueCompiler` that handles one-shot and ramp cues, a
> 5 ms `Scheduler` tick loop, a `LoggingDispatcher`, and the `TimingHarness` click-track test.
> Canonical time is sample frames (Int64). Then prove it passes every acceptance criterion in
> §11 before adding more cue types. Do not build any UI yet.”

-----

*GlitchBoard Engine — the layer that makes the rest of the spec actually fire on time.* *🦖*