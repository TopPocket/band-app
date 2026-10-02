# Guitar / Bass Tuner — Spec (draft)

Status: **proposal, not built**. Open questions are at the bottom.

## Feasibility

**Very feasible** inside the single-file app, with no libraries and no server. Everything needed is in the browser:

| Need | Browser API | Notes |
|---|---|---|
| Mic input | `getUserMedia` | **Must** pass `echoCancellation:false, noiseSuppression:false, autoGainControl:false`. The voice processing that garbled snippet recordings also pumps the input level and filters it, which wrecks pitch detection. |
| Raw samples | `AudioContext` + `AnalyserNode.getFloatTimeDomainData()` | Polled about 30×/s from `requestAnimationFrame`. An `AudioWorklet` is optional later and not needed for v1. |
| Pitch detection | Our own JS, about 80 lines | YIN or McLeod (MPM) autocorrelation. **Don't use FFT peak-picking**: at 4096 samples its resolution is about 11.7 Hz, which is useless for a 41 Hz low E. |
| Screen awake | `navigator.wakeLock` | Optional. Keeps the phone from dimming mid-tune. |

### Real risks

1. **Phone mics roll off the low end.** Many phones cut below about 80–100 Hz, so a bass's open E (41.2 Hz) and B (30.9 Hz) arrive mostly as harmonics. YIN/MPM measure *periodicity*, so they still find the fundamental when it's weak. They can occasionally jump an octave, though. Mitigation: once a string is chosen, only accept pitches within about ±5 semitones of it, and fold octave errors back.
2. **Window size vs latency.** Low B needs at least about 2 periods in the window: 8192 samples at 48 kHz (about 170 ms). Guitar is fine with 2048–4096. Pick the window by instrument.
3. **Noisy rooms** (band practice). Add a noise gate (RMS threshold) and a clarity/confidence threshold. Show "—" rather than a jumpy needle.
4. **iOS Safari** only starts audio from a user tap, and the mic needs HTTPS. Both already hold for the snippet recorder.

Accuracy target: **±1 cent** on a clean sustained note, which is better than clip-on tuners.

## UX

### Entry point
- A **Tuner** button next to the BPM tapper (the same overlay pattern as `#bpm-overlay`: full-screen blurred panel, Close button).
- It remembers the last instrument and tuning in `localStorage` (a per-device preference, so not Drive).

### Layout (portrait, matching the BPM overlay style)
```
        [ Bass ▾ ]   [ Standard (EADG) ▾ ]
                 A
               110.0 Hz
     ◄───────────┼───────────►      ← needle / cents bar, ±50¢
          -12¢   (flat)
     [ E ] [ A ] [ D ] [ G ]        ← string chips; auto-detected one lit
            A4 = 440 Hz  ±
                [ Close ]
```
- **Big note name**, with the octave subscript.
- **Needle or bar**: ±50 cents. Centre zone ±3¢ is green. It eases toward the value rather than jumping.
- **Cents readout** plus a "flat"/"sharp" word, so the reading never depends on colour alone.
- **String chips**: *Auto* mode lights the nearest string. Tapping a chip **locks** that string, which helps with low bass strings and noisy rooms. Tap it again to unlock.
- **In-tune feedback**: hold within ±3¢ for 0.5 s and the chip turns solid green with a short haptic `navigator.vibrate(30)` if available.
- **Reference A**: 430–450 Hz stepper, default 440.

### Tunings (v1)
- Bass 4: Standard EADG, Drop D (DADG), half-step down.
- Bass 5: Standard BEADG.
- Guitar 6: Standard EADGBE, Drop D, half-step down, DADGAD, Open G.
- A **chromatic** mode with no target strings, so it works for any instrument.
- (v2) Custom tunings saved to Drive `_meta` so the band shares them.

## Algorithm (v1)

1. `AnalyserNode` with `fftSize` = 8192 (bass) or 4096 (guitar). Read the time-domain buffer each animation frame.
2. **Gate**: if RMS < about 0.01, show idle and skip.
3. **YIN** (difference function → cumulative mean normalised difference → absolute threshold 0.10–0.15 → parabolic interpolation). Search range: 25–400 Hz for bass, 60–1400 Hz for guitar.
4. **Confidence**: reject the frame if the YIN minimum is above the threshold (no clear pitch).
5. **Octave guard**: in locked or auto-string mode, if the frequency is about 2× or ½× of the target string, fold it.
6. **Smoothing**: median of the last 5 frames, then exponential smoothing on the *displayed* cents only (raw values still drive in-tune detection).
7. Note math: `midi = 69 + 12·log2(f / A4)`. Note = round(midi). `cents = 100·(midi − round(midi))`. In string mode, cents are measured against the chosen string's frequency instead.

The detector should be a pure function `detectPitch(Float32Array, sampleRate, minHz, maxHz) → {hz, clarity} | null`. That makes it testable in Node against synthetic sine and sawtooth buffers before it touches the UI.

## Lifecycle / cleanup
- Open: create `AudioContext` → `getUserMedia` → `createMediaStreamSource` → analyser. **Don't** connect to `destination`, or you get feedback.
- Close, or the page becomes hidden: stop all tracks, close the context, cancel rAF, release the wake lock.
- If the snippet recorder is open, the tuner should refuse to open (or close the recorder) so two mic streams never run at once.

## Effort estimate
- Detector, plus Node tests on synthetic tones: small.
- Overlay UI, tunings, lock/auto, reference A: medium.
- Real-device tuning of thresholds on Android and iPhone with real bass and guitar: the part that takes iteration. Budget a round or two of feedback.

## Open questions for Patrick
1. Which instruments and tunings matter for v1? Is a 5-string bass in the band?
2. Where should the button live: home screen, track page, Feathers, or all of them?
3. Needle (classic) or horizontal strobe-style bar?
4. Want a "play reference tone" button (the app plays the target note)? It's easy to add.
5. Shared custom tunings in Drive (v2), or keep it per-device?
