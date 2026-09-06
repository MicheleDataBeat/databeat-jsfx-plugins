# Haruki — verification record

REAPER 7.79 (macOS arm64), project 44.1 kHz / 120 BPM, rendered to 32-bit float
and compared in Python by the harness kept in the development repository. Test material is
from the author's own
drum library — nothing is bundled with the plug-in and nothing is written into
REAPER's Data folder.

Because Haruki has no file slider, the harness loads samples the only way a
script can: it builds a `@serialize` blob and injects it into the saved project's
`<JS_SER>` chunk. Every run therefore also exercises state restoration.

Not machine-verified: **drag-and-drop itself**, and mouse interaction generally.
ReaScript cannot synthesise a file drop or drive the mouse. The drop code follows
the pattern used by REAPER's own stock `super8` and by three other shipped
plug-ins, and the interface renders correctly (screenshot), but the gesture needs
a human. Everything downstream of the drop — path storage, loading, absolute-path
`file_open`, serialization, missing-file handling — is verified below.

---

## 1. Audio transparency

Track A: the WAV placed directly on a track (item and take volume unity, pan
centre, fades and auto-fades zeroed, source looping off, no FX).
Track B: the same WAV in a Haruki channel, all controls neutral, triggered by a
MIDI note at velocity 127 aligned to the item start. Identical routing, each
rendered on its own pass.

```
render: 44100 Hz 32-bit fmt=3 (3=float)

case                           ref peak     dut peak     max|diff|   verdict
A kick   stereo 16b 44.1k      0.714172363  0.714172363  0.000e+00   SAMPLE-IDENTICAL
B clap   mono 16b 44.1k        0.851013184  0.851013184  0.000e+00   SAMPLE-IDENTICAL
C hat    stereo 16b 44.1k      0.221038818  0.221038818  0.000e+00   SAMPLE-IDENTICAL
```

Bit-exact over the whole event, so a polarity-inverted sum is numerical silence.
The neutral path now has four bypasses to get right (interpolation, attack, decay,
ladder filter); all four are skipped by branch.

## 2. Source sample rate ≠ project sample rate

```
48 kHz source in a 44.1 kHz project:
   peak ref 0.439081  dut 0.439485  ratio 1.00092 | correlation 0.999929
```

Correct speed, correct pitch at Tune 0, no level or length change. Nulling is not
claimed: REAPER's item playback and the plug-in's load-time conversion are two
independent resampling passes.

## 3. Control laws and envelopes

```
   velocity 64/127  L 0.356253088 (expect 0.356253076)  R 0.359897882 (expect 0.359897884)  ok
   gain -6.0 dB     L 0.354309142 (expect 0.354309151)  R 0.357934058 (expect 0.357934071)  ok
   pan hard left    L 0.706939697 (expect 0.706939697)  R 0.000000000 (expect 0.000000000)  ok
   attack 50        target 10.00 ms linear ramp; max error vs ideal 2.27e-03  ok
   decay 60         target 109.9 ms to -60 dB; max error vs ideal 1.42e-03  ok
```

Attack is a linear ramp (a concave RC curve is right for a drum's decay and wrong
for its attack: at "10 ms" an RC ramp still passes the first millisecond at
−8.7 dB where linear gives −20 dB). Decay is the exponential to −60 dB with its
sense inverted, so leftmost is FULL and turning right shortens.

## 4. Moog ladder filter

```
== MOOG LADDER (cutoff 55 -> 902 Hz) ==
   below cutoff   open   +12.1 dB   filtered   +12.0 dB   delta    -0.0 dB
   at cutoff      open   -22.8 dB   filtered   -31.4 dB   delta    -8.6 dB
   2-3 oct above  open   -39.2 dB   filtered   -96.1 dB   delta   -56.9 dB
   far above      open   -44.3 dB   filtered  -130.1 dB   delta   -85.7 dB
   octave-to-octave attenuation growth above cutoff: -23.6 dB (24 dB/oct target)
   resonance 70%: +7.5 dB lift at cutoff vs resonance 0
   filter output finite and bounded (peak 0.699): True
```

−23.6 dB per octave confirms a true 4-pole. Passband is flat, resonance lifts the
cutoff region, and the output stays bounded. At CUTOFF = OPEN the filter is not
merely wide open, it is branch-skipped — which is what keeps section 1 exact.

Code: REAPER's stock `Effects/Liteon/moog24db`, (C) 2008-2009 Lubomir I. Ivanov,
GPL, implementing the Stilson/Smith CCRMA approximation via CSound. Coefficients
are computed once per voice at trigger time rather than interpolated per block,
because a one-shot voice's cutoff does not move. Haruki is GPL as a result.

## 5. Sequencer, microtiming, substeps, probability

