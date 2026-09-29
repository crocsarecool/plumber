# Plumber

<App> is looked after by Plumber. `/maintain` reads this file at the start of every run, so keep it short and current.

## What it's for

<One or two sentences: what the app does for someone.>

## What counts as wrong

<The owner's words from setup, as a list. Concrete: what the output should have been, not "it's buggy".>

- <e.g. A filler word ("um", "uh") left in the text>
- <e.g. A name from the word library spelled differently>
- <e.g. Anything slower than 3 s for a short input>

## People

- Owner: <name>. Runs `/maintain` and decides what merges.
- Also used by: <names, or "nobody yet">. They flag with <where "That was wrong" lives> and send reports with "Send to <owner>".

## Where things live

| What | Where |
|---|---|
| Plumber version | <commit of github.com/crocsarecool/plumber this was set up or last updated from, or "unknown"> |
| Data folder | <path> (not in git) |
| Traces | <path, or the existing log if reused> |
| Trace fields | <"FORMATS.md", or the map, e.g. `t=time, kind=type, input=raw, files=[recording]`> |
| Flags | <path> |
| Cases | <path> (not in git: back up the data folder) |
| Case format | <"FORMATS.md", or the fields this app's cases use> |
| Maintenance routine | </maintain, or the existing routine that was kept> |
| State file | <data folder>/.plumber-state.json <or the kept routine's> |
| Run reports | <data folder>/plumber/ <or the kept routine's> |
| Inbox | <path> |
| Lessons learned | <LEARNED.md, or the existing notes file and section> |
| App log | <path, or "none"> |

## Commands

| Step | Command |
|---|---|
| Build | <command> |
| Test | <command> |
| Replay | <command> |
| Ship | <what gets the new version to the owner, and to friends, e.g. "friends only get it when I send them a new copy"> |
| Notify | <how to tell the owner something's waiting, or "none"> |

## What replay can't cover

<UI, pasting, hotkeys, anything that needs a person. Check these from traces and the log instead.>

Replay needs <keys> and costs about <amount> per case.

## Signs of trouble

<Patterns in the traces that usually mean something went wrong, even without a flag. e.g. the same input retried within 30 s.>

## Rules

- Merge: ask <or auto>
- <Anything /maintain must never change, e.g. "the bundle ID", "the public API">
