# Plumber: the product

*29 Sep 2026. A copy of the product note at https://claude.ai/code/artifact/fa01be00-ec21-43fe-b47c-519558d5c0c5, so everything lives in this folder. If the two differ, update this file.*

## The idea

An AI maintainer for software someone vibecoded for themselves and then started sharing with friends. It keeps the logs, notices when the app does the wrong thing, turns each failure into a test case, fixes it on a branch, and ships only what passes every case collected so far.

The bottleneck in vibecoded software isn't writing code anymore. It's the loop from *using* the app to fixing it. The author runs that loop in their head while it's just them. It breaks as soon as other people use the app.

The loop:

1. People use it (the author and friends)
2. Spot a failure (crash, undo, edit, or "that was wrong")
3. Save it as a case (with consent for each case)
4. Fix on a branch
5. Passes every case? Yes: ship the update, with rollback. No: back to step 4.

The gate at step 5 is the core: a fix reaches friends only if it passes every case saved so far, theirs and the author's.

## Evidence from Murmur

Murmur got about 60 commits in its first 4 days (built 25 Sep 2026), and most came from using it, not planning it. Every part of the maintainer already exists there, built by hand:

| Maintainer piece | How Murmur does it today |
|---|---|
| Logs that capture intent | `traces.jsonl`: raw transcript, unsure words, whether Claude ran, output, timings. The recordings are kept with it. |
| Failures become tests | `regress.py` replays past recordings against the current build. Commits like "Catch more grammar slips found in past dictations" came from it. |
| Implicit feedback | The learner watches the text box for 90 s and adds a name to the library when you fix it. A bug report nobody had to file. |
| Regular maintenance | The daily `cleaning-up-murmur` review, on dated `review-YYYY-MM-DD` branches, merged after `swift test` and `e2e.sh` pass. |
| Institutional memory | NOTES.md "What we learned", so a new fix doesn't undo an old one. |
| Onboarding others | AGENTS.md and the 29 Sep commit making setup work for someone new. |

The product is this loop, taken out of Murmur and made the default for any vibecoded app.

## Why sharing with friends is the wedge

While it's only the author, they are the maintainer: they feel every failure. Once friends use it, four things break:

1. **Friends don't report bugs, they just stop using it.** Failure has to be detected without a report: retries, undos, edits right after output, giving up partway.
2. **Their data is private.** Debugging a friend's case needs their input (in Murmur, their recording). That calls for consent per case ("send this one to the author"), not blanket telemetry.
3. **A fix for one person breaks another.** Murmur's regressions are tuned to one voice and one contact list. Every fix needs to pass a regression set shared across everyone who uses the app.
4. **Distribution.** Friends can't run `./build.sh`. They need an update channel with rollback.

## The hard question: generic or specific?

The loop is generic, but deciding what counts as a failure is specific to each app. So the eval set, not the fixing agent, is what's hard to copy.

Crashes and exceptions are generic, and error trackers such as Sentry already catch them (some with AI autofix). The failures that mattered in Murmur were about quality: "mistake wise" should have been "twice", and a question mark landed on a sentence that wasn't a question. Only something that knows what dictation is *supposed* to do can see those.

So the pitch is less "an AI that fixes your bugs" and more "an AI that learns from how people use your app what a failure is, builds the eval set, and only then fixes things." In v0, that knowledge lives in the "What counts as wrong" section of `PLUMBER.md`, written by the agent from interviewing the owner.

## Basic version (v0)

Instructions plus one command, with no server: the owner's coding agent is the maintainer, and Plumber gives it a fixed routine and a place to keep what it learns. See [README.md](README.md) for what a person sees, [SETUP.md](SETUP.md) for the install, and [skills/maintain/SKILL.md](skills/maintain/SKILL.md) for the routine.

**Left out of v0:** dashboards, detecting failures from undos or edits, auto-updating friends' copies, a hosted inbox, the scorecard.

**First users:** Murmur (Raunaq maintains it, Anil sends reports as a user) and Jarvis (Anil's own app, `jarvis-hq`, where Anil runs `/maintain`). Running the setup on Murmur checks the instructions against an app that already does most of this. Jarvis is the real test because it isn't Murmur.

## Risks

- **Willingness to pay.** Solo builders with 5–20 friends as users are a real group, but they don't pay much. It gets stronger as the bridge from "a tool I made" to "a small product", or sold to the platforms people vibecode on.
- **Trust in autonomous fixes.** A fix that looks right can quietly break something else. Without a strong eval gate, the maintainer becomes a source of bugs.
- **Drift into observability.** Datadog, PostHog and Sentry own dashboards and error tracking. This has to stay "a maintainer for someone who isn't one".

## Next step

Give Murmur to 3 friends and log every failure they hit, plus how you found out about it. If most only surface because you asked, detecting failures without a report is the product. If they arrive as complaints on WhatsApp, the product may just be a better "report this" button plus a shared regression set.

- [ ] Pick 3 friends (Anil first) and get Murmur running for them
- [ ] Keep a failure log: what broke, how you heard, what fixed it
- [ ] Install Plumber in Murmur, then in Jarvis

## Open questions

- Who pays: the builder, their friends, or the platform they build on?
- What's the smallest consent step that lets a friend share one failing case?
- Can the "what counts as a failure" check be learned from usage, or does the author always have to define it?
