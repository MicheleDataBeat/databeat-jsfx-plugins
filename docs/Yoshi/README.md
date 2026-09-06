# Yoshi (JSFX)

A host-synchronised MIDI arpeggiator for REAPER in one file, `Yoshi.jsfx`.
Hold a chord, pick a rate and an order; a 16-step expression pattern (accent, octave, gate,
chance, ratchet) shapes the result. Shows up in the FX browser as **JS: Yoshi**.

## Install

Copy `Yoshi.jsfx` into REAPER's `Effects` folder (Options → Show REAPER resource path),
e.g. `Effects/Yoshi/Yoshi.jsfx`. Insert it on an instrument track before the
synth. Play or hold a chord.

## Controls

| Control | What it does |
|---|---|
| **ON** | Bypass with clean hand-over: generated notes are released, keys still down are passed to the synth, and taken back when switched on again. |
| **HOLD** | The chord keeps playing after the keys are released; the next chord replaces it. |
| **PANIC** | Releases every generated note, clears the chord, sends All Notes Off on all channels. |
| **Rate** | 1/1 … 1/32, dotted (`.`) and triplet (`T`), ordered from slow to fast. Drag or scroll to step, click for the list. |
| **Order** | Up, Down, Up/Down, Down/Up (ends not repeated), As Played, Random (repeatable, no immediate repeats), Chord (the whole chord per step, climbing the octave range). |
| **Octaves** | 1–4: the sequence is repeated one octave up per extra octave. |
| **Gate** | Note length as a share of the step (5–200 %). Over 100 % overlaps the next note; a repeated pitch is always released before it sounds again. |
| **Swing** | Delays every other step; 100 % is a full triplet feel. |
| **Sync** | **Free**: the sequence starts on your chord and keeps its own phase. **Beat**: notes sit on the host grid (a chord played slightly late fires at once, an early one waits for the grid); the pattern restarts with each chord. **Song**: as Beat, and the pattern follows the song position, so accents stay on the bar wherever the chord is played. |
| **Dynamics** | 100 % follows the played velocity, 0 % plays every note at 100. |

Every value cell: drag up/down, Shift for fine control, scroll wheel to nudge, double-click to reset.

### Pattern

One lane, switched between **Accent** (0 = rest, 100 % neutral), **Octave** (−2…+2), **Gate**
(scales the Gate control), **Chance** (probability that the step plays) and **Ratchet** (1–4 hits
inside the step). Drag across the steps to draw, double-click or right-click a step to reset it,
scroll to nudge. **LENGTH** (or the strip above the steps) sets how many steps play, immediately.
**DICE** fills the shown lane with musically bounded random values, **RESET** clears it. All
randomness (Chance, Random order, Dice) is repeatable and depends on **SEED**.

Two musical rules worth knowing:

- A **rest** (Accent 0) is silent and does not consume a note of the sequence, so a rhythm drawn in
  the Accent lane plays against the full melodic cycle.
- A **Chance** skip is silent and does consume its note: the same line, thinned out, in time.

A neutral pattern (all defaults) leaves the arpeggio unchanged.

### MIDI

Notes keep their input channel; all channels are arpeggiated together. The sustain pedal (CC 64)
holds the chord while the arpeggiator is on and is not passed on; every other message (CCs, pitch
bend, aftertouch, program change) passes through at its exact position. Up to 32 held notes.

### Timing

Timing comes from the host beat position, so tempo changes, seeks, loops and transport starts are
handled by construction; while the transport is stopped a free clock continues from the last block.
A jump (seek, loop, start) releases sounding notes and re-grids. Events are placed at sample
offsets inside the block, independent of the buffer size.

## Parameters and state

Sliders 1–11 are the global controls, 21–100 the five pattern lanes (`Step n Accent (%)`,
`Step n Octave`, …), all automatable and saved with the project. The shown lane is saved in the
plug-in's serialized state (versioned).

## How it was verified

Nothing here is asserted from a listening session. REAPER is driven from the command line, a
project is rendered offline, every MIDI event the plug-in emits is logged through a probe, and a
reference implementation reconstructs the events that were expected — then the two are compared
sample by sample.

Covered that way: 47 engine cases at 44.1, 48 and 96 kHz; the transport (stopped clock, hold,
play, seek, loop, bypass, PANIC); pointer gestures injected into the panel with the parameters
read back; and three buffer sizes (64, 128, 2048).

## Licence

MIT, Michele Ibba. No third-party code is incorporated.
