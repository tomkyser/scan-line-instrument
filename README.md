# Scan Line Instrument

A browser-based instrument that turns your phone’s camera into a real-time music generator. A vertical scan line samples light, color, and contrast from the world around you — objects crossing it become notes, edges become plucks, the horizon becomes a drone, and the scene’s color temperature shapes the scale.

**Live:** <https://tomkyser.github.io/scan-line-instrument/>

-----

## How It Works

The core idea is simple: a fixed vertical line in the camera frame acts as a one-dimensional sensor. As you walk or pan, the world scrolls past it. The system samples that column of pixels every frame (~30fps) and maps what it sees to sound.

### The Mapping

|Visual Property                                    |Musical Parameter                                                     |
|---------------------------------------------------|----------------------------------------------------------------------|
|**Y-position** in the column                       |**Pitch** — top of frame = high notes, bottom = low notes             |
|**Luminance change** (object enters/leaves column) |**Note energy** — louder when there’s contrast                        |
|**Sharp edge crossing** (high frame-to-frame delta)|**Pluck transient** — bell/kalimba one-shot                           |
|**Hue** (warm vs cool colors)                      |**Scale subset** — warm colors bias low intervals, cool bias high     |
|**Saturation**                                     |**Timbre brightness** — vivid = harmonically rich, grey = pure        |
|**Horizon line position**                          |**Drone pitch** — subtle modulation as terrain undulates              |
|**Scene color temperature** (rolling average)      |**Scale selection** — warm scenes → major, cool → minor               |
|**Contrast sharpness** (depth proxy)               |**Volume + filter** — sharp/close = loud+bright, soft/far = quiet+dark|
|**Overall scene activity**                         |**Drone behavior, pad swells, glow intensity**                        |

### The Scale

The base scale is **Bb major pentatonic** (F, Bb, C, D, F, G across 4 octaves). When adaptive scale is enabled and the scene cools down (blue/grey dominant), it morphs toward **F minor pentatonic** by gliding D→Eb and G→Ab. The transition takes ~5-10 seconds to settle — slow enough to be felt, not heard as a key change.

-----

## Getting Started

### Requirements

- A phone with a rear camera (iPhone, Android)
- Safari (iOS) or Chrome (Android)
- **HTTPS required** — camera access won’t work over plain HTTP
- Headphones recommended for stereo imaging

### Running It

**Option A: GitHub Pages (recommended)**
Visit the live URL. Tap **Begin**. Grant camera access. Point and walk.

**Option B: Local**

```bash
# Clone and serve locally
git clone https://github.com/tomkyser/scan-line-instrument.git
cd scan-line-instrument
python3 -m http.server 8080
```

