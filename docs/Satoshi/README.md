# Satoshi

**A harmony instrument for REAPER. One file, no build step.**

This is a JSFX / EEL2 adaptation of the author's previous unpublished works. It makes no sound:
it reads MIDI and writes MIDI. Play a key in the mapped span and the chord at that position
sounds, voiced against the chord you played last by an explicit cost function.

```
     Your keyboard  ──▶  Satoshi  ──▶  Any instrument  ──▶  🔊
                       (picks the notes)   (makes the sound)
```

---

## Install

Copy `Satoshi.jsfx` into REAPER's `Effects` folder (**Options → Show REAPER resource path**),
for example `Effects/Satoshi/Satoshi.jsfx`.

Then in REAPER: **FX → Add → JS → Satoshi**. Put an instrument *after* it on the same track,
arm the track for MIDI, and play.

There is nothing to compile, nothing to sign, and no Gatekeeper step — which is the main
practical reason this port exists.

## What it does

Pick a key. You get a palette of thirty chords in three columns: the ones that belong
(**Diatonic**), the ones you are borrowing (**Borrowed** — modal interchange and secondary
dominants), and the ones that are frankly showing off (**Chromatic** — altered dominants,
tritone subs, chromatic mediants).

Then it voices them. Not by picking notes at random within the chord, but by scoring every
candidate voicing against the chord you just played: how far each voice has to move, how many
common tones survive, whether the bass leaps, whether the top line sings. The cheapest wins.

- **Thirty chords per key**, generated from the scale rather than a lookup table — so it works
  in Dorian, in Phrygian, in a pentatonic, or in a scale you invent.
- **Ten voicing families** — Close, Open, Drop 2, Drop 3, Rootless, Shell, Quartal, Clustered,
  Spread, Upper structure.
- **Deterministic voice leading**, five modes, scored against what you actually played last.
- **Colour that follows theory.** Turn up Extensions and chords gain sevenths, ninths and
  thirteenths stacked from *your* scale — and the chord renames itself to match. One rule does
  the work: never add a note a semitone above one already there.
- **Four genre presets** and **eight character macros**.

In C major with the colour up, the palette reads `Cmaj13 · Dm13 · Em11 · Fmaj13 · G13 · Am11 ·
Bm11b5` — textbook diatonic jazz harmony, and nobody typed it in.

## Playing it

The **white keys** carry the palette, in its **stable storage order**, and each column starts
at a C:

| Column | Starts at | In C major |
|---|---|---|
| Diatonic | the **mapping base note** (default 36 = C2) | C2 – B2, 7 cards |
| Borrowed | the next C above the diatonic column | C3 – B4, 14 cards |
| Chromatic | the next C above that | C5 – D6, 9 cards |

Black keys carry nothing, so they follow the **Unmapped notes** setting — pass them through and
you can still play a bass line under your own chords on the same keyboard. A column that does
not fill its last octave leaves the remaining white keys silent, and they flash the palette the
way any key past the last card does.

That order never changes while you play. Sliders and rank changes rearrange the cards *on
screen*; the keyboard mapping is deliberately decoupled from that so your hands keep their
bearings.

The **sustain pedal** works: CC 64 is consumed by the plug-in, which defers its own note-offs,
so the notes downstream hold for exactly as long as you hold the pedal. Everything else — pitch
bend, other CCs, aftertouch — passes through untouched.

## Sequencing

A monophonic chord lane along the bottom, borrowed from Haruki: sixteen X-0-X steps to the bar,
up to four bars, following the host transport. Click a step to switch it on, drag to paint,
right-click to select one and set its values on the three rotaries.

Each ON step carries a **card** — named by its degree and column, `V diatonic`, `bVI borrowed`
— and a **length** of 1 to 64 steps, a sixteenth of a bar up to four bars. A chord holds for its
length, or until the next ON step, or the end of the pattern, whichever comes first. **Swing**
is MPC-style — 50 % straight, 66.7 % a triplet feel, 75 % the most an MPC gave you — and delays
every second sixteenth; microtiming then nudges any step by up to half a step from wherever
swing put it.

A step stores the *chord*, not the notes, so moving any control while it plays revoices what it
plays next. While the lane runs the keyboard is ignored, so the two never fight.

## What is *not* here, versus the previous unpublished works

This adaptation contains the **harmony engine** in full. It is not the whole application. Left out, each
for a reason:

