# metronome
# TempoCoach — Web Audio Metronome

**Live app:** https://djanoska.github.io/metronome/

## 1. Project Overview

TempoCoach is a browser-based metronome and guitar practice tool built as a single self-contained HTML file.

The project goes beyond a basic metronome. It combines tempo control, rhythmic subdivisions, strumming visualization, structured tempo training, timed practice sessions, and a microphone-based guitar timing accuracy experiment.

The primary goal is to support guitar practice by helping the player develop consistent timing, improve picking and strumming control, and gradually increase playing speed.

### Core principles

- No external libraries or frameworks.
- No external audio files.
- No build process or package manager required.
- Runs directly in a modern browser.
- Designed for desktop and mobile screens.
- Hosted using GitHub Pages.
- All application code lives in `index.html`.

## 2. Technology Stack

| Technology | Purpose |
|---|---|
| HTML5 | Application structure and controls |
| CSS3 | Layout, responsive styling, themes, animations |
| Vanilla JavaScript | Application logic and event handling |
| Web Audio API | Audio generation, scheduling, and playback |
| Web Workers | Background scheduling ticks |
| Blob and Worker APIs | Create the scheduler worker dynamically |
| getUserMedia | Access the microphone for guitar timing detection |
| AnalyserNode | Analyze microphone input amplitude |
| localStorage | Persist the selected color theme |
| Screen Wake Lock API | Attempt to prevent the screen from sleeping during practice |
| GitHub Pages | Static hosting and deployment |

There are no npm dependencies, external scripts, or separate source modules.

## 3. Application Architecture

The application is divided into three logical layers within the same HTML file.

### HTML — Interface

Defines the metronome, tempo controls, practice settings, session timer, beat indicators, guitar timing detector, and settings modal.

### CSS — Presentation

Contains all styles, including:

- Dark, light, and system themes.
- Responsive layouts.
- Active beat and strumming indicators.
- Tempo and bar-turnaround visual feedback.
- Button states, modal styling, and microphone level meter.

### JavaScript — Logic

Contains the audio engine, microphone detector, application state, user interaction handlers, visual updates, and settings persistence.

#### Main JavaScript components

**`MetronomeEngine`**

Responsible for tempo, audio initialization, synthesized sound buffers, beat scheduling, practice modes, BPM progression, session timing, and playback lifecycle.

**`GuitarDetector`**

Responsible for microphone access, input filtering, amplitude analysis, strum-onset detection, timing comparison, and accuracy statistics.

**`renderInd()`**

Rebuilds the visual beat indicators based on the selected time signature, subdivision, and strumming pattern.

**`updateVisuals()`**

Runs through `requestAnimationFrame`, processes scheduled beat events, updates active indicators, and triggers the visual bar-start pulse.

**`bindGroup()`**

Provides shared event handling for groups of mutually exclusive buttons.

**`setTheme()`**

Updates the selected theme and persists the choice in localStorage.

## 4. Metronome Features

### 4.1 Tempo Control

- BPM range: 1–240.
- Editable numeric BPM field.
- Slider-based tempo control.
- Increment and decrement buttons.
- Clickable tempo scale markers.
- Automatic Italian tempo terminology labels, such as Largo, Adagio, Andante, Moderato, Allegretto, Allegro, Vivace, and Presto.

Changing the tempo updates the engine and synchronizes the numeric field, slider, and tempo label.

### 4.2 Time Signatures and Emphasis

Available options:

- 2/4
- 3/4
- 4/4
- 6/8
- 7/8
- None

The selected signature determines the number of beats per bar, except for `None`, which disables the first-beat emphasis while retaining a four-beat bar internally.

**Implementation note:** 6/8 is currently represented as six regular beat positions per bar. It does not implement a dedicated compound-meter pulse model with two dotted-quarter beats.

### 4.3 Rhythmic Subdivisions

Four subdivision settings:

- Quarter notes — 1 step per beat.
- Eighth notes — 2 steps per beat.
- Triplets — 3 steps per beat.
- Sixteenth notes — 4 steps per beat.

The subdivision controls the number of scheduled events per beat and the corresponding visual indicators.

### 4.4 Sound Presets

Five synthesized sound presets:

1. Digital Beep
2. Woodblock
3. Drum Kit
4. Mechanical
5. Cowbell & Clave

Sound buffers are generated in JavaScript using short waveforms, oscillators represented by sine functions, exponential decay, and, for some presets, generated noise.

No prerecorded sound assets are required.

The first beat of a bar uses the primary sound buffer; other beat positions and subdivisions use alternate buffers.

### 4.5 Volume and Mute

- Volume slider: 0–100%.
- Default volume: 80%.
- Mute toggle.
- Gain is controlled through a Web Audio `GainNode`.

Mute suppresses audible metronome clicks but does not stop the scheduler or visual timing system.

### 4.6 Tap Tempo

The TAP button estimates BPM from successive button presses.

