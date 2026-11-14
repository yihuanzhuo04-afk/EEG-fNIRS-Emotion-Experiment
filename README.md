# AffectLab v5.0.0 – Audiovisual EEG–fNIRS Emotion Paradigm

AffectLab is a single-file browser application for piloting the stimulus side of a prefrontal EEG–fNIRS study of acute aerobic exercise and expressive suppression. Version 4.0.0 adds paired audio to the v3.0.0 visual paradigm. The v3 architecture, settings, session storage key (`affectlab-sessions-v1`), default timing and both CSV exports are kept. Existing v3 column names are kept, and new columns are added alongside them.

| Display code | Condition | Legacy code (CSV `condition`, image bank) | Event-code offset |
| --- | --- | --- | --- |
| NEU-NAT | Neutral natural exposure | `WNeu` | 1 |
| NEG-NAT | Negative natural exposure | `WNeg` | 2 |
| NEG-SUP | Negative expressive suppression | `ES` | 3 |

## What changed in v5.0.0

- **New look**: refreshed operator interface (colour, cards, buttons), a segmented AV / VO / AO modality switch, and a proportional trial timeline that shows the relative length of each phase. The participant screen is calmer: a neutral dark background, a drawn fixation cross, operator header and footer dimmed until hovered, and round 1–5 rating buttons.
- **Audio-only (AO) mode** for the pilot comparison of AO, VO and AV. AO uses the same image–sound pairs as AV but plays only the sound over the fixation cross (the image is not shown). Stimulus onset is logged as `stimulus_onset` (codes 21–23, the same offsets as `image_onset`).
- **Audio onset within the stimulus window** (`Audio Onset within Stimulus (s)`, default 0). For example, 1 s centres a 6 s IADS-E clip in an 8 s window. The duration check now reports clips that run past the window or end more than 0.5 s early.
- **Two new trial-CSV columns** (64 in total): `imageShown` and `audioOnsetDelaySec`. `modality` is `av`, `visual` or `audio`.
- Participant instructions are worded for each modality (watch and listen / view / listen).

v4 data remain readable; all v4 columns, event codes and the session storage key are unchanged.

## What changed in v4.0.0 (from v3.0.0)

- **Modality setting**: *Audiovisual (paired image + audio)* (default) or *Visual only* (v3 behaviour).
- **Audio pairing**: by a pairing manifest (CSV), or by an identical file stem in the same set and condition (`NEG_01.png` ↔ `NEG_01.wav`). Before the session, the app lists conditions with too few pairs and pairs whose durations differ by more than 0.5 s.
- **Playback**: Web Audio API. All clips are decoded and preloaded before the first trial, and RMS and peak levels are logged per clip. One fixed gain applies to the whole session. At stimulus offset the sound fades out (default 50 ms), so no clip runs into the next phase. Only one clip can play at a time.
- **Level check**: before an audiovisual session can start, a 1 kHz reference tone (2 s, −20 dBFS) must be played at the session gain. The tone is required again after any gain change. The browser cannot measure sound pressure, so set the system volume and headphones to the approved level with a sound level meter.
- **Autoplay policy**: the audio context is started by the operator's click (reference tone or Start); no audio plays without a user gesture.
- **Failure handling**: if a clip is missing, fails to decode or fails to play, the trial continues visually. An `audio_error` event (code 90) is logged, the row gets `audioStatus` ≠ `ok` and `audioValid = false`. Nothing fails silently.
- **Optional STOP cue** (`STOP Cue (seconds; 0 = off)`, default 0), for the alternative timing considered earlier. The default timing does not change.
- **Practice flag** (`isPractice`), **photodiode patch** (48 px corner square, white during the stimulus) and a numeric **event code** for every event.
- **Validity flags**: `trialStatus`, `ratingValid` and `audioValid`. Missing ratings stay empty and are never filled in.
- **Marker hook**: if the page defines `window.AffectLabMarker = function ({code, event, trial, elapsedMs}) {…}`, it is called synchronously for every logged event, and `markerHookCalled` records whether the call succeeded. No acquisition integration is included.

## Run a demonstration

1. Serve this folder over HTTP: `python -m http.server 8000`, then open `http://localhost:8000/` in desktop Chrome or Edge. Opening `index.html` from the file system also works, but storage behaves per file path.
2. Enter a test ID such as `DEMO-01`, then choose the day, Exercise/Rest, Pre/Post and set A–D.
3. Leave **Stimulus Modality** on *Audiovisual*. Tick **Demo Mode** to use generated placeholder images and tones; no import is needed.
4. In **Audio Playback & Level Check**, set the gain and press **Play 1 kHz Reference Tone**. The green confirmation line must appear before Start is enabled.
5. Start, accept the instructions and run the trials. The defaults are 10 trials per condition (30 per assessment) with 2 s fixation, 8 s audiovisual stimulus, 8 s rating and 20 s recovery, giving 38 s per trial and 19 min per assessment, excluding instructions. Use **Skip Trial** or **End Experiment** to test interruption handling.
6. Open **Ratings & Data** and download the trial CSV and the event CSV.

To try real files, import the material in `sample_assets/` in the **Emotion Image Library**. Import `images/` per condition under **Add Stimulus Images**. Import `audio/` per condition under **Add Stimulus Audio**. Optionally load `sample_pairing_manifest.csv` under **Pairing Manifest → Load Manifest CSV**. All sample files are synthetic test assets (labelled images and tones/noise at −26 dBFS RMS) generated for this package. They are not emotional stimuli.

