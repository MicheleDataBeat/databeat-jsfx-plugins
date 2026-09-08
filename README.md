# DataBeat JSFX Plugins

Seven plug-ins for REAPER, written in JSFX: five instruments, a reverb and a beat replayer. Each is a single text file: no installer,
build step, signing or manual compilation. You can read every line of code you run and
change it during playback. That simplicity and access to the source are why we chose JSFX.

---

## Why JSFX

Unlike most hosts, REAPER supports plug-ins that are plain text files. Drop one in the
`Effects` folder to install it. Edit it and press Ctrl+S to recompile it during playback.
These plug-ins need no notarisation, do not phone home and do not expire.

- **One file per instrument.** Presets, wavetables, graphics and the interface are all
  inside the `.jsfx` file. Nothing is installed alongside it or written to disk.
- **No dependencies.** No runtime, DLLs or sample folder to maintain.
- **Custom interfaces, fixed size.** Each instrument draws its own `@gfx` panel, showing
  all its controls at once in signal order. Each panel has a size chosen for that
  instrument rather than being scaled to fit.
- **Audio thread.** No look-ahead, declared PDC latency or memory allocation while
  you play.
- **Automatable.** Every parameter has a slider beneath the custom panel, so REAPER's
  automation, modulation and parameter learn work normally.
- **Readable source.** Each file starts with an outline of its contents.

---

## The instruments

| | Instrument | What it is | Panel | Licence |
|---|---|---|---|---|
| 🥁 | **Haruki** | Ten-lane drum instrument and step sequencer | 1180 × 720 | GPL-2.0-or-later |
| 🌊 | **Satya** | Polyphonic wavetable synthesizer, 153 presets | 1400 × 900 | MIT |
| 🎹 | **Kozue** | Ten-voice subtractive synthesizer, 128 presets | 1100 × 690 | MIT |
| 🎼 | **Satoshi** | Harmony instrument — MIDI in, voiced chords out | 1400 × 1044 | AGPL-3.0-only |
| 🪉 | **Yoshi** | Host-synchronised MIDI arpeggiator | 864 × 422 | MIT |
| 🍂 | **Momiji** | Algorithmic stereo reverb, 50 presets | 900 × 400 | MIT |
| 🔁 | **Raja** | Beat replayer: scheduled or hand-played repeats, 20 presets | 980 × 560 | MIT |

### Haruki — drums

Ten sample channels with a host-synchronised step sequencer. Each step supports velocity,
probability, substeps and microtiming, with up to 64 steps across pages. Drag a WAV onto a
channel from the Media Explorer, a track or Finder. Drop several files at once to load a
whole kit. Outputs 1-2 always carry the mix of all ten channels; outputs 3-12 carry the ten
channels one by one, so each can be processed on its own track. Haruki includes no samples
and creates no sample folder. At neutral settings, it reproduces the source PCM bit-exactly.

### Satya — wavetable synth

A polyphonic wavetable synthesizer with two oscillators, sub and noise, dual filters, a
modulation matrix and an effects chain: drive, EQ, chorus, phaser, delay, reverb, compressor
and limiter. All 153 factory presets are stored in the file.

### Kozue — subtractive synth

Ten voices with two oscillators plus noise feeding a multimode state-variable filter,
two ADSRs, two LFOs and a four-slot modulation matrix. Effects follow: chorus, flanger,
delay, reverb, compressor and limiter. Includes 128 factory presets in eight banks.

### Satoshi — harmony

A MIDI processor that produces no sound of its own. Play a key in the mapped span to
trigger its assigned chord. An explicit cost function chooses the voicing based on the
previous chord. Satoshi generates thirty chords per key from the scale rather than a
lookup table, with ten voicing families and deterministic voice leading. Put an
instrument after it on the track to hear the output.

### Yoshi — arpeggiator

Hold a chord and choose a rate and note order. A 16-step expression pattern controls
accent, octave, gate, chance and ratchet. Supports Free, Beat and Song sync. All randomness
is seeded and repeatable, so a render matches what you heard.

### Momiji — reverb

A sixteen-line feedback delay network with orthogonal scattering, in-loop low and high
damping, balanced modulation from one quadrature oscillator and a delay geometry that moves
from echo-like clustering to a dense field. Delay can lock to the host tempo (1/32 to two
whole notes, dotted and triplet). Fifty presets, a Switch Flip button that generates a bounded
musical state, and no declared latency. The dry path is the current sample.

### Raja — beat replayer

Raja catches a moment of the sound passing through it and repeats a slice of that moment: a
stutter, a fill, a held texture, a falling echo. Captures happen on a schedule you set, with a
chance you choose, or the instant you press HOLD. Presses and releases can wait for the next
sixteenth, eighth, quarter or bar, and a Pattern setting makes the random decisions come back at
the same places in every pass of a loop while the sound stays live. Slices from 1/256 to a whole
note with triplets, pitch drop and fall, level and fade, a band filter, three routings (Layer,
Replace, Repeats Only), twenty presets, no declared latency. A live timeline shows every chance,
capture and repeat.

---

## Install

Copy everything in `plugins/` into REAPER's `Effects` folder. To find it, choose
**Options → Show REAPER resource path in explorer/finder**, then open `Effects`.