| Not ported | Why |
|---|---|
| **The 1–32 bar sequencer**, live/punch recording, undo, clipboard | REAPER *is* the sequencer. Record Satoshi's output to a MIDI item and edit it with tools far better than a JSFX lane. This was the single largest subsystem in the original and its whole surface is mouse gestures a script cannot verify. |
| **Standard MIDI File export and drag-out** | Same reason: record the output. |
| **Key detection from an audio file** | It needs an FFT over decoded audio; JSFX cannot decode MP3 or FLAC, and the analysis path is a second subsystem with its own model. |
| **The voicing picker's inverse control search** | Its defining behaviour is searching 5,445 control combinations for one that ranks your pick first — and the original measures that no such setting exists about two thirds of the time. The **Alternatives** row here shows the top six candidates and auditions them on click; it does not move your controls. |
| **Three responsive layout modes** | A `@gfx` section has one size. The canvas is a fixed 1400×1044, sized from the content. |
| **The settings sheet and the custom-scale sheet** | Every setting is a slider; the custom scale is a 12-bit degree mask on slider 31. JSFX has no text entry, so a custom scale has no name. |

## Controls

Every control is a slider, so every control automates and every control is saved with the
project. The sliders are hidden from the generic UI because the `@gfx` panel is the interface.

| Slider | | Slider | |
|---|---|---|---|
| 1 | Tonic | 17 | Macro: Unexpected |
| 2 | Scale (12 + Custom) | 18 | Macro: Angular |
| 3 | Spelling | 19 | Macro: Modal |
| 4 | Preset | 20 | Macro: Chromatic |
| 5 | Family | 21 | Mapping base note |
| 6 | Notes (1–10) | 22 | Input channel (0 = omni) |
| 7 | Register (MIDI 0–127) | 23 | Output channel (0 = follow input) |
| 8 | Density | 24 | Follow-input fallback |
| 9 | Spread | 25 | Unmapped notes: block / pass |
| 10 | Extensions | 26 | UI velocity |
| 11 | Include root | 27 | Preview length (ms) |
| 12 | Voice-leading mode | 28 | Preview on select |
| 13 | Macro: Tense | 29 | Theory labels |
| 14 | Macro: Bright | 30 | Mood words |
| 15 | Macro: Lush | 31 | Custom scale degree mask |
| 16 | Macro: Open | | |

**Set your macros first, then fine-tune Shape.** Moving a macro re-derives the whole shape from
the preset and discards hand-tuned Shape edits. That is inherited, documented behaviour from
the previous unpublished works, not a defect of the adaptation.

The previous unpublished works included fuller theory documentation, control explanations and
ten worked examples. Their material corresponding to Parts I, II, III (except the sequencer,
settings sheet and analysis sections), V and VI applies here unchanged.

## The diagnostic report

A JSFX has no console. So the plug-in can be asked, over MIDI, to describe its own state as
text — which is how the test harness reads the palette, the enrichment ladder, the candidate
list and the resolved style without a debugger.

Send **CC 119** on any channel; the value selects a report (1 palette, 2 style, 3 candidates,
4 registry, 5 tones, 6 scale). Send **CC 118** to choose which card is displayed, **CC 117** to
set the tonic and **CC 113** to set the scale — those last two exist because a script cannot
move a slider mid-render, and without them one render could only ever observe one key. The report
comes back as CC 118 messages on channel 16, one ASCII byte each, terminated by CC 116. Both
request CCs are consumed, never forwarded.

The footer also shows what the engine actually costs on the audio thread — `trigger 0.34 ms
(peak 0.91)` — because an unmeasured budget is an assumed one. Click it to reset the peak.

## Licence

Copyright (C) 2026 Michele Ibba.

**GNU Affero General Public License, version 3 only (AGPL-3.0-only).** This is a derivative
work of the author's previous unpublished works and is released under AGPL-3.0-only. The full
grant is reproduced in the header of `Satoshi.jsfx`.

No JUCE, no Steinberg SDK, no FLAC, no HarfBuzz and no bundled typeface are present in or
linked by this file. Those components belonged to the previous unpublished works, not to this
one. JSFX runs inside REAPER and links nothing.

**Shared code with Haruki.** The `@gfx` drawing toolkit — the colour, rectangle, text and
hit-test helpers, the vertical-drag control, the rotary and the step strip — is taken from
[Haruki](../Haruki/README.md), and so is the sequencer lane's visual design. Haruki is by the
same author and is distributed under GPL-2.0-or-later because it embeds a third-party filter;
none of that filter is present here, and the shared code is the author's own, so it is
licensed AGPL-3.0-only as part of this work. The chord sequencer's scheduling logic is an
independent re-implementation, not a copy.

**Provenance.** Satoshi is the public JSFX adaptation of the author's previous unpublished
works.
