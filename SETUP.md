# Setting up Plumber

You are a coding agent, and someone asked you to set up Plumber in the repo you're working in. Follow these steps in order. [FORMATS.md](FORMATS.md) defines every file, so read it first.

The owner should only have to answer a few questions and try the app once. Do everything else yourself. Work on a branch named `plumber-setup`, and don't merge it until step 9 passes and the owner agrees.

**Fit into what exists.** Plumber is a set of conventions, not a framework. When the app already does part of this (a log, test cases, a notes file, a maintenance routine), keep it, record it in `PLUMBER.md`, and add only what's missing. Never rename fields or files other code reads.

## 1. Look before you ask

Read the repo's README, AGENTS.md or CLAUDE.md, and its entry points. Work out:

- what kind of app it is (Mac app, CLI, web app, local server with a web UI, bot) and where it runs: on each user's own machine, or on one server
- what "one thing the app did" means here: a dictation, a request, a command, a job
- how it's built, tested and shipped, and how the people using it get updates
- what already exists: a data folder, a log with one line per action, test cases and a way to replay them, a notes file of lessons learned
- **whether something already maintains it**: scheduled agents (`~/.claude/scheduled-tasks/`, other agents' schedulers), cron jobs, CI workflows, review branches, report folders or state files in the data folder. If you find one, `/maintain` must not run next to it as a second routine. Ask the owner (step 2) whether `/maintain` should replace it, or whether it should stay as is and only borrow what it lacks.

## 2. Ask a few questions

Ask them all in one message, each with the default you'd propose:

1. **What should the app do for someone, and what's the most annoying way it gets that wrong?** Ask for two or three real examples. The answer becomes "What counts as wrong" in `PLUMBER.md`, so push past "crashes" to what the output should have been.
2. **Who uses it besides you, and how do they get new versions?** This tells you whether to add "Send to <owner>" and what shipping means.
3. **Where should its data live?** Propose the existing data folder, or something like `~/.<app>/`. It has to be outside git.
4. Only if step 1 found an existing routine: **should `/maintain` replace it or leave it be?**

## 3. Write PLUMBER.md

Copy [templates/PLUMBER.md](templates/PLUMBER.md) to the repo root and fill it in. Keep it short, because every `/maintain` run reads it. Be exact about two things `/maintain` depends on:

- **Cases:** which file, and its format if it differs from FORMATS.md.
- **What replay can't cover:** UI, pasting, hotkeys, anything that needs a person. Also what replay costs to run and which keys it needs.

If there's no notes file of lessons learned, copy [templates/LEARNED.md](templates/LEARNED.md) to the repo root. If there is one (say `NOTES.md` with a "What we learned" section), point `PLUMBER.md` at it.

## 4. Record traces

Each thing the app does, failures included, should append one line to a traces log in the format in FORMATS.md.

- If the app already logs one line per action, keep that log and only **add** missing fields (`id`, `version`, `error`). Don't rename or remove existing fields, because other code may read them.
- Otherwise, write one small function next to the code that finishes each action.
- Stamp `version` with the build's git commit. Add it at build time if the app doesn't already know it.
- Cap new logs and new files (for example 30 days or 50 MB). Leave the retention of existing ones alone unless the owner asks.
- Never log keys or passwords. If the app runs on a server, traces go in a folder there that the owner can read.

## 5. Add "That was wrong"

Add one action, in the most natural place for this app, that appends a flag for the last trace to `flags.jsonl` and asks for an optional one-line note:

| App | Natural place |
|---|---|
| Mac menu bar app | A menu item "That was wrong…", with an optional hotkey |
| CLI | `<app> wrong "optional note"` |
| Web UI | A small "That was wrong" button next to each result |
| Chat bot | Replying "wrong" or "👎" to the bot's message |

**Owner or friend.** The first time someone flags something, ask once: "Is this your app, or are you using <owner>'s?" Remember the answer in the app's settings. The owner's flags stay in `flags.jsonl`. A friend's flag also offers **Send to <owner>**.

**Send to <owner>** builds the report zip in FORMATS.md. Before saving, it shows what's inside so the friend can really see it. Text is shown in full. Each file is its own checkbox with a real preview: play the audio, show the image. Screenshots and any file that could show other people's messages start unchecked. Say plainly that <owner> will replay it through the app, which may send it to the services the app uses. Then save it to Downloads and tell them to send it any way they like. Nothing is uploaded.

## 6. Make it replayable

`/maintain` has to run a saved input through the current build without the UI. This is the most important engineering step.

- If the app already has a headless mode or a test entry point that takes an input and prints the output, use it as it is. Don't refactor it during setup. If it's a separate copy of the app's logic, note that in `PLUMBER.md` as a risk.
- If it doesn't have one, add the smallest one: a flag such as `--replay <input>`, or a test helper, that calls the same code the real app uses.
- If the repo already has a replay script, keep it. Otherwise write `scripts/replay`, in whatever language the repo uses for scripts. It builds the app, runs every case, checks `expect` and `reject`, and reruns a failure once if the app calls a model. It prints ✔ or ✘ per case, with the output under each ✘. It reports a case it couldn't run (no network, no key) separately, and never as a failure. It exits non-zero only on real failures of cases that aren't `known_failing`.

## 7. Install /maintain

Copy [skills/maintain/SKILL.md](skills/maintain/SKILL.md) to `.claude/skills/maintain/SKILL.md` in the repo, so it's versioned with the app. Add this to the repo's AGENTS.md or CLAUDE.md, creating AGENTS.md if neither exists:

```markdown
## Plumber

This app is looked after by Plumber. Read PLUMBER.md before changing behaviour. To run a maintenance pass,
use /maintain (Claude Code), or follow .claude/skills/maintain/SKILL.md step by step (other agents).
```

If the owner chose in step 2 to replace an existing routine, change it so it runs `/maintain`, or retire it. Don't leave two running.

## 8. Seed the cases

Turn the examples from question 1 into the first cases, in the file and format `PLUMBER.md` names. If there are already traces, look through recent ones for failures that match "What counts as wrong" and suggest a few more. Add those only once the owner agrees.

## 9. Check it end to end

1. Build and run the app's tests.
2. Run replay. The seeded cases should run. Any that fail should be real, known problems: mark them `known_failing` with a reason.
3. Ask the owner to use the app once and press "That was wrong". Check that a trace and a flag appeared and point at each other. If "Send to <owner>" exists, check that the zip opens and holds only what was ticked. If trying it means installing over the owner's working copy of the app (a `build.sh` that replaces the installed app, a deploy), ask first.
4. Run `/maintain` once, attended, and check that it picks up the flag.

Then commit, merge only if the owner says so, and tell them in a few lines:

- how to flag something, and how friends send reports
- that `/maintain` is the only command to remember, with an offer to run it on a schedule (daily is a good start) unless something else already does
- where the data lives, and that it stays on their machine
