# DAW — Version Roadmap

Each version adds one feature. Every version is playable and complete before the next begins.

---

## v1.0 — Piano Roll MVP ✅
Canvas-based piano roll (C2–B5, 48 rows × 16th-note grid). Click to place/remove notes. Click-drag right to extend note duration. Sine wave playback via Web Audio lookahead scheduler. BPM slider, bars selector, loop toggle, clear button. Dark theme. Piano key column previews notes on click.

---

## v1.1 — Waveform Selector ✅
Add a waveform button group in the transport bar: **Sine / Square / Triangle / Saw**. The selected waveform is used for all new note playback. Reuse `oscillator()` waveform logic from `../Vocoder/index.html`.

---

## v1.2 — ADSR Envelope ✅
Add Attack, Decay, Sustain, Release sliders in the transport bar (or a collapsible panel). Apply to each note's `GainNode` automation. Default: A=5ms, D=80ms, S=0.7, R=60ms.

---

## v1.3 — Grid Resolution + Zoom ✅
Add a resolution selector: **1/4 / 1/8 / 1/16 / 1/32** note grid (changes `COL_W` and `CELL_BEATS`). Add a horizontal zoom slider that scales `COL_W` without changing the musical grid.

---

## v2.0 — Multiple Tracks ✅
Stack 3 independent tracks, each with its own piano roll canvas, mute button, solo button, and volume knob. Tracks play simultaneously. Transport Play/Stop controls all tracks in sync.

---

## v2.1 — Note Velocity ✅
Click-drag **vertically** on a placed note to set its velocity (0–127). Notes rendered with opacity proportional to velocity. `fireNote()` scales gain by velocity.

---

## v3.0 — Reverb Effect
Per-track reverb toggle. Uses `ConvolverNode` with a procedurally generated impulse response (port `createReverbIR()` from `../Noise/app.js`). Controls: Mix (0–100%), Room size (small/medium/large), Decay.

---

## v3.1 — Echo / Delay Effect
Per-track delay using `DelayNode` + feedback `GainNode` + dry/wet blend `GainNode`s. Controls: Delay time (ms), Feedback (0–90%), Mix. Port the concept from `echoDelay()` in `../voice-mod/index.html`.

---

## v3.2 — EQ (Bass + Treble)
Per-track `BiquadFilterNode` pair: low-shelf + high-shelf. Bass ±12 dB, Treble ±12 dB. Port `lowShelfBiquad()` / `highShelfBiquad()` coefficient formulas from `../Vocoder/index.html` as a reference for the filter parameters.

---

## v3.3 — Distortion / Grit
Per-track `WaveShaperNode` with a tanh-based curve. Port the grit curve generation from `applyGrit()` in `../Noise/app.js`. Control: Grit amount (0–1).

---

## v4.0 — WAV Export ✅ *(implemented early)*
"Export WAV" button in the transport. Uses `OfflineAudioContext` to render the full pattern at 44100 Hz. Reconstructs all tracks and effects offline. Encodes to 16-bit PCM WAV using `encodeWAV()` ported from `../Vocoder/index.html`. Download as `daw-export.wav`.

---

## v4.1 — Noise Instrument
Per-track instrument type selector. Add "Noise" type: white / pink / brown noise burst shaped by a lowpass filter whose cutoff maps to the note's MIDI pitch. Port the `NoiseProcessor` AudioWorklet class from `../Noise/app.js` (with ScriptProcessor fallback). Good for drum-like rhythmic textures.

---

## v4.2 — Pad / Drone Instrument
Add "Pad" instrument type: 5 detuned sawtooth oscillators panned ±36%, ±18%, 0% (from the pad mode in `../Noise/app.js`). Controls: Detune width (cents), mix gain. Long sustain. Good for ambient layers.

---

## v5.0 — Sub Bass Toggle
Per-track sub-oscillator: sine wave one or two octaves below the note pitch, mixed at adjustable amount. Port the sub-oscillator + 130 Hz lowpass pattern from `../Noise/app.js`. Controls: Sub amount (0–100%), Octave (−1 / −2).

---

## v5.1 — Vibrato
Per-track vibrato: `OscillatorNode` LFO connected to the `detune` AudioParam of each source oscillator. Controls: Rate (0.1–10 Hz), Depth (0–100 cents). Port from Noise project's `vibratoLfo` pattern.

---

## v5.2 — Tremolo
Per-track tremolo: `OscillatorNode` LFO → `GainNode.gain`, biased so gain stays positive. Controls: Rate (0.1–20 Hz), Depth (0–100%). Port from Noise project's `tremoloLfo + tremoloBias` pattern.

---

## v6.0 — Chorus Effect
Per-track chorus: LFO-modulated delay line. Port `applyChorus()` concept from `../voice-mod/index.html` as real-time Web Audio nodes (`DelayNode` + `OscillatorNode` modulating delay time). Controls: Rate, Depth, Mix.

---

## v6.1 — Telephone Filter
Per-track telephone filter: 4th-order bandpass 300–3400 Hz. Port `applyTelephone()` from `../voice-mod/index.html`, implemented as two cascaded `BiquadFilterNode`s in the Web Audio graph. Mix control (0–100%).

---

## v7.0 — Instrument Presets
Save and load instrument+effects settings per track as named presets. Stored in `localStorage`. UI: preset name chips (matching voice-mod's chip style). Ships with 5 built-in presets: Clean Sine, Warm Pad, Sub Bass, Lo-Fi Bell, Noise Snap.

---

## v7.1 — Pattern Clipboard
Click-drag to select a rectangular region of notes (highlight overlay). Keyboard shortcuts: **Ctrl+C** copy, **Ctrl+V** paste (click to set paste position), **Delete** remove selected. Visual selection box drawn on canvas.

---

## v8.0 — Vocoder Instrument
New instrument type: "Vocoder". Upload an audio file as the modulator for a track. The track's synth output feeds the vocoder as the carrier. Port `runVocoder()`, `buildFilterBank()`, `hilbertEnvelope()`, and `extractEnvelope()` from `../Vocoder/index.html`. Processed offline per note via `OfflineAudioContext`; output cached as an `AudioBuffer` and played back via `AudioBufferSourceNode`. Controls: Bands, Attack, Release, Contrast.

---

## v8.1 — Formant Shift Effect
Per-track formant shift (vocal-tract resonance). Port `formantShift()` from `../voice-mod/index.html`. Factor control: 0.5× (giant) → 2.0× (chipmunk). Applied during offline export.

---

## v9.0 — Microphone Recording into Track
Record from microphone directly into a track's timeline. Uses `getUserMedia` + `MediaRecorder` (pattern from `../Vocoder/index.html`). Recorded audio appears as a waveform clip in the piano roll. Playback via `AudioBufferSourceNode` at the recorded position.

---

## v9.1 — Noise Generator Track (from Noise project)
A special track type that mirrors the Noise & Drone project's full synthesis engine (noise/sine/pad modes with texture, darkness, movement, grit, space, drift, age controls). The track outputs a continuous generated sound that runs for the full pattern duration. Useful for ambient beds and drone layers.

---

## Future Ideas
- URL-shareable patterns (base64 state in fragment)
- MIDI import/export
- Multiple patterns + arrangement view
- Mixer view with per-track EQ visualization
- Real-time AudioWorklet synthesis per note (lower latency than OscillatorNode scheduling)