## Pairing manifest

Download the template with **Pairing Manifest → Download Template**. Columns:

```
set,condition,image,audio,category,valence,arousal
A,NEU-NAT,NEU_01.jpg,NEU_01.wav,neutral,,
A,NEG-NAT,NEG_01.jpg,NEG_01.wav,fear,,
A,NEG-SUP,NEG_11.jpg,NEG_11.wav,fear,,
```

`condition` accepts NEU-NAT / NEG-NAT / NEG-SUP (or WNeu / WNeg / ES). `category`, `valence` and `arousal` are optional normative values copied into each trial row (`stimulusCategory`, `normValence`, `normArousal`). They are there to document the matching of NEG-NAT and NEG-SUP clips on category, normative ratings, duration, loudness and onset.

## Data files

**Trial CSV (62 columns, one row per planned trial).**

- Session: `sessionId, appVersion, participant, day, intervention, phase, imageSet, modality, isDemo, isPractice, sessionStatus, sessionStartUtc`
- Stimulus: `trial, condition, conditionCode, imageType, imageFile, imageId, pairId, audioFile, audioId, audioDurationSec, audioRmsDbfs, stimulusCategory, normValence, normArousal`
- Ratings: `valence, arousal, valenceResponseMs, arousalResponseMs` (1–5; empty if not given)
- Timing (ms from session start, `performance.now()` clock): `fixationOnsetMs, imageOnsetMs, visualFrameMs, audioScheduledMs, audioOnsetEstMs, avOffsetEstMs, audioOutputLatencyMs, audioBaseLatencyMs, stimulusOffsetMs, stimulusDurationMs, stopOnsetMs, samOnsetMs, recoveryOnsetMs`
- Status: `audioStatus` (ok / missing / decode_failed / play_error / visual_only), `audioEndedBeforeOffset, visibilityInterrupted, skipped, skippedAtStep, skipTimeMs, imageSizeAtOnsetPercent, trialStatus` (complete / incomplete_rating / skipped / interrupted / visibility_interrupted / not_presented), `ratingValid` (true only if `trialStatus = complete`), `audioValid`
- Settings: `durationFixSec, durationImageSec, durationStopSec, durationSamSec, durationRecoverySec, audioGainPercent, audioFadeMs, audioCheckUtc, photodiodeMarker`

`visualFrameMs` is the timestamp of the first animation frame after the image is inserted. `audioOnsetEstMs` maps the audio context's output timestamp to the same clock, and includes the reported output latency where the browser provides it. `avOffsetEstMs = audioOnsetEstMs − visualFrameMs`. All three are software estimates (see *Timing limits*).

**Event CSV (one row per event).** `sessionId, appVersion, participant, day, intervention, condition, conditionCode, phase, set, isDemo, isPractice, event, eventCode, trial, imageType, imageFile, audioFile, elapsedMs, utcIso, markerHookCalled, detail`

**Event codes.**

| Code | Event | Code | Event |
| --- | --- | --- | --- |
| 1 / 2 / 3 | session start / completed / aborted | 50 | rating screen onset |
| 10 | fixation onset | 51 / 52 | valence / arousal response |
| 21–23 | image onset (+1 NEU-NAT, +2 NEG-NAT, +3 NEG-SUP) | 53 | rating complete |
| 31–33 | audio onset (same offsets) | 60 | recovery onset |
| 40 | stimulus offset (fade starts) | 70 / 71 | trial complete / skipped |
| 45 | STOP cue onset (only if enabled) | 80 / 81 | block start / end |
| 90 | audio error | 0 | informational (preload, instructions, visual frame, visibility change, audio end) |

Code 95 (`audio_check`) is reserved. The reference-tone time is stored in `audioCheckUtc` instead.

Sessions are stored in this browser's localStorage, and images and audio in IndexedDB (`AffectLabImageBank`, stores `pictures` and `sounds`). Nothing is uploaded. Export promptly and keep exports with the approved data-management plan. Use `isDemo` and `isPractice` to filter test rows.

## Timing limits (read before research use)

- All times are **browser software timestamps**. They show when the program issued an event, not when light left the screen or sound left the headphones.
- Display timing depends on the refresh rate, compositor and browser. In a background or occluded tab, the display loop is throttled, and the first frame can be delayed by hundreds of milliseconds (observed in testing; see `VERIFICATION.md`). The timer backstop keeps phase durations correct, but `visualFrameMs` and `avOffsetEstMs` are not valid there. Run studies in a foreground fullscreen window.
- Audio output latency reported by the browser is an estimate, and Bluetooth audio adds variable delay. Use wired headphones.
- **This software does not synchronise EEG or fNIRS hardware.** Before research use, measure visual onset with the photodiode patch and a light sensor, and sound onset by audio loopback. Then confirm that shared markers appear in both acquisition systems.

## Stimuli and licensing

The MIT licence (`LICENSE`) covers the code and the generated demo and sample assets only. Research images and audio (for example IAPS, IADS or other databases) must be obtained under their own licence or permission. Do not commit them to a public repository. Use test IDs on any public deployment.

## Publish on GitHub Pages

Commit `index.html`, `README.md`, `VERIFICATION.md`, `LICENSE`, `.nojekyll` and, optionally, `sample_assets/` to the root of `main`. Then choose **Settings → Pages → Deploy from a branch → main / (root)**. Open the published URL in a private desktop window, run Demo Mode with a test ID and download both CSV files.
