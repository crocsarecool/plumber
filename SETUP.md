# Setting up Plumber

You are a coding agent, and someone asked you to set up Plumber in the repo you're working in. Follow these steps in order. [FORMATS.md](FORMATS.md) defines every file, so read it first.

The owner should only have to answer a few questions and try the app once. Do everything else yourself. Work on a branch named `plumber-setup`. It merges into main in step 9, once its checks pass and the owner agrees, because `/maintain` works from main.

**Fit into what exists.** Plumber is a set of conventions, not a framework. When the app already does part of this (a log, test cases, a notes file, a maintenance routine), keep it, record it in `PLUMBER.md`, and add only what's missing. Never rename fields or files other code reads.

## 1. Look before you ask

Read the repo's README, AGENTS.md or CLAUDE.md, and its entry points. Work out:

- what kind of app it is (Mac app, CLI, web app, local server with a web UI, bot) and where it runs: on each user's own machine, or on one server
- what "one thing the app did" means here: a dictation, a request, a command, a job
- how it's built, tested and shipped, and how the people using it get updates
- what already exists: a data folder, a log with one line per action, test cases and a way to replay them, a notes file of lessons learned
- **whether something already maintains it**: scheduled agents (`~/.claude/scheduled-tasks/`, other agents' schedulers), cron jobs (`crontab -l`), launchd jobs (`~/Library/LaunchAgents/`), CI workflows, review branches, and report folders or state files in any data folder the app already has. If you find one, `/maintain` must not run next to it as a second routine. Ask the owner in step 2 what to do about it.

Then, before asking anything, tell the owner in a few lines what the app already has and what you'll add. Everything happens on the `plumber-setup` branch, and nothing merges without their yes.

## 2. Ask a few questions

Ask them all in one message, each with the default you'd propose:

1. **What should the app do for someone, and what's the most annoying way it gets that wrong?** Ask for two or three real examples. The answer becomes "What counts as wrong" in `PLUMBER.md`, so push past "crashes" to what the output should have been.
2. **What name do your friends know you by, who uses it besides you, and how do they get new versions?** The name goes on "Send to <owner>" (default: the git user name, unless that's a tool's name such as "Cursor Agent"). The rest tells you whether friends need that button at all, and what shipping means.
3. **Where should its data live?** Propose the existing data folder, or something like `~/.<app>/`. It has to be outside git.
4. Only if step 1 found an existing routine, offer three choices:
   - **Keep it, and teach it to read flags and friends' reports.** Recommend this when the routine is already working well.
   - **Replace it with `/maintain`.**
   - **Leave it alone.** Warn that nothing will then read flags or reports.

Anything that lives outside the repo, such as a scheduled task in `~/.claude/scheduled-tasks/`, is off the branch. Describe the change you'd make to it, and make it only after the owner says yes.

## 3. Write PLUMBER.md

Copy [templates/PLUMBER.md](templates/PLUMBER.md) to the repo root and fill it in. Keep it short, because every `/maintain` run reads it. Be exact about the things `/maintain` depends on:

- **Plumber version:** the commit of this repo you're reading (`git rev-parse --short HEAD` in your clone, or `unknown` if you read it as web pages). Updates start from it.
- **Cases:** which file, and its format if it differs from FORMATS.md. If the app has no cases yet, put them in the data folder: they hold people's input, so they stay out of git. That means they aren't on any branch, and losing the data folder loses the gate. Tell the owner to back it up.
- **Ship:** how a fix reaches the owner and how it reaches friends. If friends only get it when the owner sends them a new copy, write exactly that. `/maintain` uses it to say what a friend is waiting on.
- **Trace fields:** if the app's log uses its own names, map them to FORMATS.md, for example `t=time, kind=type, input=raw, files=[recording]`. `/maintain` and friends' reports read traces through this map.
- **State and reports:** if an existing routine is kept, use its state file and report folder rather than adding Plumber's own.
- **What replay can't cover:** UI, pasting, hotkeys, anything that needs a person. Also what replay costs to run and which keys it needs.

If there's no notes file of lessons learned, copy [templates/LEARNED.md](templates/LEARNED.md) to the repo root. If there is one (say `NOTES.md` with a "What we learned" section), point `PLUMBER.md` at it.

## 4. Record traces

Each thing the app does, failures included, should append one line to a traces log in the format in FORMATS.md.

- If the app already logs one line per action, keep that log and only **add** missing fields (`id`, `version`, `error`). Don't rename or remove existing fields, because other code may read them.
- Otherwise, write one small function next to the code that finishes each action.
- Stamp `version` with the git commit the app was built from. Add it at build time if the app doesn't already know it. If there's no build step (a plain script), read it at run time when it runs from the repo, with `git describe --always --dirty`, so a run with uncommitted changes says so. If friends get a copy of the file, add the smallest command that makes that copy with the commit stamped in, and name it in Ship. Otherwise write `unknown`.
- Cap new logs and new files (for example 30 days or 50 MB). Leave the retention of existing ones alone unless the owner asks.
- Never log keys or passwords. If the app runs on a server, traces go in a folder there that the owner can read.

## 5. Add "That was wrong"

Add one action, in the most natural place for this app, that appends a flag to `flags.jsonl` and asks for an optional one-line note. It flags the result it sits next to, or the last trace when there's no such place (a menu item, a command):

| App | Natural place |
|---|---|
| Mac menu bar app | A menu item "That was wrong…", with an optional hotkey |
| CLI | `<app> wrong "optional note"` |
| Web UI | A small "That was wrong" button next to each result |
| Chat bot | Replying "wrong" or "👎" to the bot's message |

Friends may have only the app, not the repo, so make the action findable from the app itself: its menu, its help text, `<app> --help`.

**Owner or friend.** The first time someone flags something, ask once: "Is this your app, or are you using <owner>'s?" A friend is also asked their name, which goes in the flag's `by`. Remember the answers in the app's settings. The owner's flags stay in `flags.jsonl`. A friend's flag also offers **Send to <owner>**.

**Send to <owner>** builds the report zip in FORMATS.md. Before saving, it shows what's inside so the friend can really see it. Text is shown in full. Each file is its own checkbox with a real preview: play the audio, show the image. Screenshots and any file that could show other people's messages start unchecked. If the trace has no files, show the text and skip the checkboxes. Say plainly that <owner> will replay it through the app, which may send it to the services the app uses. Then save it to Downloads and tell them to send it any way they like. Nothing is uploaded.

## 6. Make it replayable

`/maintain` has to run a saved input through the current build without the UI. This is the most important engineering step.

- If the app already has a headless mode or a test entry point that takes an input and prints the output, use it as it is. Don't refactor it during setup. If it's a separate copy of the app's logic, note that in `PLUMBER.md` as a risk.
- If it doesn't have one, add the smallest one: a flag such as `--replay <input>`, or a test helper, that calls the same code the real app uses.
- If the repo already has a replay script, keep it, and add only what it lacks from the list below: `known_failing`, and reporting "couldn't run" separately from a failure. Otherwise write `scripts/replay`, in whatever language the repo uses for scripts. It builds the app, runs every case in the file `PLUMBER.md` names, checks `expect` and `reject`, and reruns a failure once if the app calls a model. It prints one line per case: `✔ <id>`, `✘ <id>`, or `⚠ <id> couldn't run: <reason>` (no network, no key, a missing file). It prints the output under each ✘, and under every case with a `check` rule, because `/maintain` has to read it. It runs the app with its data folder pointed at a throwaway folder, so replay never adds traces or flags to the real one. A `known_failing` case that fails still prints ✘, with its reason. It exits non-zero only on real failures of cases that aren't `known_failing`. A case's `input` is always text. A file input is named in `file` (FORMATS.md), so a one-word input is never mistaken for a missing file.

## 7. Install /maintain

**If the owner kept an existing routine** (step 2), don't install `/maintain`. With their yes, add a section to that routine that does steps 3 and 4 of [skills/maintain/SKILL.md](skills/maintain/SKILL.md): collect flags and friends' reports, and treat each one as worth acting on. In the snippet below, name that routine instead of `/maintain`.

Otherwise, copy [skills/maintain/SKILL.md](skills/maintain/SKILL.md) to `.claude/skills/maintain/SKILL.md` in the repo, so it's versioned with the app. Add this to the repo's AGENTS.md or CLAUDE.md, creating AGENTS.md if neither exists:

```markdown
## Plumber

This app is looked after by Plumber (github.com/crocsarecool/plumber). Read PLUMBER.md before changing behaviour.

- Owner: to run a maintenance pass, use /maintain (Claude Code), or follow .claude/skills/maintain/SKILL.md
  step by step (other agents).
- Anyone else: if the app got something wrong, use "That was wrong" and then "Send to <owner>" (PLUMBER.md → People
  says where it is). Never upload or send a report for them. They send it themselves.
```

If the owner chose in step 2 to replace an existing routine, change it so it runs `/maintain`, or retire it. Don't leave two running.

## 8. Seed the cases

Turn the examples from question 1 into the first cases, in the file and format `PLUMBER.md` names. An example with no trace gets a short readable id, such as `setup-dinner-uneven`. If there are already traces, look through recent ones for failures that match "What counts as wrong" and suggest a few more. Add those only once the owner agrees.

## 9. Check it end to end

1. Build and run the app's tests.
2. Run replay. The seeded cases should run. Any that fail should be real, known problems: mark them `known_failing: "not tried yet: <what's wrong>"`, so `/maintain` works on them in its first runs.
3. Ask the owner to use the app once, press "That was wrong", and type `test` as the note, so `/maintain` doesn't try to fix it. Check that a trace and a flag appeared and point at each other. If "Send to <owner>" exists, try it yourself as a friend: point the app's data folder at a throwaway folder, flag once, send, and check that the zip opens and holds only what was ticked. Then delete the zip and the throwaway folder, so `/maintain` never mistakes it for a real report. If trying it means installing over the owner's working copy of the app (a `build.sh` that replaces the installed app, a deploy), ask first.
4. Commit on `plumber-setup`, show the owner what changed in a few lines, and ask to merge it into main. Merge only on a yes. `/maintain` starts from main and never commits to it, so it can't run on the setup branch.
5. On main, run `/maintain` once, attended, as a check only: its steps 1–3, then step 8's state update and report. Don't fix anything. The "not tried yet" cases wait for the first real run. Check that it picks up the test flag and leaves it alone. If the owner said no to merging, skip this and tell them the first `/maintain` after they merge is the check.

Then tell them in a few lines:

- how to flag something, and how friends send reports
- that `/maintain` is the only command to remember, with an offer to run it on a schedule (daily is a good start) unless something else already does. The scheduled prompt must say nobody is watching, for example "Run /maintain. This is a scheduled run: don't ask, don't merge."
- where the data lives, and that it stays on their machine
