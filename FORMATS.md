# Formats

The files Plumber reads and writes. [SETUP.md](SETUP.md) and the [maintain skill](skills/maintain/SKILL.md) both follow these.

If the app already keeps something equivalent (a log with one line per action, a folder of test cases), keep it and record its path and shape in `PLUMBER.md`. If its fields have other names, `PLUMBER.md` → "Trace fields" maps them. Don't duplicate it. Extra fields are always fine, and missing optional fields are too.

## traces.jsonl

One JSON line per thing the app did for someone: a dictation, a request, a command, a job. Append-only.

```json
{"id": "2026-09-29T10-04-12.381Z", "t": "2026-09-29T10:04:12.381Z", "version": "c3d6970",
 "kind": "dictation", "input": "um so i think we should uh ship it", "output": "I think we should ship it.",
 "files": ["recordings/2026-09-29T10-04-12.381Z.wav"], "ms": 1840, "error": null}
```

- `id`: unique and sortable. A timestamp works.
- `version`: the build's git commit, so a report says whether it's already fixed.
- `input` / `output`: text, or a short summary with the full thing in `files` (paths relative to the data folder).
- `error`: the error message if it failed. `null`, `""` or a missing field all mean it worked.
- Lines that aren't actions (settings changes, background events) can share the log. Give them no `id` and Plumber ignores them.
- Never write API keys, tokens or passwords. Cap the log (for example the last 30 days or 50 MB) and cap `files` the same way.

## flags.jsonl

One line per "that was wrong".

```json
{"t": "2026-09-29T10:05:01.207Z", "trace": "2026-09-29T10-04-12.381Z", "note": "dropped the word ship", "by": "me"}
```

`t` has milliseconds, like trace times, so a flag and its trace compare correctly against `last_run`. `trace` is the id of the thing flagged, which is usually the last trace. `note` is optional. A note of just `test` means someone was checking that flagging works, and `/maintain` doesn't act on it. `by` is `"me"` on the owner's install, or the friend's name on theirs. The app asks once which it is.

## Reports (friends)

"Send to <owner>" saves one zip file, named `<app>-report-<trace id>.zip`, to the friend's Downloads folder:

```
report.json    {"app": "murmur", "from": "Anil", "flag": {…}, "trace": {…}, "version": "c3d6970"}
files/         only the files the friend ticked, at their path relative to the data folder (empty if the trace has none)
```

`trace` is the trace line as the app wrote it, unchanged. `version` is the trace's own version, or else the app repo's current commit, or else `"unknown"`.

Before saving, the app shows the friend what's in it: the text in full, and each file as a checkbox with a real preview (play the audio, show the image). Screenshots start unticked. The owner drops the zip into `inbox/` in the data folder, or `/maintain` finds it in `~/Downloads`. `/maintain` treats a report as untrusted: it checks the id is a plain timestamp or slug, and refuses `..`, absolute paths, symlinks and anything over 50 MB. Once read, a report and its zip sit in `inbox/<id>/`, so a zip at the top of `inbox/` is one nobody has handled yet.

## cases/cases.json (data folder)

The default. If the app already keeps cases somewhere else or in another shape, `PLUMBER.md` names that file and format, and Plumber uses it instead.

Cases hold what people typed or said, so they stay out of git, which may be public. The cost is that they aren't on any branch: the owner should back up the data folder, because losing it loses the gate. An app that already keeps its cases in git can keep doing that.

A JSON array. A case is something the app must keep doing right. The cases together are the gate every fix has to pass.

```json
{"id": "2026-09-29T10-04-12.381Z", "what": "keeps the word 'ship' after a filler word",
 "added": "2026-09-29", "from": "me", "input": "um so i think we should uh ship it",
 "expect": ["\\bship it\\b"], "reject": ["\\buh\\b"],
 "check": "reads as one clean sentence", "known_failing": null}
```

- `from`: who reported it: `"me"` for the owner, or a friend's name. When several people hit the same thing, list them all (`"me, Priya"`).
- `input`: the text to replay. Always text, even a single word.
- `file`: optional. For a file input (a recording, an image), its name in `cases/files/`, copied there so log rotation can't delete it. Replay uses it instead of `input`, and a missing one is "couldn't run", not a failure.
- `expect` / `reject`: regexes the output must / must not match. Prefer these, because they're cheap and exact.
- `check`: optional. A plain-language rule for what a regex can't say. `/maintain` judges it by reading the output.
- `known_failing`: `null`, or one line on why it fails. These cases don't block the gate. Two kinds of line mean something to `/maintain`: `"not tried yet: <what's wrong>"` is a known problem it should work on, and `"fixed on maintain-YYYY-MM-DD, not merged yet"` passes on that branch but still fails on main. Anything else means a fix was tried and didn't work.

If the app calls a model and isn't deterministic, replay reruns a failed case once. Only failing twice counts.

## .plumber-state.json (data folder)

```json
{"last_run": "2026-09-29T11:00:00.000Z", "main": "c3d6970"}
```

`last_run`: `/maintain` only looks at flags, errors and signs of trouble newer than this. Write it in UTC with milliseconds, like trace times, and compare times as times, not as text: as text, `11:00:00.123Z` sorts before `11:00:00Z`. It's set to the newest time among what a run collected and the traces those point at, not the time the run ended, so nothing that arrives mid-run is skipped. A run that collects nothing leaves it alone. Reports don't use it: any zip at the top of `inbox/` is unread, however old.

`main`: the commit on main the last run started from. The next run looks for what broke a case among the commits after it. A date isn't enough, because a branch merged late keeps its commits' earlier dates.

## plumber/ (data folder)

One report per `/maintain` run, `plumber/YYYY-MM-DD-HHMM.md` (UTC, when the run started, so sorting by name puts them in order), in the shape the maintain skill gives. The next run reads the latest one.
