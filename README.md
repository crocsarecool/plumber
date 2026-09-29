# Plumber

> **Coding agents:** start with [AGENTS.md](AGENTS.md).

A maintainer for software you built for yourself and then shared with a few friends.

Your coding agent already writes the code. Plumber gives it a routine for looking after the app once people use it. Every time the app gets something wrong, that becomes a saved case. Fixes happen on a branch, and a fix only ships if it passes every case saved so far, yours and your friends'.

There's no server and no account. Everything lives in your repo and your app's data folder.

## What to tell your agent

Open your coding agent (Claude Code, Codex or OpenCode) and point it at this repo. There's nothing to install. It reads the instructions and does the rest.

| You are | Say |
|---|---|
| Building an app | "Set up Plumber in this app from github.com/crocsarecool/plumber" |
| Using a friend's app, and it got something wrong | "I use <friend>'s <app> and it got something wrong. Help me report it, using github.com/crocsarecool/plumber" |
| Already using Plumber | "Update Plumber from github.com/crocsarecool/plumber" |

Setup looks at your app, asks you a few questions, and wires everything in on a branch. It takes about 10 minutes of your time, and the agent does the rest.

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
