# Plumber: notes

Why Plumber is shaped the way it is. Read this before changing SETUP.md or the maintain skill.

The product thinking (the idea, why friends are the wedge, risks, open questions) is in [PRODUCT.md](PRODUCT.md).

## Where it came from (29 Sep 2026)

Murmur, a dictation app Raunaq vibecoded, got about 60 commits in its first 4 days, and most came from using it. By then it already had every part of a maintainer, built by hand: `traces.jsonl`, `regress.py` replaying real recordings, the learner catching fixes, a daily review routine, and NOTES.md "What we learned". Plumber is that loop, taken out of Murmur so any vibecoded app can have it.

First users: Murmur (Raunaq maintains it, and Anil uses it and sends reports) and Jarvis (Anil's own app, `jarvis-hq`, a CLI with a web UI).

## Test run of SETUP.md on Murmur (29 Sep 2026)

A fresh agent read SETUP.md as if installing Plumber into Murmur, without changing anything. It found 12 problems. All are fixed in commit 40f7f4e. Most serious first:

1. **A second maintenance routine.** SETUP only read the repo, so it missed Murmur's daily review (the scheduled task `cleaning-up-murmur`, with its state in `data/reviews/last-covered`). It would have added `/maintain` next to it, with two state files and two branch schemes. *Now:* step 1 looks for scheduled tasks, cron, CI and state files, and asks whether to replace them or leave them.
2. **A hardcoded case format.** The skill always wrote `cases/cases.json`. Murmur's cases are in `regressions/cases.json`, with audio by id and different fields. *Now:* PLUMBER.md names the case file and format, and everything uses that.
3. **A wrong error test.** Murmur writes `"error": ""` on success, and some lines (`learned`) have no id. Telling the agent to follow the format could also have made it rename fields that `Suggester.swift`, `cost.py` and `scorecard.py` read. *Now:* null, empty and missing all mean no error, lines with no id are ignored, and setup adds fields but never renames them.
4. **Leaks in reports.** A flagged command in Murmur would have put a screenshot of the chat on screen into the report, and "list the files and their sizes" doesn't show someone what's in a WAV or JPG. *Now:* each file is a checkbox with a real preview, screenshots start unticked, and the friend is told the owner will replay it through the app's services.
5. **Unattended runs.** On a schedule, "ask the owner" stalls the run, and `Merge: auto` plus "ship" would have run `./build.sh`, which reinstalls the app with nobody checking. *Now:* unattended runs never ask, merge, ship or install. Questions go under "Decisions for you".
6. **No baseline.** "Must still pass" can't be judged with model cases that give different results each run. *Now:* replay runs on main first, and that result is the baseline.
7. **What replay can't do.** Murmur's `--test` is a copy of the pipeline, and "not a copy" invited a refactor during setup. Pasting, the learner and hotkeys can't be replayed. A network failure looked like a regression. *Now:* setup doesn't refactor, PLUMBER.md lists what replay can't cover, and "couldn't run" is reported separately.
8. **`last_run` could skip flags** that arrived during a run. *Now:* it's set to the newest item collected.
9. **Unzipping untrusted reports.** The id in a friend's filename became a path. *Now:* ids are checked, and `..`, absolute paths, symlinks and anything over 50 MB are refused.
10. **Capping logs.** "Cap the log" would have cut `traces.jsonl`, which Murmur keeps whole on purpose (suggestions, cost, 7-day baselines). *Now:* only new logs are capped.
11. **Owner or friend.** Nothing said how the app knows whose install it is, when everyone builds from the same repo. *Now:* it asks once, the first time someone flags something.
12. **Recordings rotate away.** Murmur keeps the last 500, so a flagged recording could be gone before it's copied. *Now:* input files are copied first, with a text-only case if one is already gone.

## Agent entry point, and a second test run (29 Sep 2026)

AGENTS.md is now the front door, so someone can point any agent at the repo and it knows what to do: set up, report as a friend, update, or work on Plumber. CLAUDE.md only imports it.

A fresh agent tried two opening messages on Murmur without changing anything. The first was an owner saying "set up Plumber". The second was a non-coding friend saying "it got something wrong, help me report it". Both reached the right files. What it found, now fixed:

1. **Keeping an existing routine had no path.** Choosing "leave it" still installed `/maintain`, which meant two routines, and nothing read flags. *Now:* step 2 offers keep-and-extend (recommended), replace, or leave alone. Keeping it means the routine gains SKILL steps 3–4 and `/maintain` isn't installed.
2. **Trace field names.** Murmur's log uses `time`, `type`, `raw` and `recording`. *Now:* PLUMBER.md has a "Trace fields" map, along with rows for the state file and run reports.
3. **The friend path never asked what was wrong.** *Now:* it asks what was wrong, what it should have been, and their name. It finds the data folder itself, reads only the last few traces, skips lines with no id, opens each file for them, and warns that replay goes through the app's services.
4. Smaller: the owner's name wasn't asked, an existing replay script lacked `known_failing` and "couldn't run", "10 minutes" oversold it, and the two reading orders disagreed.

## Third test run: iou, a new app (29 Sep 2026)

Cursor built `iou`, a small CLI that splits a shared bill, then set up Plumber from SETUP.md and ran a full cycle. The owner flagged a dinner where tax and tip were split in half. A friend's report said "3 teas 4 each" came out as $3. `/maintain` fixed both on a branch, and replay went from failing to five passing cases. The owner loop worked. What it found, now fixed:

1. **Setup deadlocked.** SETUP said not to merge `plumber-setup` until step 9, and step 9 ran `/maintain`, which starts from main and never commits to it. The agent had to fast-forward main to get through. *Now:* step 9 asks the owner to merge before running `/maintain`, and the skill stops if main has no `PLUMBER.md`. `/maintain` still never merges itself.
2. **"Fixed in the next update" wasn't true.** Ship was "No command", so the friend kept the broken script. *Now:* setup records how fixes reach friends, and the report's reply to friends is worded from that. If the owner has to send a new copy, it says so.
3. **"Regression" was wider than the report.** Seeded cases already marked `known_failing` failed on the baseline. *Now:* only a failing case that isn't `known_failing` is a regression.
4. **`known_failing` was never cleared,** so "Still broken" would have listed fixed cases. *Now:* the gate clears it when the case passes.
5. **A one-word input looked like a missing file,** because `input` could be text or a file name. *Now:* `input` is always text, and a file goes in `file`. Replay's per-case output, including "couldn't run", is spelled out.
6. **Handled reports stayed in `inbox/` and Downloads.** *Now:* Downloads zips are moved, not copied, and a handled zip goes into `inbox/<id>/`.
7. Smaller: Send to <owner> assumed every trace had files, and the default owner name was the git user, which was "Cursor Agent".

**Kept on purpose: cases stay out of git.** The run pointed out that cases in `~/.iou/cases/` aren't on the fix branch, so losing the data folder loses the gate. Murmur keeps its cases out of git because they hold dictation text and the repo is public, and friends' inputs are the same. So setup now tells the owner to back up the data folder, and the report names the cases file the fix passed.

## Read-through after the iou run (29 Sep 2026)

A read of every file, looking for what a real run over several days would hit. What it found, now fixed:

1. **An unmerged fix looked like a regression.** A case fixed on a `maintain-` branch still fails on main, so the next run's baseline called it broken. The iou fixes made it worse by clearing `known_failing` as soon as the branch passed. Cursor's logs show it: after the run, all five cases had `known_failing: null` and the branch was left unmerged, so the next run on main would have reported four regressions. *Now:* it reads "fixed on <branch>, not merged yet" until the branch is in main, and each run starts by catching up on those branches.
2. **Work was lost for good.** `last_run` moves past everything collected, but a run fixes only two things, so a third flag was never seen again. Setup's own examples, the owner's worst complaints, were never worked on either. *Now:* both are cases marked "not tried yet", and `/maintain` works through them after flags and errors.
3. **Scheduled runs fought the owner.** A run stopped whenever the owner had uncommitted work, and when it did run, it left their checkout on the fix branch. Cursor's run ended that way, on `maintain-2026-09-29`. *Now:* `/maintain` works in its own worktree of main and never switches the owner's checkout.
4. **Update could add a second routine.** For an app that kept its own routine (the recommended choice for Murmur), "replace SKILL.md" would install `/maintain` next to it. It also couldn't tell what changed since setup. *Now:* `PLUMBER.md` records the Plumber version, update reads the changes since then, and it updates a kept routine instead of installing `/maintain`. Changes to cases or `PLUMBER.md` need the owner's yes.
5. **Old reports were skipped.** A report about something from two weeks ago was older than `last_run`. *Now:* every unread zip in `inbox/` counts, whatever its date.
6. **Finding the breaking commit** used `git log --merges`, which misses fast-forward merges, and a date misses branches merged late. *Now:* the state file records the main commit each run started from.
7. **The gate couldn't compare `check` rules,** because the baseline didn't judge them. It also didn't say what to do when a case couldn't run on the branch. *Now:* both runs judge them, and a fix that couldn't be fully checked says so.
8. **Setup's test flag got "fixed".** *Now:* the owner types `test` as the note, and `/maintain` leaves it alone.
9. Smaller: the README said "about 10 minutes" again, and claimed the report never has anyone's words while its example quoted them. "That was wrong" said "the last trace" even for a button next to a result. Setup told apps that already keep cases in git to move them.

## Borrowed from Murmur's daily review

`cleaning-up-murmur` (in `~/.claude/scheduled-tasks/`) was more mature than the first `/maintain`. These rules came from it:

- "Nothing worth changing today" is a good result. The goal is not to find something to change.
- Act only on a regression, something a person flagged, or a problem seen at least twice. Skip one-offs and speculative improvements.
- Make at most two fixes per run.
- Leave judgment calls (speed against accuracy, higher cost) to the owner, with a recommendation.
- Show the case failing before the fix and passing after it.
- Name the commit that caused a regression.
- Write the report to a file, and notify the owner only when something is waiting on them, never with people's input text.
- Never install.

Not borrowed yet: the scorecard (latency, failure rates and costs compared with the week before). It's worth adding once a second app shows which parts are generic.

## Open

- **Murmur:** keep `cleaning-up-murmur` and have it read flags and friends' reports too (recommended), or replace it with `/maintain`.
- **GitHub:** Anil's agent can't fetch Plumber until it's there, as `crocsarecool/plumber`.
- **Jarvis:** the real test, because it isn't Murmur.
- **Apps on a server.** SETUP says traces go in a folder on the server, but `/maintain` runs on the owner's machine and nothing says how it reaches that folder. Friends of a web app also share the owner's server, so their flags may land there directly and "Send to <owner>" may not be needed. Needs a decision before Jarvis or any web app.
- **Friends' updates.** The friend loop ends at the report. When friends run a copy the owner sent them, a fix reaches them only when the owner sends another one. The report now says so, but closing the loop needs an update channel, which PRODUCT.md leaves out of v0.
