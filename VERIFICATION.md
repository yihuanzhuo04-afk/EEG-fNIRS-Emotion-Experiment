# AffectLab v4.0.0 – Verification checklist

Status as of 11 Nov 2026. "Done" means the check was actually run and its result is recorded below. "Pending" checks have **not** been run. They must pass before the software is used with participants.

**Environment of the recorded tests:** Windows 11. Browser: the Chromium-based browser pane of the Claude desktop application (Chrome 156 engine), served from `python -m http.server`. The pane was in the background during the runs, so its display loop was throttled; this is not representative of a foreground fullscreen study display. Screenshots: headless Microsoft Edge, 2160 × 1350 px.

## A. Functional checks – done

| # | Check | Session | Result |
| --- | --- | --- | --- |
| A1 | Audiovisual session with the six `sample_assets` pairs, paired by manifest; 2 trials per condition; 1/2/0/3/3 s (fixation/stimulus/STOP/rating/recovery) | TEST-02 | 6/6 trials `complete`, `ratingValid = true`; `audioStatus = ok` 6/6; clips decoded as 8 s, −26 dBFS RMS; no clip ended before stimulus offset |
| A2 | Reference-tone check logged before an audiovisual start; manifest values carried into rows | TEST-02 | `audioCheckUtc` recorded before session start; gain 80 %, fade 50 ms in every row; `stimulusCategory` taken from the manifest |
| A3 | Phase durations hold when the display loop is throttled (timer backstop) | TEST-02, TEST-03 | Logged stimulus duration 2007–2017 ms (TEST-02, 6 trials) and 2007–2020 ms (TEST-03, 3 trials) for a 2 s setting |
| A4 | Event codes and condition offsets | TEST-02 | Codes 1, 2, 10, 21–23, 31–33, 40, 50–53, 60, 70, 80, 81 present in order; 80 events |
| A5 | Visual-only mode and early end of a session | TEST-03 | Trials 1–2 `complete`; trial 3 `interrupted`, `ratingValid = false`; trials 4–6 `not_presented`; session `aborted`; `audioStatus = visual_only` |
| A6 | Missing rating and skipped trial are flagged, not filled (pre-release build) | DEMO-01 | Unanswered rating → `incomplete_rating`, empty values; skipped trial → `skipped`, code 71 |
| A7 | Exports | TEST-02 | Trial file with 62 columns, event file with 21 columns; UTF-8 |
| A8 | Console | all | No console errors |
| A9 | Interface rendering | headless Edge | Set-up, audio check, stimulus and rating screens rendered |

## B. Known result that limits interpretation

In TEST-02, the first animation frame followed image insertion by 34–55 ms in three trials, where `avOffsetEstMs` was −3 to +22 ms. In the other three trials it followed by 0.39–0.96 s, because the background pane was throttled. **`visualFrameMs` and `avOffsetEstMs` from these runs are not valid measures of display timing or audiovisual synchrony.**

## C. Checks still pending (required before research use)

| # | Check | How | Pass criterion (to be set in the approved protocol) |
| --- | --- | --- | --- |
| C1 | Visual onset latency and jitter | Enable the photodiode patch; record a light sensor on the study display, foreground fullscreen, 30-trial session | Latency and SD within protocol limit; no dropped frames |
| C2 | Audio onset relative to visual onset | Loop the audio output (wired) and the photodiode into one recorder | Audiovisual asynchrony within protocol limit |
| C3 | Sound level | Sound level meter at the headphones with the 1 kHz tone at the session gain | Matches the approved level |
| C4 | Marker transfer to acquisition | Implement `window.AffectLabMarker` for the EEG–fNIRS system; record shared markers in both systems | Every code received; latency, jitter and drift reported |
| C5 | Full-length session | 30 trials at default timing on the study computer | 30/30 rows; durations match settings |
| C6 | Other browsers | Firefox and Safari (desktop) | Same functional results as section A |
| C7 | Research stimuli | Licensed audiovisual pairs imported; duration-mismatch warnings reviewed; RMS levels matched | No unmatched pairs; documented matching |
| C8 | Participant-facing usability | Pilot with test volunteers after ethics approval | Instructions and rating screens understood |

## D. Files in this package

`index.html` (the application), `README.md`, `VERIFICATION.md`, `LICENSE` (MIT), `.nojekyll`, and `sample_assets/`: six synthetic PNG images labelled "synthetic test asset", six matching 8 s mono 44.1 kHz WAV files at equal −26 dBFS RMS, and `sample_pairing_manifest.csv`. All sample assets were generated for this package; none is a copyrighted stimulus.


## v5.0.0 checks (14 Nov 2026, Chromium-based browser pane of the Claude desktop application, Windows 11, Demo Mode)

| Check | Result |
| --- | --- |
| Page loads, no console errors | yes |
| AO session: stimulus screen shows only the fixation cross; `stimulus_onset` code 21 and `audio_onset` code 31 logged; `imageShown=false`, `audioStatus=ok` | yes |
| Audio onset delay 0.5 s: `audioOnsetEstMs − imageOnsetMs` | 551 ms (0.5 s delay plus reported output latency; software estimate) |
| AV session: image shown, `audioStatus=ok`, `avOffsetEstMs` | −66 ms (software estimate, not a measured onset) |
| VO session: audio panel hidden, Start enabled without the tone check; ratings 4 / 2 recorded, `trialStatus=complete`, `ratingValid=true` | yes |
| Ending a session early marks unpresented trials `not_presented` | yes |
| Trial CSV column count | 64 |

Not tested: Firefox and Safari, real IADS-E or GAPED files, physical onset timing (photodiode, audio loopback).