Then open `https://your-local-ip:8080` from your phone on the same wifi. Note: iOS Safari may require HTTPS even on LAN — you can use [mkcert](https://github.com/FiloSottile/mkcert) for a local cert if needed.

### First Use

1. Tap **Begin** — the camera opens with a magenta scan line overlaid
1. Point at a scene with some visual variety (not a blank wall)
1. **Walk slowly** or pan the phone at a walking pace
1. Objects crossing the scan line will trigger notes
1. Use headphones to hear the stereo field and reverb tail

**Tip:** The system takes 2-3 seconds to build a background model. If everything fires at once on startup, just hold still for a moment and let it calibrate, then start moving.

-----

## Settings Guide

Tap the **⚙** gear icon (top-right) during a session to open the settings panel.

### Scene

**Indoor Mode** — Switches the vision pipeline from background-model deviation (outdoor, expects sky/ground/horizon) to edge-contrast detection (indoor, expects clutter). Disables horizon tracking and replaces the drone source with vertical energy center-of-mass. Also increases smoothing to handle close-range parallax. Turn this on when you’re inside.

**Multi-Line** — Adds two flanking scan lines at 28% (left, blue) and 88% (right, warm) of frame width. The 15 voices split into three groups: low register (F2-G3) driven by the left line, mid register (Bb3-G4) by center, high register (Bb4-F5) by right. Objects crossing the frame produce a cascade as they hit each line in sequence, creating natural counterpoint.

**Depth Sense** — Estimates object distance from contrast sharpness at the scan column. Sharp, high-contrast edges (close objects) play louder with brighter filters. Soft, low-contrast edges (distant objects) play quieter with darker, warmer filters. This approximates the distance-to-volume relationship that makes outdoor scenes sound spatial.

### Sound Engine

**Drone Mode** (4-way selector)

- **Classic** — Two detuned triangle oscillators at constant gain. Steady foundation.
- **Reactive** (default) — Same oscillators but gain inversely tracks scene activity. Swells when the scene is sparse, ducks when voices are busy. Prevents mix crowding.
- **Texture** — No pitched tone. Brown noise through a bandpass filter that breathes slowly. Atmospheric rather than harmonic. Good for indoor/ambient use.
- **Off** — Silence the drone entirely.

**Color Mode** — When enabled, the scan column is decomposed into HSL. Hue determines which notes in the scale get energy-boosted (warm → low, cool → high). Saturation modulates FM synthesis depth (vivid = harmonically bright, grey = pure). Turn off for purely luminance-based behavior.

**Pluck Transients** — When a sharp contrast edge enters the scan column (high frame-to-frame energy delta), a one-shot kalimba/harp pluck fires on top of the sustained voice. Three-partial sine synthesis with pitch micro-bend, sweeping filter, and a subtle noise transient. Per-zone cooldown prevents machine-gunning. Turn off for a smoother, pad-only sound.

**Drone** — Master on/off for whichever drone mode is selected.

**Delay** — Ping-pong delay (375ms/500ms) with lowpass-filtered feedback. Creates rhythmic echoes from sparse events. Feeds into the reverb for washed-out trails. Turn off for a drier, more immediate sound.

**Pad Layer** — Stacked sine tones (root, fifth, octave, twelfth) that swell gently with overall scene energy. Adds harmonic body underneath everything. Subtle but noticeable when toggled.

**Adaptive Scale** — Tracks the scene’s color temperature over a ~10-second rolling window. Warm scenes (golden hour, incandescent light, reds/oranges) hold the major pentatonic. Cool scenes (overcast, blue light, greens) drift toward minor pentatonic. The shift is a smooth glide, not a jump.

### Mix

**Reverb Send** — Controls how much signal feeds the convolver reverb (3.5s generated impulse response). Higher = more wash and sustain. Lower = drier and more articulate.

**Sensitivity** — Controls the energy threshold and gain multiplier for the vision pipeline. Higher = reacts to subtle motion and low-contrast edges. Lower = requires bold contrast to trigger. If you’re getting too much noise, pull this down. If it feels dead, push it up.

**Pluck Volume** — Independent gain for pluck transients relative to the sustained voices.

-----

## Recording

Tap the **REC** button (bottom-right) to start recording. A pulsing red dot and elapsed timer appear. Tap again to stop and save.

- **Video + audio** capture when supported (mp4 on Safari, webm on Chrome)
- Falls back to **audio-only** if video recording isn’t available
- The scan line glow, flanking lines, and note spark effects are composited into the recorded video
- **60-second maximum** with auto-stop
- On iOS, the native **share sheet** opens for saving to Camera Roll, AirDrop, etc.
- On other platforms, a download link is generated

**Note:** The HTML overlay elements (note labels, settings panel) are not included in the recording — only canvas-rendered effects. The recorded video shows the camera feed with glow effects and scan lines.

-----

## Audio Architecture

```
                    ┌──────────────────────────────────────┐
                    │          15 FM Voices                 │
                    │  (carrier + mod + shimmer + fifth)    │
                    │  StereoPanner per voice               │
                    └───────────┬───────────────────────────┘
                                │
     ┌──────────┐     ┌────────▼────────┐     ┌──────────┐
     │  Drone   │────▶│    Dry Bus      │◀────│   Pad    │
     │ (3-mode) │     │   (0.45)        │     │  Layer   │
     └──────────┘     └──┬──────────┬───┘     └──────────┘
                         │          │
                    ┌────▼───┐ ┌────▼─────┐
                    │ Reverb │ │  Delay   │
                    │ (3.5s) │ │ (ping-   │
                    │  IR    │ │  pong)   │
                    └───┬────┘ └────┬─────┘
                        │           │
                    ┌───▼───────────▼───┐
                    │    Master Bus     │
                    │     (0.50)        │
                    └────────┬──────────┘
                             │
                    ┌────────▼──────────┐
                    │   Compressor /    │──────▶ destination
                    │   Limiter        │──────▶ recDest
                    │  (12:1, -24dB)   │
                    └───────────────────┘
```

### Voice Design

Each of the 15 voices is an **FM synthesis** chain: a sine carrier frequency-modulated by a sine at a harmonic ratio (1x for low notes, 2x for mid, 3x for high). FM depth scales with energy — quiet notes are glassy pure sines, loud notes bloom with harmonics. On top of that, each voice has a detuned octave shimmer and a quiet fifth partial for richness.

### Pluck Design

Plucks are spawned as fresh short-lived oscillators (not pooled). Three sine partials — fundamental with a micro pitch-bend on attack (starts 0.8% sharp, settles over 60ms), detuned octave, and quiet third partial. Two-stage envelope: quick initial drop to 45% then a long exponential tail (1.4-2.0s). A sweeping lowpass filter goes from bright to warm over the decay. A tiny (20ms) bandpass-filtered noise burst adds the initial “tick” texture at 15% volume. Plucks route 55% to reverb for a wet quality.

### Drone Modes

- **Classic/Reactive**: Two detuned triangle oscillators through a shared lowpass with LFO
- **Texture**: Brown noise (integrated white noise) through bandpass + lowpass with slow LFO breathing
- All modes share an output filter and the same pitch-tracking logic

-----

## Vision Pipeline

### Outdoor Mode

1. **Horizon detection** — Scans the safe zone band (middle 44% of frame) across ~40 columns. Finds the row with the strongest luminance gradient. Smoothed with EMA (0.91).
1. **Column sampling** — 5-pixel-wide band at the scan line position. For each of the 15 vertical zones: compute mean luminance, standard deviation (texture), deviation from background model, and frame-to-frame temporal delta.
1. **Background model** — Each zone maintains a slowly-adapting luminance average (rate 0.008). Energy is measured as deviation from this background, so static scenes produce no sound.
1. **Energy formula**: `(stddev × 0.4 + bgDeviation × 0.6 + temporalDelta × 1.2) × sensitivity - threshold`

### Indoor Mode

1. **No horizon** — Disabled entirely. Drone pitch driven by vertical center-of-mass of energy.
1. **Edge contrast** — Instead of comparing each zone to its own background history, computes the luminance gradient between adjacent zones. Object boundaries trigger; flat surfaces don’t.
1. **Competitive normalization** — Only the top 5 most energetic zones get full output. The rest are suppressed to 15%. Forces musical sparsity in cluttered scenes.
1. **Heavier smoothing** — Higher EMA coefficients absorb close-range parallax. Doubled pluck cooldown.

### Depth Estimation

Measures the average absolute luminance gradient between consecutive pixel rows within each zone at the scan column. High gradient = sharp edges = close objects. Low gradient = smooth/blurry = distant. Normalized to 0-1 range. Used as a per-voice gain and filter multiplier.

### Multi-Line

Three independent scan columns sampled per frame. The flanking lines use a lighter-weight sampler (single pixel width, no color analysis, no background model — just stddev + temporal delta). Voice groups are blended 70% from their assigned line and 30% from center for coherence.

-----

## Tech Stack

- **Zero dependencies** — vanilla HTML/CSS/JS, no build step, no frameworks
- **Web Audio API** — all synthesis, effects, and routing
- **getUserMedia** — rear camera access
- **Canvas 2D** — frame processing (320×240), display rendering, glow effects
- **MediaRecorder** — video+audio capture
- **~1400 lines** in a single `index.html`

### Browser Support

|Browser       |Status                                      |
|--------------|--------------------------------------------|
|Safari iOS 15+|✅ Full support (primary target)             |
|Chrome Android|✅ Full support                              |
|Safari macOS  |✅ Works (rear camera becomes webcam)        |
|Chrome desktop|✅ Works                                     |
|Firefox       |⚠️ getUserMedia works, MediaRecorder may vary|

### Performance

The processing canvas is fixed at 320×240 regardless of device resolution. Vision computation targets <5ms per frame. Audio scheduling uses `setTargetAtTime` for smooth interpolation without per-sample callbacks. On an iPhone 12+ the full pipeline (vision + 15 FM voices + drone + delay + reverb + plucks + glow canvas) runs comfortably at 30fps.

-----

## Project Structure

```
scan-line-instrument/
├── index.html      # Everything — UI, audio engine, vision pipeline, recording
└── ROADMAP.md      # Feature roadmap with shipped/planned status
```

-----

## License

MIT

-----

## Inspiration

Inspired by an Instagram reel showing a program that generates music from dashcam footage — a vertical scan line triggering notes as objects cross it, with distance mapping to volume and a drone following the horizon. This project reimagines that concept for handheld mobile use with real-time camera input.