- Uses `performance.now()` for timestamping.
- Keeps a rolling window of up to four tap timestamps.
- Calculates the average interval between taps.
- Resets the sequence if the gap between consecutive taps exceeds two seconds.
- Updates the main BPM control.

## 5. Guitar Practice Tools

The practice controls are grouped inside a collapsible panel.

### 5.1 Strumming Direction

Two visualization patterns are available:

- **Alternate:** alternating downstroke and upstroke arrows.
- **Down only:** downstroke arrows for all positions.

These arrows are visual practice guides. They do not generate separate strumming sounds.

### 5.2 Training Wheels

After 30 seconds of playback, the engine suppresses subdivision clicks while retaining the main beat clicks.

The goal is to encourage the player to maintain the subdivision internally instead of relying on every metronome click.

The elapsed time is measured from the start of the metronome session.

### 5.3 Gap Click

Mutes the metronome for every fourth bar, using the internal bar counter.

The player must maintain the pulse during the silent bar and reconnect with the metronome when it resumes.

The mode resets its bar counting when playback starts.

### 5.4 Random Drop

Randomly suppresses approximately 18% of eligible scheduled clicks.

The first beat of a bar is excluded from random dropping. The purpose is to challenge internal timekeeping while retaining periodic rhythmic reference points.

### 5.5 Automatic BPM Ladder

Configurable parameters:

- Start BPM.
- Target BPM.
- BPM increment.

The ladder changes tempo after every four completed bars.

Example:

- Start: 80 BPM.
- Target: 130 BPM.
- Step: 5 BPM.

The intended progression is 80 → 85 → 90 BPM and so on, ending at the target tempo.

### 5.6 Pyramid Mode

When enabled together with the BPM ladder, the tempo reverses direction upon reaching the target BPM and descends toward the starting BPM.

The direction then reverses again, allowing repeated ascending and descending practice.

Pyramid mode is a modifier of the ladder rather than an independent scheduler.

## 6. Session Timer

Available durations:

- OFF — unlimited session.
- 5 minutes.
- 10 minutes.
- 15 minutes.

The countdown starts when playback begins and is updated by the scheduler's timer checks.

When the countdown expires:

1. Playback stops.
2. A synthesized completion chime plays.
3. The Start button and visual indicators are reset.
4. The interface displays a completion message.

The timer is session-based and is not persisted between page reloads.

## 7. Guitar Timing Accuracy — BETA

This experimental feature attempts to measure how closely detected guitar strums align with scheduled metronome events.

### 7.1 Microphone Input

The detector uses `navigator.mediaDevices.getUserMedia()` to access the microphone.

The audio input passes through the following processing chain:

`Microphone → GainNode → High-pass Filter → Low-pass Filter → AnalyserNode`

Current settings:

- Input gain: 2.5.
- High-pass filter: 65 Hz.
- Low-pass filter: 3,000 Hz.
- Analyser FFT size: 512.
- Analysis: 512-sample time-domain RMS amplitude.

Echo cancellation, noise suppression, and automatic gain control are requested to be disabled.

### 7.2 Strum Detection

The detector calculates RMS amplitude from successive audio samples.

When the signal exceeds the configured threshold, a strum onset is detected. A short release condition and a 130 ms retrigger interval reduce repeated detections from the same strum.

The sensitivity slider adjusts the amplitude threshold:

- Low: less sensitive to quiet signals.
- High: more sensitive to quiet signals and potentially more susceptible to background noise.

### 7.3 Timing Evaluation

Detected strums are compared against recently scheduled metronome events.

The configurable offset is applied to the detected timestamp before comparison.

Default offset: −60 ms.

A hit is evaluated only when the nearest scheduled event is within 240 ms and has not already been evaluated.

Classification:

- **Perfect:** absolute timing difference is 45 ms or less.
- **Early:** more than 45 ms early.
- **Late:** more than 45 ms late.

The displayed accuracy percentage is the percentage of evaluated hits classified as Perfect.

### 7.4 Limitations

This is an experimental amplitude-based onset detector, not a guitar transcription system or pitch detector.

Potential sources of inaccuracy include:

- Microphone and operating-system latency.
- Room acoustics and background noise.
- Distortion and sustained guitar notes.
- Multiple strums occurring close together.
- Acoustic feedback and audible metronome clicks entering the microphone.
- The choice of offset and sensitivity settings.

For more reliable testing, use headphones to minimize metronome bleed into the microphone.

The accuracy score should be treated as an approximate practice aid rather than a calibrated measurement instrument.

Microphone access generally requires a secure browser context, such as HTTPS or localhost, and explicit user permission.

## 8. Settings and Persistence

### Persisted setting

The selected color theme is stored under:

`pb_theme`

Supported values:

- `system`
- `light`
- `dark`

System mode follows the browser's preferred color scheme.

### Non-persisted settings

