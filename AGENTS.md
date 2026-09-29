# Notes for coding agents

Plumber is a set of instructions, not a program. It turns you, the person's coding agent, into the maintainer of an app they built and shared with a few friends. There's nothing to install or run from this repo. You read these files and do what they say in the person's own app.

## First, work out what the person wants

Someone pointed you at this repo. It's usually one of four things. If it isn't clear, ask in one line and give the options.

| They say something like | What to do |
|---|---|
| "Set up Plumber", "add Plumber to my app" | **Set up.** Follow [SETUP.md](SETUP.md) in *their app's* repo, not this one. |
| "I use <someone>'s app and it got something wrong" | **Report as a friend.** See below. |
| "Update Plumber" | **Update.** See below. |
| "Work on Plumber", "change the setup steps" | **Work on this repo.** Read [NOTES.md](NOTES.md) and [PRODUCT.md](PRODUCT.md) first. |

If you're already inside an app that has a `PLUMBER.md`, Plumber is set up. The person probably wants `/maintain` (the owner) or to report something (a friend).

## Getting the files

You need this repo's files while working in the person's app. Use whichever works:

- **You have the path or a local clone:** read the files from there.
- **You have the GitHub link:** `git clone --depth 1 https://github.com/crocsarecool/plumber <scratch dir>/plumber`, then read from there. Don't clone it inside their app's repo.
- **You can only fetch web pages:** read each file at `https://raw.githubusercontent.com/crocsarecool/plumber/main/<path>`.

For setup you need [SETUP.md](SETUP.md), which tells you when to read [FORMATS.md](FORMATS.md), the two [templates](templates/), and [skills/maintain/SKILL.md](skills/maintain/SKILL.md). A friend's report only needs this file and FORMATS.md.

## Set up

Go to the person's app repo, and ask them for the path if you aren't in it. Then follow [SETUP.md](SETUP.md) from step 1. It's written for you, and the person should only have to answer a few questions.

## Report as a friend

The person uses an app someone else built (the owner), and it did the wrong thing. They didn't build the app and may not code, so do the digging yourself and keep your questions plain.

1. **If the app has "That was wrong"** (a menu item, a button, a `wrong` command), tell them where it is. Then have them use **Send to <owner>**. You're done.
2. **If it doesn't,** make the report by hand, in the shape [FORMATS.md](FORMATS.md) gives under "Reports (friends)":
   1. **Find the data folder yourself.** Look in the app's `PLUMBER.md`, README or AGENTS.md, and in its config or env file. Ask them only if all of those fail.
   2. **Find what went wrong.** Read only the last few lines of the traces log, not the whole history. Skip lines with no id, since they aren't things the app did. Show the latest few as time, app and what came out, and ask which one it was.
   3. **Ask two things:** what was wrong and what it should have been, which becomes the flag's note, and what name the owner knows them by.
   4. **Show them what would go in.** Show only what they gave the app, what came out, and which app they were in. Say that the rest is timing data. Open each file for them (`open <file>` on a Mac) and ask yes or no for each one. Leave out screenshots, window text and anything else that could show other people's messages unless they ask for it. Tell them the owner will replay it through the app, which may send it to the services the app uses.
   5. **Save** `<app>-report-<trace id>.zip` to their Downloads, and tell them to send it to the owner any way they like.
3. **Never upload anything or send it for them.** The report holds what they typed, said or asked. It goes to the owner only when they send it themselves. In your own replies, refer to it by id and time, and don't repeat its text back more than you need to.

If they want to install or update the owner's app, that's in the app's own README or AGENTS.md, not here.

## Update

The app already has Plumber, and the person wants the latest version.

Work on a branch named `plumber-update`.

1. **Find what changed.** `PLUMBER.md` → "Plumber version" says which commit of this repo the app was set up from. Run `git log -p <that commit>..HEAD -- SETUP.md FORMATS.md skills templates` here, and read [NOTES.md](NOTES.md) for why. If the version is missing or `unknown`, or you can't run git here, go through SETUP.md and FORMATS.md step by step and compare the app with each one.
2. **The maintenance routine.** If `PLUMBER.md` → "Maintenance routine" is `/maintain`, replace the app's `.claude/skills/maintain/SKILL.md` with [skills/maintain/SKILL.md](skills/maintain/SKILL.md). If it names a routine the owner kept, don't install `/maintain`, because two routines would run. Update that routine's Plumber section instead, and only with the owner's yes if it lives outside the repo.
3. **Everything else** that the app is missing, apply the same way SETUP.md would.
4. **The app's own things.** `PLUMBER.md`, the lessons-learned file, cases and data belong to the app. If a change needs them changed (a new field, a new marker in `known_failing`), say what and why, and change them only on the owner's yes. The one exception: set "Plumber version" in `PLUMBER.md` to the commit you updated from.
5. Tell the owner in a few lines what changed, and merge only on their yes.

## Working on this repo

- Plain Markdown, no code. Keep the writing short and plain, with one idea per paragraph.
- SETUP.md and the maintain skill are read by agents in other people's repos. Every step must work without this conversation, and without asking the owner anything the repo can answer.
- When you change a step, check it against Murmur (`~/Experiments/Murmur` on Raunaq's Mac). NOTES.md shows how the last test run was done.
- Record why things are the way they are in NOTES.md.
