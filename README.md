# Orchestra Panic

**Original game design and foundation by diegosecor** ([diegosecor/HW7-](https://github.com/diegosecor/HW7-)). Phase 2 changes are listed below and in [prompt_log.md](prompt_log.md).

`Orchestra Panic` is a local browser rhythm game. You conduct an orchestra by raising and lowering both hands in front of your webcam. Notes for your left and right hands fall toward landing markers, and you score when the matching hand is at the right height as each note arrives. [SPEC.md](SPEC.md) is the target design.

## Run locally

Requirements: Node.js (any recent version; tested with v26), a webcam, and Chrome or Edge.

```bash
npm install    # required: installs @mediapipe/tasks-vision 0.10.21 into node_modules/
npm start      # same as: node start-server.mjs — keep this terminal open
```

Open **http://127.0.0.1:4173** and allow camera access.

- **Don't skip `npm install`.** `node_modules/` is not in the repo, and the hand/face tracking runtime is loaded from it. Without it, starting a concert fails with "Motion tracking could not start."
- **Use the server.** Opening `index.html` directly won't work, because the game uses ES modules and camera access, which need `http://127.0.0.1`.
- **macOS:** your browser needs camera permission in System Settings → Privacy & Security → Camera.
- **Stay on the tab.** Switching tabs during a concert stops it and turns the camera off.
- The tracking models live in `assets/models/`, so nothing is downloaded at runtime once `npm install` has been run.

## How to play

1. The included demo song (**Panic Parade**, 124 BPM, about 32 s) is selected by default. You can also choose your own MP3 or WAV (3 s to 20 min, under 100 MB).
2. Pick a difficulty and click **START THE CONCERT**.
3. Allow the camera, then step back so your **face and both hands** are visible. The music starts after 3 seconds of steady tracking.
4. Blue **L** notes fall down the left rail and coral **R** notes down the right rail. Each note lands on a box showing the height you need. Move the matching hand so the conductor's hand is inside the box when the note lands.
5. When the song ends, you see your score, hits, misses, accuracy and best combo. **PLAY AGAIN** returns to the setup screen with your song still selected.

**Scoring:**
- A hit scores PERFECT, GREAT or GOOD (100, 75 or 50 points) depending on how close your hand is to the note's height.
- A combo multiplies your points: ×2 at 10 hits in a row, ×3 at 20, ×4 at 30.
- A miss briefly muffles the music.
- Five misses in a row makes the audience boo, once per streak.
- When misses outnumber hits, tomatoes start flying.

**Difficulty:**
- **Easy:** a wide hit zone, notes on every other beat, and landing heights kept to the middle of the rail.
- **Normal:** a medium hit zone, notes on every beat alternating hands, and occasional notes for both hands at once.
- **Hard:** a narrow hit zone, extra notes between beats, and large height jumps.

## Code map

| File | What it does |
|---|---|
| `index.html` | Three screens as DOM sections: setup (song picker, difficulty, START), game (canvas + HUD), results. Loads `game.js` as a module. |
| `game.js` | Almost all game logic (details below). |
| `audio-chart.js` | Beat timing: `findBeats()` detects energy peaks in an uploaded song, `createDemoBeats()` returns the demo's known 124 BPM grid, and `validateDuration()` enforces 3 s–20 min. |
| `start-server.mjs` | Small static file server on `127.0.0.1:4173`. |
| `styles.css` | Setup, HUD and results styling. |
| `generate-demo-track.mjs` | Dev-only script that synthesized `demo-track.wav` (`npm run generate-demo`). Not needed to play. |
| `assets/models/` | MediaPipe hand and face landmark models. |

Key functions in `game.js`:
- **Starting a round:** `startConcert()` decodes the audio, builds the note chart, loads tracking (`ensureVisionModels()`) and opens the camera (`ensureCamera()`).
- **Main loop:** `gameLoop()` runs every frame:
  - `updateTracking()`
  - calibration: 3 s of valid tracking, then `beginAudio()`
  - `evaluateNotes()`, then `render()`
- **Hand tracking:** `updateTracking()` runs MediaPipe `HandLandmarker` and `FaceLandmarker`. Hand positions come from palm landmark 9 and the head position from face landmark 1. Left/right labels are swapped to match the mirrored camera. `cameraToLevel()` turns hand height relative to the head into a 0–1 level.
- **Chart generation:** `createNotes()` assigns beats to hands based on difficulty, and `nextLevel()` picks each note's landing height from a seeded random sequence.
- **Judging:** `evaluateNotes()` judges each note once, at its arrival time, then calls `recordHit()` or `recordMiss()`. A note only counts as a hit if tracking is fresh (updated within the last 240 ms).
- **Drawing:** `render()` calls `drawTheatre()`, `drawControlRails()`, `drawNotes()`, `drawConductor()` and `drawEffects()`.
- **Audio feedback:** `triggerDissonance()` (on a miss), `playBoo()`, `playApplause()`, all synthesized with Web Audio.
- **End of song:** `finishConcert()` fills in the results screen.

## Acceptance criteria report

Each status below was checked against the code.

| # | Status | Notes |
|---|---|---|
| 1 | **Met** | START requests the camera, and a denied, missing or busy camera shows a specific message on the setup screen (but see known issue 2). |
| 2 | **Met** | Left and right hands are tracked separately and drawn as distinct blue **L** and coral **R** conductor hands, with a HUD status for "head + both hands detected." |
| 3 | **Met** | Each hand's level is computed from its height relative to the head and drives that conductor hand independently. |
| 4 | **Met** | A local MP3 (or WAV) can be chosen, and the round ends when the song finishes playing. |
| 5 | **Met** (one caveat) | **Changed in Phase 2:** notes now fall from the top down their own labeled rail to a landing marker at varying heights. Before, they slid in sideways at their final height. Uploaded songs get their timing from detected energy peaks. The caveat: songs where too few peaks are detected fall back to a fixed 0.62 s grid (known issue 3). |
| 6 | **Met** | At each note's arrival time, the matching hand's level is compared with the note's height within the visible zone. Each note is judged exactly once, and hits add to the score. |
| 7 | **Met** | The results screen shows the score, hits and misses (plus accuracy and best combo). PLAY AGAIN returns to setup, where you can replay or choose another song. |
| 8 | **Met** | When the head or either hand is lost, the HUD shows a warning, the conductor fades, and notes arriving during the loss count as "NO TRACK!" misses. |
| 9 | **Met** | The hit tolerance is 0.22 / 0.14 / 0.085 of the rail height for Easy / Normal / Hard. The drawn zone box uses the same value as the judging check. |
| 10 | **Met** | The canvas shows curtains, spotlights, a 10-musician orchestra on stage, and an audience row, with the rails, notes and hands drawn on top. |
| 11 | **Met** | Five consecutive misses play a synthesized boo once per streak. A hit resets the streak, and the boo bypasses the music's volume. |
| Optional mute rule | **Not met** | SPEC leaves this rule undecided. The code instead muffles the music to 28% volume with a low-pass filter for about 0.4 s on a miss, rather than fully muting for 0.3 s. |

## Known issues and next steps (in priority order)

1. **No calibration countdown on screen.** `gameLoop()` writes "Ready in 3…" to a `#stage-message` element that doesn't exist in `index.html`. During the 3-second wait, players only see the small HUD tracking status. *Next step:* add the element to the game screen.
2. **Error messages mention a "Watch preview" button that doesn't exist.** `game.js` has a working camera-free autoplay preview (`startConcert(true)`), but no `#preview-button` in the HTML. *Next step:* add the button or reword the messages.
3. **Beat-detection fallback isn't based on rhythm.** If `findBeats()` finds too few peaks, it returns a fixed 0.62 s grid. The demo also uses a hard-coded 124 BPM grid, which does match how that track was generated. *Next step:* a tempo estimate (e.g. autocorrelation of the energy envelope) for the fallback.
4. **Other dead element references.** `#stage-preview`, `#back-button`, `#check-tracking`, `#mode-badge`, `#song-progress`, `#song-time` and `#song-detail` are looked up but missing from `index.html`, so their features (progress bar, mode badge, etc.) do nothing. They're guarded with `?.`, so nothing crashes.
5. **The START button changes its label.** `setBusy()` renames it to "Start with camera" after the first start attempt.
6. **A faint leftover rectangle** is drawn at the top of each rail in `drawControlRails()`, sized to the tolerance. It isn't a landing marker and may confuse players.
7. **Miss feedback vs. SPEC.** If the proposed 0.3 s mute is adopted, change `triggerDissonance()` to set the gain to 0 for 0.3 s.

## Phase 2 changes

- **No substantial restructuring.** All files, functions and names are the original author's.
- **Criterion 5:** in `drawNotes()` (`game.js`), notes now fall vertically from just above the screen to their landing marker over the existing `FLIGHT_TIME`, instead of sliding in from the sides. This changed 5 lines. Judging, chart generation and timing are untouched.
- **This README:** rewritten to match the code. It adds the missing `npm install` step and corrects the server address, the chart-generation claim, and the camera-video note. The video is shown faintly behind the stage, not only used for tracking.

## Privacy notes

- Chosen songs are played from a local object URL and are never uploaded or stored.
- The camera feed stays on your device. It drives hand tracking and is also drawn faintly (10% opacity) behind the stage.
- All sound effects (boo, applause, miss) are synthesized in the browser. `demo-track.wav` is an original track generated by `generate-demo-track.mjs`.
