# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Running the App

Open `index.html` directly in a browser — no server, no build step, no dependencies.
```
start index.html   # Windows
```
No tests, no linter, no CI. Verify manually: place notes, press Play, listen.

## Architecture

Everything lives in `index.html`. Internal layout order:

```
<style>      CSS design system (variables, transport, canvas, buttons)
<body>       Transport bar → piano roll canvas wrapper
<script>
  Layout     KEYS_W, RULER_H, ROW_H, COL_W, PITCH_HI/LO, NUM_ROWS, BLACK_KEYS
  State      notes[], bpm, bars, playing, looping, currentBeat
  Helpers    totalBeats(), numCols(), secPerBeat(), patternSec(), isBlackKey(), midiToHz()
  Canvas     resizeCanvas(), drawGrid(), drawNotes(), drawCursor(), drawAll()
  AnimLoop   animLoop() — requestAnimationFrame, only active during playback
  Audio      ensureAudio(), fireNote(), tick() scheduler, startPlay(), stopPlay()
  Mouse      mousedown/mousemove/mouseup on canvas — note placement, drag-extend, piano preview
  Transport  playBtn, stopBtn, bpmSlider, barsSelect, loopBtn, clearBtn event listeners
  Boot       resizeCanvas(), drawAll()
```

## Note Data Model

```js
// A note
{ pitch: 60, start: 0.25, duration: 0.5 }
// pitch:    MIDI number (36 = C2 … 83 = B5)
// start:    beat position (0.25 = second 16th note; 1 beat = 1 quarter note)
// duration: length in beats (0.25 = one 16th note cell, 1.0 = one quarter note)
```

`CELL_BEATS = 0.25` — one column = one 16th note = 0.25 beats.

## Audio Scheduling

Uses the Web Audio API **lookahead scheduler** pattern:
- `setInterval(tick, 25)` runs every 25ms during playback
- `tick()` schedules any notes whose audio start time falls within the next `AHEAD = 0.12` seconds
- Each note creates an `OscillatorNode → GainNode → masterGain` subgraph
- Gain envelope: 5ms attack, 30ms release (avoids clicks)
- Nodes are created fresh per note and auto-collected after `.stop()`
- `scheduled` Set tracks `"noteIdx:loopIdx"` keys to prevent double-scheduling

## Canvas Coordinate System

```
x = KEYS_W + col * COL_W          // grid column → canvas x
y = RULER_H + pitchToRow(p) * ROW_H  // pitch → canvas y

col   = floor((x - KEYS_W) / COL_W)
row   = floor((y - RULER_H) / ROW_H)
pitch = PITCH_HI - row              // row 0 = highest pitch (top)
```

## Related Projects (DSP sources)

| Project | Path | Reusable items |
|---------|------|----------------|
| Vocoder | `../Vocoder/index.html` | `encodeWAV`, `normalize`, `fft/ifft`, all biquad filters, `schroederReverb`, `oscillator`, `runVocoder`, `decodeAndResample` |
| Voice-Mod | `../voice-mod/index.html` | `echoDelay`, `applyChorus`, `applyTelephone`, `formantShift`, `pitchShift`, `applyThickness` |
| Noise | `../Noise/app.js` | `NoiseProcessor` AudioWorklet, `createReverbIR`, grit curve, pad-mode oscillators, sub-oscillator pattern, LFO patterns |

## Adding a New Effect (future versions)

1. Add the Web Audio nodes to a `buildEffectChain(audioCtx, destination)` factory per track
2. Return a `{ input, output, params }` object — notes connect to `input`, `output` connects downstream
3. Add UI controls (sliders/toggles) to a collapsible per-track panel
4. Wire controls to AudioParam `.setTargetAtTime()` for smooth real-time changes
5. For offline export: reconstruct the same graph in `OfflineAudioContext`

## Adding a New Instrument Type (future versions)

1. Each instrument type exposes a `createSource(audioCtx, pitch, duration) → AudioNode` factory
2. The scheduler calls this factory instead of directly creating `OscillatorNode`
3. Source node connects into the track's effect chain input
4. For AudioWorklet instruments (Noise): register the worklet module once on `ensureAudio()`, reuse thereafter
