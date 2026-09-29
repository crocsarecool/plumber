# Plumber

> **Coding agents:** if someone asked you to set up Plumber, follow [SETUP.md](SETUP.md).

A maintainer for software you built for yourself and then shared with a few friends.

Your coding agent already writes the code. Plumber gives it a routine for looking after the app once people use it. Every time the app gets something wrong, that becomes a saved case. Fixes happen on a branch, and a fix only ships if it passes every case saved so far, yours and your friends'.

There's no server and no account. Everything lives in your repo and your app's data folder.

## Install

Open your coding agent (Claude Code, Codex or OpenCode) in your app's repo and say:

> Set up Plumber from github.com/crocsarecool/plumber

It reads [SETUP.md](SETUP.md), looks at your app, asks you three questions, and wires everything in. This takes about 10 minutes.

## Two things to learn

1. **"That was wrong"** in your app. It marks the last thing the app did as wrong, with an optional note. Your friends get **Send to <you>**, which saves one small file they can look inside before sending it to you over WhatsApp, AirDrop or any other way.
2. **`/maintain`** in your agent. It collects what went wrong since the last run, saves each failure as a case, fixes what it can on a branch, checks the fix against every case, and tells you in three lines what happened. You decide whether to merge.

## What it adds to your app

| File | Where | What it holds |
|---|---|---|
| `PLUMBER.md` | repo | What the app is for, what counts as wrong, and how to build, test, replay and ship |
| `LEARNED.md` | repo | What each fix taught. Read before every fix so old fixes don't get undone. |
| `traces.jsonl` | data folder | One line per thing the app did |
| `flags.jsonl` | data folder | Every "that was wrong" |
| `cases/` | data folder | Failures saved so they can be replayed against any build |
| `inbox/` | data folder | Friends' reports waiting for `/maintain` |

The data folder never goes into git, because it holds what people typed, said or asked. Nothing leaves anyone's machine unless they choose to send a report.
