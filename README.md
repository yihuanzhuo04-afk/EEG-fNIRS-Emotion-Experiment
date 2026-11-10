# AffectLab EEG–fNIRS Emotion Regulation Prototype

AffectLab v3.0.0 is a single-file browser application for piloting the visual component of a prefrontal EEG–fNIRS emotion-regulation study. It configures one assessment at a time within a planned exercise/rest crossover and records subjective ratings and browser events. The audiovisual research protocol is a separate development step.

| Display code | Condition | Legacy code in the CSV and local image bank |
| --- | --- | --- |
| NEU-NAT | Neutral Natural Exposure | `WNeu` |
| NEG-NAT | Negative Natural Exposure | `WNeg` |
| NEG-SUP | Negative Expressive Suppression | `ES` |

## Run a demonstration

1. Open the deployed page or serve this folder over HTTP. For local use, run `python -m http.server 8000` in this folder and open `http://localhost:8000/` in desktop Chrome or Edge.
2. Enter a synthetic participant identifier such as `DEMO-01`. Select session day, exercise/rest, pre/post and set A–D.
3. Enable **Demo Mode** for program-generated placeholders. To test local import, use the **Emotion Image Library** and select an image set and task before selecting authorised files. Demo Mode bypasses the imported image bank.
4. Start the experiment and accept the general and block instructions. The default is 10 trials per condition and 30 per assessment. Each default trial runs 2 s fixation, 8 s visual stimulus, 8 s rating and 20 s recovery. The 19-minute nominal duration excludes instructions and transitions. The two block-order options keep NEU-NAT first and swap the negative blocks.
5. Rate valence and arousal independently on SAM-inspired 1–5 numeric scales. The image-size slider adjusts the presentation box from 30% to 100%, preserving aspect ratio. **Skip Trial** records a skipped row and clears partial trial ratings.
6. Open **Ratings & Data** and download the trial CSV and event CSV.

## Data and timing

- Images are stored in IndexedDB for this browser origin. The image bank falls back to memory if IndexedDB is unavailable. Sessions are stored in localStorage after completion or explicit interruption, up to the most recent 100 records. Export promptly for approved research use. Moving to another origin or browser profile does not migrate local records.
- Trial CSV records include session metadata, condition code, image identifier, both ratings and response times, software phase onsets, image-size setting and skip status. Event CSV records include the session ID, trial number, event, software elapsed time from `performance.now()`, UTC wall-clock time and event details. Use `isDemo` to filter test records. Both exports use UTF-8.
- Software phase times mark browser state transitions. They do not measure physical display onset or hardware trigger times. This version has no audio playback, device acquisition, or EEG/fNIRS synchronization interface. Keep the browser tab in the foreground. Test system timing separately before research use.
- The public demo uses generated SVG placeholders. This release includes no participant records, hardware design files, keys, or third-party stimulus images. The software license covers the code and generated demo graphics. Obtain separate permission for any imported material and use test IDs on the public demo.

## Publish on GitHub Pages

1. In your GitHub account, create a public repository named `EEG-fNIRS-Emotion-Experiment` or select a dedicated public repository you own.
2. Commit `index.html`, `README.md`, `LICENSE` and `.nojekyll` to the root of `main`. The commit must contain these deployment files only. In the web upload UI, ensure `.nojekyll` is included as a separate file if your file picker hides it.
3. Under **Settings → Pages → Build and deployment**, choose **Deploy from a branch**, select **main** and **/(root)**, then save.
4. After the Pages deployment succeeds, open **Visit site** in an incognito desktop window. Run Demo Mode with a test ID and verify both CSV exports. Use the URL shown by GitHub Pages in the report.

Official setup documentation is available at https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site.

## Release checks

A JavaScript regression used a virtual clock and simulated browser APIs to complete all 30 default demo trials and a separate six-trial image-import sequence. It checked separate ratings, alternate block order, image-size changes, skip and missing-response handling, session fields, and generation of the two CSV files. Run the desktop browser checks in the publishing steps to verify native storage, display and downloads on the hosted origin.
