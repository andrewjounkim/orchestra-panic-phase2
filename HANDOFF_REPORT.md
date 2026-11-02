# Handoff Report: Orchestra Panic

To diegosecor: thanks for the foundation. Here's what it was like to pick it up.

## What I read first, and did it help?

I read SPEC.md first, then README.md, then the code. The spec helped the most: the "Intended Features" list and the numbered acceptance criteria made it clear what the finished game should be. The README helped with how to play, but the run steps were incomplete (see below). Reading the code against the criteria showed that almost everything was already there, and that the one clear gap was criterion 5. Notes slid in from the sides instead of falling from the top.

## Was the spec clear?

Yes, mostly. I could tell what you wanted: a conductor game where each hand follows its own falling targets, judged by height relative to the head. Most criteria were checkable. #6 (judge at arrival time, score once per target) and #11 (boo once per streak of 5, reset on a hit) were especially easy to verify against the code. A few were harder to check:
- "Clearly labeled", "readable" and "clear message" (#1, #5, #10) are judgment calls.
- The 0.3 s miss-mute rule and the minimum-version music are marked "still open". Since they were undecided, I couldn't tell whether to implement them.

## What made the code easy to work with

- **Function names say what they do.** `evaluateNotes()`, `recordHit()`, `recordMiss()`, `cameraToLevel()`, `createNotes()` and `finishConcert()` made it fast to find where each criterion lived.
- **Beat detection is separate.** It lives in `audio-chart.js`, away from the game loop, so chart timing was easy to check on its own.
- **Tunable values are grouped at the top of `game.js`.** Difficulty tolerances are in one `difficultySettings` object, and the layout constants (`LEFT_RAIL`, `RIGHT_RAIL`, `FLIGHT_TIME`, `CONTROL_TOP`/`CONTROL_BOTTOM`) are together, so it was easy to see how the zone size and note timing work.
- **`drawNotes()` is self-contained.** Making notes fall from the top took a 5-line change inside it, and nothing else had to change.
- **The tracking models ship in `assets/models/`**, and the camera errors (denied / not found / busy) each get their own message.

## What cost me time

- **The undocumented `npm install` step.** `node_modules/` is gitignored, but the tracking code imports MediaPipe from it. The README's run steps only say `node start-server.mjs`, so following them exactly gives "Motion tracking could not start."
- **Half-wired features.** `game.js` looks up `#stage-message`, `#preview-button`, `#back-button`, `#check-tracking`, `#mode-badge`, `#song-progress`, `#song-time` and `#song-detail`, but none of them exist in `index.html`. Several error messages tell the player to use "Watch preview", which has no button. Working out which features were real took a while.
- **One 1,059-line file with very few comments.** Magic numbers like `((headY - handY) + 0.34) / 0.78` in `cameraToLevel()` have no explanation of why those values were chosen.
- **The README overstated chart generation.** It says notes come from the MP3's audio energy, but the demo uses a fixed 124 BPM grid, and songs with weak detection fall back to a fixed 0.62 s grid.

## One thing I'd ask you to do differently next time

Before handing off, either finish or delete half-built features, and make the README's run steps work on a fresh clone. Most of my lost time came from code that pointed at things that weren't there.
