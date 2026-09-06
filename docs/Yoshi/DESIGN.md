# Yoshi.jsfx — design notes (2026-09-06)

## Concepts (kept separate in the code)
1. **Held-note source** — table of (pitch, vel, ch, state, arrival). States: key down / pedal-sustained / hold-latched.
2. **Order engine** — `pool_build()` turns the table into the play sequence for the Order and Octaves settings.
3. **Step expression** — 5 lanes × 16 steps as hidden sliders 21..100 (lane L, step s = 21 + 16L + s):
   Accent % (0 = rest, 100 neutral), Octave (−2..+2, 0), Gate % (100), Chance % (100), Ratchet (1).
4. **Scheduler** — stateful: next grid time `sch_base`, absolute grid index `sch_g`, chord-relative step `sch_k`,
   plus the step in progress (`st_*`, ratchets). Everything in beats; offsets from `beat_position` per block.
5. **Generated-note registry** — (pitch, ch, off beat, on sample). Same pitch retriggered → off before on.
   Every note-off lives here until sent; discontinuity/bypass/panic flush it at offset 0.
6. **Output queue** — per-block, sorted by (offset, off-before-on). Passthrough events go through it too.

## Clock
- Playing: `beat_position`. Stopped: free clock continuing from the last block end (continuous on stop).
- Discontinuity (seek, loop, transport start): |b0 − last_end| > blocklen + 0.01 beats → flush offs, re-grid.
- Free sync keeps its phase across seeks (base shifts by the jump); Beat/Song re-grid to the next host grid point.

## Sync modes
- Free: step 0 at the chord onset; timing = chord onset + k·S.
- Beat: onsets on the host grid; chord onset rounds to the nearest grid point (late → fires immediately, early → waits).
  Pattern step = k mod len (restarts with the chord).
- Song: like Beat but the pattern step = absolute grid index mod len (locked to the song position), random too.

## Rest vs chance
- Accent 0 (rest) is silent and does NOT consume a note of the sequence (rhythm gates the melodic line).
- Chance skip is silent and DOES consume (a sparse version of the same line).

## Randomness
- `hash01(counter, salt)` with the Seed parameter; counter = k (Free/Beat) or absolute grid g (Song). Fully repeatable.

## MIDI channel
- Notes keep their input channel. Sustain pedal (CC64) is consumed as hold while enabled. All other messages pass.
- Bypass: generated notes flushed; keys still down are re-sent to the synth; on enable those are released first.

## Slider map
1 Enable, 2 Rate, 3 Order, 4 Octaves, 5 Gate, 6 Swing, 7 Hold, 8 Sync, 9 Dynamics, 10 Pattern Length, 11 Seed,
12–20 reserved, 21–36 Accent, 37–52 Octave, 53–68 Gate, 69–84 Chance, 85–100 Ratchet.

## Measured facts that shaped the code (2026-09-06)
- EEL2 identifiers are case-insensitive: a parameter `T` and a local `t` are one variable.
- `cond ? a : x = 1` parses as `(cond ? a : x) = 1`; assignments in ternary branches need parentheses.
- Due-ness is decided on block-relative sample offsets, never on beats: a note-off whose beat time
  landed an ulp inside the block would otherwise be clamped one sample early.
- Note-offs are swept only up to each new note-on: with large buffers a same-pitch retrigger would
  otherwise be cut by the previous instance's off later in the same block.
- REAPER quantises item MIDI to 960 ticks per quarter note; a chord one tick late fires "immediately"
  in Beat sync (one sample after the grid), by design.
- ReaScript `TrackFX_SetParam` uses the slider's native units for JSFX and rounds to its step.

## Verification channels
- Offline render with ArpProbe.jsfx after the arp: every MIDI event logged to gmem (abs sample, beat, bytes),
  written to a text file by the Lua batch driver. Python reconstructs the expected list and compares exactly.
- Live session with ArpSource.jsfx before the arp (scripted MIDI from gmem) for stopped transport, seek, loop.
- Not observable automatically: what it sounds like, the panel pixels (screenshots are inspected by eye).
