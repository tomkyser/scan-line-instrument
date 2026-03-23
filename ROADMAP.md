# Scan Line Instrument — Feature Roadmap

## Status Key

- 🟢 Shipped
- 🔵 In Progress
- ⚪ Planned
- 💭 Idea / Exploratory

-----

## v1 — Foundation (Shipped)

🟢 Rear camera capture via getUserMedia
🟢 Vertical scan line column sampling at fixed x-position
🟢 Luminance-based energy detection per zone
🟢 Background model with slow adaptation (rejects static scenes)
🟢 Horizon detection within safe zone band (full-width gradient scan)
🟢 Smoothed horizon → drone pitch modulation
🟢 Bb major / F mixolydian scale quantization
🟢 Triangle oscillator voices with per-zone gain mapping
🟢 3-osc detuned drone with sub-octave and LFO filter sweep
🟢 Convolver reverb (generated IR)
🟢 Compressor on master bus
🟢 Note energy indicator bars along scan line
🟢 GitHub Pages deployment (iOS Safari compatible)

## v2 — Sound Design & Sensitivity (Shipped)

🟢 FM synthesis voices (carrier + modulator, depth scales with energy)
🟢 Ghost harmonics per voice (octave shimmer + detuned fifth)
🟢 Ping-pong delay (375ms/500ms, LP-filtered feedback, feeds reverb)
🟢 Drone sub-octave sine
🟢 Drone filter + tremolo respond to global energy
🟢 Ambient pad layer (stacked root/fifth/octave, swells with scene energy)
🟢 Temporal differencing (frame-to-frame luminance delta per zone)
🟢 Multi-column sampling (5px wide scan band)
🟢 Lower energy threshold (12 → 4), faster attack (80ms → 40ms), longer release (800ms)
🟢 Glow canvas overlay (scan line radiation + per-note spark dots)
🟢 Filtered reverb IR for smoother tail (3.5s duration)

## v3 — Color, Transients & Settings (Shipped)

🟢 Color-aware pitch mapping

- Scan column decomposed to HSL per zone
- Hue biases energy toward warm (low intervals) or cool (high intervals) via per-note affinity weights
- Saturation modulates FM depth (vivid = harmonically bright, grey = pure/muted)
- Visual indicators and glow dots tinted by zone hue (warm → orange, cool → cyan)
- Smoothed hue/saturation tracking (EMA 0.8) to avoid jitter

🟢 Velocity-triggered pluck voices (kalimba/harp style)

- Sharp energy onset (delta > 0.15) fires one-shot pluck
- 3-partial sine synthesis: fundamental (w/ micro pitch bend), detuned octave, quiet third partial
- Two-stage decay envelope: quick initial drop then long tail (1.4-2.0s)
- Sweeping LPF (bright attack → warm sustain)
- Subtle noise texture layer (15% vol, 20ms bandpass burst)
- Wet routing: 55% reverb / 45% dry, feeds delay
- Per-zone cooldown (8 frames / ~270ms) prevents machine-gunning
- Spawns fresh oscillators per pluck (auto-cleanup pool)

🟢 Feature toggle settings panel

- Slide-out panel (gear icon, top-right)
- Toggles: Color Mode, Pluck Transients, Delay, Pad Layer
- Sliders: Reverb Send, Sensitivity, Pluck Volume
- All settings read live per frame (no restart required)
- Touch scroll exemption so sliders work on iOS
- Sensitivity slider controls both energy threshold (2-10) and multiplier (0.3-1.7x)

## v4 — Indoor Mode (Shipped)

🟢 Scene mode toggle (Outdoor / Indoor) in settings panel

🟢 Indoor energy detection

- Adjacent-zone luminance gradient replaces background-model deviation
- Object boundaries trigger, flat surfaces suppress
- Competitive normalization: top 5 zones get full energy, rest suppressed to 15%
- Temporal differencing retained but weighted lower (0.8x vs 1.2x outdoor)
- Lower threshold floor (1-6 vs 2-10 outdoor)

🟢 Indoor drone source

- Horizon detection disabled in indoor mode
- Vertical energy center-of-mass drives drone pitch (heavy EMA 0.97)
- Visual guide line turns cyan and tracks CoM instead of horizon
- Safe zone guides hidden in indoor mode

🟢 Indoor temporal tuning

- Higher smoothing coefficients (attack EMA 0.65 vs 0.55, decay 0.96 vs 0.94)
- Doubled pluck cooldown (16 frames / ~530ms)
- Pluck threshold raised (0.18 vs 0.15)
- Drone pitch tracking slowed (time constant 0.5s vs 0.3s)

