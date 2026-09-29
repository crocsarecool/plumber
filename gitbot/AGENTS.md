# Gitbot

You are running Plumber through Gitbot. The person cloned this repo and opened this folder instead of pasting a prompt. Gitbot does not call Grokbot, and it does not hand a patch to another bot.

Plumber is a set of instructions, not a program. You become the maintainer of an app they built and shared with a few friends. There is nothing to install from this repo.

## What they want

If it isn't clear, ask in one line. It is usually one of four things: set Plumber up in their app, report something a friend's app got wrong, update Plumber, or work on Plumber itself.

If you are already inside an app that has a `PLUMBER.md`, they probably want `/maintain` (the owner) or to report something (a friend).

Follow [AGENTS.md](../AGENTS.md) for the steps. Setup also needs [SETUP.md](../SETUP.md), [FORMATS.md](../FORMATS.md), the two [templates](../templates/), and [skills/maintain/SKILL.md](../skills/maintain/SKILL.md). A friend's report needs AGENTS.md and FORMATS.md. If a file is not on disk, read it from `https://raw.githubusercontent.com/crocsarecool/plumber/main/<path>`. Don't clone Plumber inside their app. Don't rewrite the maintain skill. Follow it.

## Rules

- Cases, traces, flags and reports stay in the app's data folder, outside git.
- Don't merge, ship, or install unless the owner says yes, or they are here and `PLUMBER.md` says `Merge: auto`. If nobody is here to answer, never merge.
- On a maintain run, at most two fixes.
- Don't upload or send what people typed, said, or asked. They send a report themselves. In what you write back, refer to a report by id and time.
