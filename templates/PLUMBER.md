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
| Data folder | <path> (not in git) |
| Traces | <path, or the existing log if reused> |
| Flags | <path> |
| Cases | <path> |
| Inbox | <path> |
| Lessons learned | <LEARNED.md, or the existing notes file and section> |
| App log | <path, or "none"> |

## Commands

| Step | Command |
|---|---|
| Build | <command> |
| Test | <command> |
| Replay | <command> |
| Ship | <what gets the new version to the owner and to friends> |

## Rules

- Merge: ask <or auto>
- <Anything /maintain must never change, e.g. "the bundle ID", "the public API">