The current implementation does not persist BPM, volume, time signature, subdivision, sound preset, practice modes, ladder parameters, timer selection, or microphone calibration settings.

These settings return to their HTML or JavaScript defaults after a page reload.

The guitar detector's running state and session statistics are also transient.

## 9. Audio Scheduling and Timing

The metronome uses the Web Audio API for playback timing.

### Scheduling model

The engine maintains a `nextNoteTime` value in the audio context's time domain.

A scheduler checks whether upcoming events fall within a 120 ms look-ahead window. Events inside that window are scheduled directly on the audio context timeline.

The scheduler advances by:

`60 / BPM / subdivision`

At the end of each bar, the engine updates its bar counter and checks whether the BPM ladder should advance.

### Scheduler worker

A dynamically created Web Worker emits scheduler ticks every 25 ms.

If worker creation fails, the engine falls back to a main-thread `setInterval()` scheduler.

The worker provides regular scheduling opportunities; actual audio event timing is handled by the Web Audio API.

### Visual synchronization

The interface uses `requestAnimationFrame()` to process queued events and update the visual indicators.

Consequently, audio scheduling and screen animation operate on separate timing mechanisms. Visual indicators may not align perfectly with audio at the display-frame level.

## 10. Browser APIs and Runtime Requirements

- Modern browser with Web Audio API support.
- JavaScript enabled.
- Audio output device.
- HTTPS or localhost for microphone access.
- Microphone permission for guitar timing detection.

Screen Wake Lock support is optional. The application attempts to request a screen wake lock during playback and release it when playback stops.

The application also installs a global error handler that displays JavaScript error information in an on-screen banner.

## 11. File Structure

The repository intentionally uses a single-file architecture.

```text
metronome/
├── index.html    # Complete application
└── README.md     # Project and technical documentation
```

`index.html` contains:

- HTML markup.
- Embedded CSS in a `<style>` element.
- Embedded JavaScript in a `<script>` element.
- Audio synthesis and scheduling logic.
- Microphone detection logic.
- All UI event handlers.

There is no build step, bundler, package manifest, or separate asset directory.

## 12. Development and Deployment

### Local development

1. Clone or download the repository.
2. Open `index.html` in a modern browser.
3. Test metronome playback and settings.
4. For microphone testing, use localhost or another secure context.

A local static HTTP server can be used when testing browser APIs or behavior that differs from direct file access.

### Deployment

GitHub Pages serves the static application directly from the repository.

There is no compilation or packaging stage. Changes to `index.html` can be deployed through the configured GitHub Pages publishing workflow.

### Testing checklist

After changing the code, manually verify:

- BPM limits and synchronization across controls.
- Tempo changes during playback.
- Time signatures and subdivision indicators.
- All five sound presets.
- Volume and mute behavior.
- Tap tempo calculations.
- Training Wheels timing.
- Gap Click bar counting.
- Random Drop behavior.
- BPM ladder and Pyramid direction changes.
- Timer expiration and completion chime.
- Microphone permissions, sensitivity, and offset.
- Theme selection and persistence.
- Desktop and mobile layouts.

No automated test suite is currently included in the repository.

## 13. Known Limitations and Future Improvements

Potential areas for further development:

- Persist user preferences beyond the theme.
- Improve microphone onset detection and calibration.
- Add visual feedback for timing distribution, not only the Perfect percentage.
- Improve handling of compound time signatures such as 6/8.
- Add automated tests for scheduling and practice-mode state transitions.
- Improve lifecycle cleanup for audio resources, animation loops, and worker resources.
- Improve accessibility, keyboard navigation, and screen-reader labels.
- Consider splitting the application into separate modules if the codebase grows significantly.

These are potential improvements, not commitments to a particular roadmap.

## 14. AI Handover Notes

When using an AI assistant to modify this project, follow these guidelines:

1. **Read the entire `index.html` before making changes.** The UI, audio engine, detector, and event handlers are tightly coupled.
2. Preserve the single-file architecture unless explicitly asked to refactor it.
3. Do not introduce frameworks, dependencies, build tooling, or external assets without approval.
4. Keep HTML element IDs and their corresponding JavaScript references synchronized.
5. Treat audio scheduling and visual rendering as separate systems.
6. Preserve the Web Audio look-ahead scheduling model when modifying tempo or timing behavior.
7. Check interactions between practice modes, particularly BPM Ladder, Pyramid, Gap Click, and Training Wheels.
8. Do not describe microphone timing results as exact measurements. The detector uses amplitude-based onset detection and configurable latency compensation.
9. Distinguish implemented behavior from proposed improvements.
10. After each change, provide a concise summary of the modifications and a manual regression-testing checklist.

### Project status

TempoCoach is a functional browser-based metronome with additional guitar practice tools. The microphone-based Guitar Timing Accuracy feature is explicitly experimental and should be evaluated separately from the core metronome.

The current implementation is contained in `index.html`; this file is the authoritative source for application behavior.

---

**End of README**
