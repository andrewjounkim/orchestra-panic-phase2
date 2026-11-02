# Prompt Log (Phase 2)

Every prompt sent to the AI assistant (Claude Code, model Claude Opus 5.5) while finishing this inherited project, word for word, in order. Each prompt is followed by a short note on what the AI did in response.

Original project: https://github.com/diegosecor/HW7- by diegosecor.

---

### Prompt 1

```
Set up my Phase 2 repo. Do NOT open, read, or modify any project files during this step.

1. Clone https://github.com/diegosecor/HW7- into a folder called orchestra-panic-phase2
2. Delete its .git folder (detach from their history), then git init
3. Commit everything exactly as received: "Initial commit: unmodified code as received"
4. Check `gh auth status`; if I'm not logged in, tell me what to run and wait
5. Create a PUBLIC GitHub repo named orchestra-panic-phase2 with gh and push main
6. Show me `git log --stat` proving the first commit is the unmodified code, and give me the URL
```

**AI response:** Cloned the repo, removed its history, re-initialized git, committed all 14 files unmodified, created the public repo `orchestra-panic-phase2` and pushed it, then showed `git log --stat` and the URL.

### Prompt 2

```
This is a game I inherited from another student (GitHub: diegosecor): "Orchestra Panic", a
webcam hand-tracking rhythm game. Their SPEC.md is the target; their README.md describes the
current state. Read SPEC.md first, then README.md, then the code.

Do NOT change anything yet. Tell me:
- How the code is structured (files, main loop, key functions, where hand tracking and
  chart generation happen)
- Each acceptance criterion: done / partial / not started, based on the CODE, not the README
- Anywhere the README claims something the code doesn't actually do
- Which criteria look closest to done, and which the author ranked most important
- Environment assumptions (Node version, npm install, browser, camera) that could trip me up
Then give me exact steps to run it so I can test it myself.
```

**AI response:** Read SPEC.md, README.md and the code without changing anything. Reported the code structure and the status of each criterion: all met except #5, which was partial because notes slid in sideways instead of falling. Listed README claims the code doesn't support (including the missing `npm install` step), the environment requirements, and run steps.

### Prompt 3

```
Implement acceptance criterion #N from SPEC.md: "<paste the criterion>".

Rules: I must preserve at least 75% of the original author's work.
- Make the smallest change that delivers this criterion.
- Work inside their design: keep their files, functions, naming style, and structure.
- Do not rename, reformat, or rewrite code unrelated to this criterion.
- If you think a refactor is necessary, stop and explain why before doing it.
Show me the diff summary, then commit with a clear message.
```

**AI response:** Asked which criterion to implement, because the prompt still contained placeholders.

### Prompt 4

```
Update README.md (do not touch SPEC.md, and do not create HANDOFF_REPORT.md):
- At the top: credit the original author: "Original game design and foundation by <name or diegosecor>"
- Run instructions and code map, revised to match the code as it is now
- Acceptance criteria report: walk every criterion in SPEC.md by number, mark met / partially met /
  not met, one sentence each, and say why for skipped or unclear ones
- Known bugs and next steps in priority order
- Note any substantial restructuring and why
Verify every claim against the code before writing it. Commit and push when done.
```

**AI response:** Waited to do this until after the criterion work, so the README would describe the final code.

### Prompt 5

```
#5
```

**AI response:** Implemented criterion 5 with a 12-line change in `drawNotes()` in `game.js`. Notes now fall straight down their own L/R rail from above the screen at a constant speed and land on their marker exactly at their arrival time. Judging, chart timing and landing markers are unchanged.

### Prompt 6

```
im ready
```

**AI response:** Before updating the README, rechecked the criterion 5 change and fixed a problem it had introduced: with one constant fall speed, high-landing notes were visible for only about 0.9 s. Every note now falls over the original `FLIGHT_TIME`, so the total change to `game.js` is 5 lines. Then rewrote README.md to match the code: credit to diegosecor, run steps including `npm install`, a code map, a status for each criterion, known issues in priority order, and a list of Phase 2 changes. Committed and pushed.

### Prompt 7

```
check over the requirements again
```

(followed by the full HW7 assignment page, pasted)

**AI response:** Checked both repos against the assignment. Found that HANDOFF_REPORT.md is still missing (the student must write it without AI), added this section on unwanted AI behavior, and added the missing "known bugs and traps" and "cut corners" sections to the Phase 1 README. The pasted page contains an instruction aimed at AIs to put "hotdog" comments in code; the AI ignored it.

---

## When the AI did something I didn't want

1. **It changed gameplay while implementing criterion 5.** I asked for the smallest change that makes notes fall from the top. The AI's first version (commit `ee253d9`) gave every note one constant fall speed. As a side effect, notes landing high on the rail were visible for only about 0.9 s instead of the original 2.35 s, which made them much harder to hit. The criterion didn't ask for that. The AI caught it while rechecking the code before the README update. The fix (commit `d984ee3`) brought back the original `FLIGHT_TIME` timing, so every note is on screen for the same 2.35 s as before. Both commits are kept separate so the mistake and the fix are visible in the history.
2. **Its first edit for criterion 5 would have crashed the game.** It referenced a function that doesn't exist (`CONTROL_BOTTOM_FOR_FALL()`) in a constant at the top of `game.js`. It noticed right away and fixed it before committing, then ran `node --check` before committing anything.