A known pattern was injected as `@serialize` state and rendered offline. Rather
than hunt for onsets, the check **rebuilds the expected render** by summing the
source PCM at the offsets the scheduler's own formula predicts, and requires the
difference to be zero. 1/16 at 120 BPM = 5512.5 samples per step, so step onsets
deliberately fall on fractional sample positions.

```
expected 18 events reconstructed from the source PCM
lane   step      sample   note
1      1         0
2      3         12403    micro +25%
6      4         16537    substep 1/2
6      4         19293    substep 2/2
1      5         22050
3      7         31696    micro -25%
1      9         44100
1      13        66150
5      15        77175    vel 50%
                          (bar 2 repeats all of the above at +88200)

max|render - reconstruction| = 0.000e+00  (peak 0.2252)
probability-0 lane at samples 55125 and 143325 contributes nothing: True

SEQUENCER + MICROTIMING: EXACT
```

A zero difference over the whole four-second render means every event — including
the microtimed ones — is on its exact predicted sample, with exact amplitude.

### Microtiming hazards and how they are handled

| hazard | handling |
|---|---|
| a shifted event falls outside the scanned step range and is never emitted | the scheduler scans two steps either side of the block; with swing ≤ +0.5 step and microtiming ∈ ±0.5 step an onset lies in `[g−0.5, g+1.0]` steps, which that range covers |
| a late step followed by an early one collapses or inverts the substep spacing | the substep span is computed from the **swing-only** grid, so it never depends on either step's microtiming; the shift then translates the whole substep group |
| step 0 of the pattern pulled before time zero | the onset is simply never inside a block, so nothing is emitted — no wrap, no crash |
| events emitted out of order | the event queue is sorted by sample offset before `@sample` consumes it |

## 6. All ten channels from MIDI, and missing-file safety

Nine channels loaded, channel 10 pointed at a file that does not exist; ten MIDI
notes 36-45, one every 0.4 s, sequencer off.

```
ch     note   sample    expected
1      36     0         sounds
2      37     17640     sounds
...
9      44     141120    sounds
10     45     158760    SILENT (file deliberately missing)

max|render - reconstruction| = 0.000e+00  (peak 0.2210)
channel 10 window peak = 0.000e+00 (must be ~0)

TEN CHANNELS FROM MIDI + MISSING-FILE SAFETY: ALL PASS
```

## 7. Measured JSFX facts behind the design decisions

**Drag-and-drop exists in JSFX and is the only workable loading route.**
`gfx_getdropfile(idx[, #str])` is absent from the JSFX graphics documentation but
present in the REAPER binary's EEL function table and used by Cockos' own stock
`super8` (`InstallFiles/Effects/loopsamplers/super8:1159`), by Saike's tools and
by Tukan's. Introduced in REAPER 5.91 (June 2018). Semantics: returns 1 and fills
the string if `idx` is in range, 0 otherwise; the list persists across frames
until `gfx_getdropfile(-1)` clears it; `mouse_x`/`mouse_y` are valid on the drop
frame, which is what gives per-lane targeting. A drag from REAPER's *FX browser*
delivers `@fx:<ident>` rather than a path (6.68+), so those are filtered out.

**JSFX has no file dialog**, and a script **cannot enumerate a file slider**:
`strcpy_fromslider` returns REAPER's own current selection and ignores a value
written to the slider. Measured four ways in a probe plug-in — directory of four
files, first filename character returned for indices 0-7:

```
A plain assign       a a a a a a a a
B +sliderchange      a a a a a a a a
C +slider_automate   a a a a a a a a
D index arg          a - - - - - - -
```

Row D also shows the second argument is a *slider index*, not a value. An earlier
build assumed enumeration worked and would have shown one filename 300 times.

## 8. Defects found and fixed during this round

| # | defect | evidence | fix |
|---|---|---|---|
| 1 | A resonant ladder rings on after the sample ends; cutting the voice at the last PCM frame would clip the tail | design review of the new filter against the existing end-of-voice rule | when the filter is engaged the voice keeps clocking silence through the ladder until its output falls below 1e-6, bounded to 4096 samples |
| 2 | Microtiming could collapse or invert substep spacing, and could push an onset outside the scheduler's scan window | hazard analysis, section 5 | substep span computed from the swing-only grid; scan widened to two steps either side |
| 3 | A drag from the FX browser would be stored as a sample path and fail to open | research finding on `@fx:` pseudo-paths | drops whose path begins with `@fx:` are ignored |

Two further problems were in the **harness**, not the plug-in, and are recorded so
the numbers are not misread: the 48 kHz test file turned out to be a 4.67 s loop
whose tail bled into the two following cases (fixed by moving it last), and
`seq_check.py` called `main()` at import time, so importing a constant from it
aborted the patch step before any render happened.
