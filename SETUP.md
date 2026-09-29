# Setting up Plumber

You are a coding agent, and someone asked you to set up Plumber in the repo you're working in. Follow these steps in order. [FORMATS.md](FORMATS.md) defines every file. Read it first.

The person should only have to answer three questions and try the app once. Do everything else yourself. Put the work on a branch named `plumber-setup` and don't merge it until step 9 passes.

## 1. Look before you ask

Read the repo's README, AGENTS.md or CLAUDE.md, and its entry points. Work out:

- what kind of app it is (Mac app, CLI, web app, local server with a web UI, bot) and where it runs: on each user's own machine, or on one server
- what "one thing the app did" means here: a dictation, a request, a command, a job
- how it's built, tested and shipped, and how the people using it get updates
- whether it already has a data folder, a log with one line per action, test cases, or a notes file of lessons learned. **Reuse whatever already exists.** Record where it lives. Don't build a second copy.

## 2. Ask three questions

Ask all three in one message, each with the default you'd propose:

1. **What should the app do for someone, and what's the most annoying way it gets that wrong?** Ask for two or three real examples. The answer becomes "What counts as wrong" in `PLUMBER.md`, so push past "crashes" to what the output should have been.
2. **Who uses it besides you, and how do they get new versions?** This tells you whether to add "Send to <owner>" and what shipping means.
3. **Where should its data live?** Propose the existing data folder or a default such as `~/.<app>/`. It has to be outside git.

## 3. Write PLUMBER.md

Copy [templates/PLUMBER.md](templates/PLUMBER.md) to the repo root and fill it in from steps 1 and 2. Keep it short, because every `/maintain` run reads it.

If there's no existing notes file of lessons learned, copy [templates/LEARNED.md](templates/LEARNED.md) to the repo root. If there is one (say `NOTES.md` with a "What we learned" section), point `PLUMBER.md` at it instead.

## 4. Record traces

Make the app append one line to `traces.jsonl` for each thing it does, including failures, in the format in FORMATS.md. Write it in one small function next to the code that finishes each action. Stamp `version` with the build's git commit: add it at build time if the app doesn't already know it. Cap the log and any files it keeps. Never log keys or passwords.

If the app runs on a server, traces go in a folder on that server that the owner can read.

## 5. Add "That was wrong"

Add one action, in the most natural place for this app, that appends a flag for the last trace to `flags.jsonl` and asks for an optional one-line note:

| App | Natural place |
|---|---|
| Mac menu bar app | A menu item "That was wrong…" and an optional hotkey |
| CLI | `<app> wrong "optional note"` |
| Web UI | A small "That was wrong" button next to each result |
| Chat bot | Replying "wrong" or "👎" to the bot's message |

If the app has other users (question 2), put **Send to <owner>** in the same place. It builds the report zip described in FORMATS.md, first shows the person exactly what's inside (the text, plus each file and its size), then saves it to their Downloads folder and tells them to send it to the owner any way they like. Ask for their name the first time, then remember it. Nothing is ever uploaded.

## 6. Make it replayable

`/maintain` has to run a saved input through the current build without the UI. This is the most important engineering step.

- If the app already has a headless mode or a test entry point that takes an input and prints the output, use it.
- If it doesn't, add the smallest one: a CLI flag such as `--replay <input>`, or a test helper. It should go through the same code the real app uses, not a copy.
- Write a `replay` script (in `scripts/`, in whatever language the repo already uses for scripts) that builds the app, runs every case in `cases/cases.json`, checks `expect` and `reject`, reruns a failure once if the app calls a model, and prints ✔ or ✘ per case with the output under each ✘. It exits non-zero if any case that isn't `known_failing` fails. It skips `check` rules; `/maintain` judges those.

If the repo already has a replay (Murmur's `scripts/regress.py`, say), keep it. If its case format differs, record the difference in `PLUMBER.md` rather than rewriting it.

## 7. Install /maintain

Copy [skills/maintain/SKILL.md](skills/maintain/SKILL.md) to `.claude/skills/maintain/SKILL.md` in the repo, so it's versioned with the app. Then add this to the repo's AGENTS.md or CLAUDE.md (create AGENTS.md if neither exists):

```markdown
## Plumber

This app is looked after by Plumber. Read PLUMBER.md before changing behaviour. To run a maintenance pass,
use /maintain (Claude Code), or follow .claude/skills/maintain/SKILL.md step by step (other agents).
```

## 8. Seed the cases

Turn the examples from question 1 into the first cases, in the format in FORMATS.md. If the data folder already has traces, look through recent ones for failures that match "What counts as wrong" and suggest a few more as cases. Only add those once the owner agrees.

## 9. Check it end to end

1. Build and run the app's tests.
2. Run `replay`. The seeded cases should run, and any that fail should be real, known problems. Mark those `known_failing` with a reason.
3. Ask the owner to use the app once and press "That was wrong". Check that a trace and a flag appeared and point at each other. If "Send to <owner>" exists, check that the zip opens and holds only that trace's files.
4. Run `/maintain` once, and check that it picks up the flag.

Then commit, merge to the main branch only if the owner says so, and tell them in a few lines:

- how to flag something, and how friends send reports
- that `/maintain` is the only command to remember, plus an offer to run it on a schedule (daily is a good start)
- where the data lives, and that it stays on their machine