🟢 Drone toggle

- Independent on/off in settings panel (defaults on)
- Smooth fade in (300ms) / fade out (150ms), no clicks

🟢 Gain staging overhaul (v3.1 hotfix)

- Compressor → limiter (ratio 12:1, threshold -24dB, knee 6)
- Master bus, dry bus, per-voice, drone, pluck, pad, reverb/delay sends all reduced
- Relative balance preserved, absolute levels sit below limiter ceiling
- Eliminates distortion during dense multi-voice activation

## v5 — Spatial, Feel & Drone Rework (Shipped)

🟢 Stereo field from y-position

- StereoPannerNode per voice, panned -0.35 (lowest) to +0.35 (highest)
- Drone and pad remain center-routed
- Complements pseudo-stereo from ping-pong delay

🟢 Adaptive scale selection

- Scene warmth (0-1) computed from average hue warmness of saturated zones
- Heavy EMA (0.993) — takes ~5-10 seconds to settle, no jarring jumps
- Warm scenes stay Bb major pentatonic (F, G, Bb, C, D)
- Cool scenes morph toward F minor pentatonic (D→Eb, G→Ab via coolFreq lerp)
- Voice retuning via setTargetAtTime with 0.8s time constant (glide, not jump)
- Plucks use current tuned frequencies
- Note labels update to reflect shifted pitch names
- Toggle in settings panel (defaults on)

🟢 3-mode drone engine (replaces old single-mode drone)

- Classic: 2 detuned triangles, constant gain, no sub (lighter footprint)
- Reactive (default): same oscillators but gain inversely tracks globalEnergy
  - Swells when sparse, ducks when voices fire (sidechain-style without sidechain)
  - Floor at 0.08 to never fully disappear
- Texture: brown noise through bandpass (1.5x drone pitch) + lowpass
  - Slow LFO (0.06 Hz) breathes bandpass center
  - LP opens with scene warmth + energy
  - Inverse energy gain (floor 0.15)
- Off: all modes fade to zero in 150ms
- Segmented control UI (4-way: Classic / Reactive / Texture / Off)
- All modes share output LPF with LFO and pitch-tracking logic
- Removed sub-octave oscillator and tremolo (headroom savings)

## v6 — Capture & Share (Shipped)

🟢 Recording / export

- MediaStreamDestination tapped from compressor output
- Canvas captureStream(30) for video track
- Combined audio+video MediaRecorder (auto-detects best mime: mp4 > webm)
- Falls back to audio-only if video recording unsupported
- During recording, glow overlay + scan line composited onto camera canvas for capture
- REC button with pulsing red dot indicator + elapsed timer
- 60-second max with auto-stop
- iOS share sheet integration via navigator.share (files API)
- Fallback to download link for non-share-capable browsers
- Stop button safely terminates active recording
- Chunks collected every 1s for progressive capture

## v7 — Advanced Vision (Shipped)

🟢 Multi-scan-line mode

- 3 scan lines: Left (0.28), Center (0.62), Right (0.88)
- Voices split into groups: L=low 5 (F2-G3), M=mid 5 (Bb3-G4), R=high 5 (Bb4-F5)
- Each group primarily fed by its corresponding scan line (70/30 blend with center)
- Flanking energies independently smoothed
- Visual: blue-tinted left line, warm-tinted right line, both with energy-reactive glow
- Glow dots render on their respective scan line positions
- Toggle in settings panel (defaults off)
- Recording composite includes flanking lines
- CSS class body.multi-line shows/hides secondary line DOM elements

🟢 Depth estimation from contrast sharpness

- Per-zone vertical gradient magnitude at scan column = sharpness proxy
- Sharp (high gradient) = close → louder (0.5-1.2x gain), brighter filter (0.6-1.4x cutoff)
- Blurry (low gradient) = far → quieter, darker/warmer filter
- Pluck velocity also scaled by depth
- Smoothed per-zone (EMA 0.85) to avoid jitter
- Visual: note bar width and glow radius modulated by sharpness
- Toggle in settings panel (defaults on)

💭 Motion vector estimation (deferred — high cost, subtle payoff)
💭 Object persistence / tracking (deferred — complex for incremental benefit)

-----

## Technical Debt / Quality

⚪ Performance profiling on older iOS devices
⚪ Orientation lock handling (landscape vs portrait behavior)
⚪ Wake lock to prevent screen dimming during use
⚪ Accessibility considerations for visual overlay elements
⚪ Error recovery if audio context is interrupted (phone call, etc.)