| OS | Effects folder |
|---|---|
| macOS | `~/Library/Application Support/REAPER/Effects/` |
| Windows | `%APPDATA%\REAPER\Effects\` |
| Linux | `~/.config/REAPER/Effects/` |

You can use a subfolder per instrument to keep the browser tidy. To copy the `.jsfx` files
directly into `Effects` on macOS:

```bash
cp plugins/*.jsfx ~/Library/Application\ Support/REAPER/Effects/
```

In REAPER, choose **FX → Add → JS** and select the instrument. If it does not appear, use
**Options → Preferences → Plug-ins → ReaScript/JSFX → Re-scan**.

Place the plug-ins on your tracks as follows:

- **Haruki, Satya, Kozue** are instruments. Put them on a track that receives MIDI.
- **Satoshi and Yoshi** are MIDI processors. Put them on the track *before* the instrument
  that will make the sound.
- **Momiji** is an audio effect. Put it on an audio track, or on a send/bus with Mix at 100 %.
- **Raja** is an audio effect. Put it on the track you want to repeat, or on a return track with
  Routing set to Repeats Only.

---

## Documentation

`docs/` contains a folder for each plug-in. Where a full manual is available, it comes
in two formats: `MANUAL.md`, readable on GitHub, and `MANUAL.html`, which you can open in
a browser. The HTML file contains every screenshot, so no separate image folder is needed.

| Instrument | Documentation |
|---|---|
| Haruki | [Manual](docs/Haruki/README.md) · [Verification notes](docs/Haruki/VERIFICATION.md) |
| Satya | [Manual](docs/Satya/MANUAL.md) · [HTML](docs/Satya/manual/MANUAL.html) · [Preset list](docs/Satya/PRESETS.md) |
| Kozue | [Manual](docs/Kozue/MANUAL.md) · [HTML](docs/Kozue/MANUAL.html) |
| Satoshi | [Manual](docs/Satoshi/README.md) |
| Yoshi | [Manual](docs/Yoshi/README.md) · [Design notes](docs/Yoshi/DESIGN.md) |
| Momiji | [Manual](docs/Momiji/MANUAL.md) · [HTML](docs/Momiji/MANUAL.html) |
| Raja | [Manual](docs/Raja/MANUAL.md) · [HTML](docs/Raja/MANUAL.html) |

---

## Licences

Each instrument has its own licence, stated in its source file and summarised here.
Two of the seven use copyleft licences for different reasons.
The complete licence texts and the file-by-file scope are in [`LICENSE.md`](LICENSE.md).

**Satya, Kozue, Yoshi, Momiji, Raja — MIT.** Copyright © 2026 Michele Ibba. Do what you like.

**Satoshi — AGPL-3.0-only.** A JSFX adaptation of the author's previous unpublished works.
Its `@gfx` toolkit and sequencer lane design come from Haruki, which is the same author's own
code, so no third licence is involved.

**Haruki — GPL-2.0-or-later.** It embeds Liteon's `moog24db` ladder filter, which is released
under the GPL, and that makes the whole file GPL. The filter implements the Moog approximation
from the Stilson/Smith CCRMA paper by way of the CSound source. Ivanov's notice asks for more
than the bare GPL does — acknowledgement of the author — so it is reproduced in full and
unaltered inside `Haruki.jsfx`, beside the filter itself, and again here:

> (C) 2008-2009, Lubomir I. Ivanov
>
> NO WARRANTY IS GRANTED. THIS PLUG-IN IS PROVIDED ON AN "AS IS" BASIS, WITHOUT WARRANTY OF ANY
> KIND. NO LIABILITY IS GRANTED, INCLUDING, BUT NOT LIMITED TO, ANY DIRECT OR INDIRECT, SPECIAL,
> INCIDENTAL OR CONSEQUENTIAL DAMAGE ARISING OUT OF THE USE OR INABILITY TO USE THIS PLUG-IN,
> COMPUTER FAILTURE OF MALFUNCTION INCLUDED. THE USE OF THE SOURCE CODE, EITHER PARTIALLY OR IN
> TOTAL, IS ONLY GRANTED, IF USED IN THE SENSE OF THE AUTHOR'S INTENTION, AND USED WITH
> ACKNOWLEDGEMENT OF THE AUTHOR. FURTHERMORE IS THIS PLUG-IN A THIRD PARTY CONTRIBUTION, EVEN IF
> INCLUDED IN REAPER(TM), COCKOS INCORPORATED OR ITS AFFILIATES HAVE NOTHING TO DO WITH IT. LAST
> BUT NOT LEAST, BY USING THIS PLUG-IN YOU RELINQUISH YOUR CLAIM TO SUE IT'S AUTHOR, AS WELL AS
> THE CLAIM TO ENTRUST SOMEBODY ELSE WITH DOING SO.
>
> Released under GPL: <http://www.gnu.org/licenses/>.

Upstream states no GPL version, so "v2 or later" is this project's reading, not Ivanov's words.
Ivanov's code appears in `Haruki.jsfx` in three places — the anti-denormal constant, the p/k/r
coefficients, and the four-stage ladder — and each is marked, so a future attempt to drop the
copyleft dependency knows what it has to remove.

Copyright © 2026 Michele Ibba, except where a file names another author.

---

## References

- [REAPER](https://www.reaper.fm) · [JSFX programming reference](https://www.reaper.fm/sdk/js/js.php) · [EEL2 language](https://www.cockos.com/EEL2/)
- The author's previous unpublished works — the basis for Satoshi
- Liteon's JSFX, including `moog24db`, ship with REAPER and are also on [ReaTeam/JSFX](https://github.com/ReaTeam/JSFX)

---

## Layout

```
README.md      this file
plugins/       the seven .jsfx files, exactly as installed
docs/          a folder per instrument
```

`plugins/` contains the finished builds, byte-identical to those installed and running in
REAPER. All seven repository copies have been loaded in REAPER and report their expected
parameter counts.

`docs/` is adapted from each instrument's development repository. The text is the author's,
with references to test harnesses, review logs and other files not included here removed.
Every file referenced in a manual is available to the reader. Each instrument is developed
in a separate private repository.
