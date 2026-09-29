# Gitbot

Gitbot runs Plumber through git. You clone this repo and open this folder, instead of pasting a prompt into your agent.

## The step

```bash
git clone --depth 1 https://github.com/crocsarecool/plumber.git /tmp/plumber
```

Put the clone in a scratch folder. Don't put it inside the app.

## What happens next

Open `/tmp/plumber/gitbot` in your coding agent. It reads the instructions in this folder and follows them in your app: set Plumber up, report something that went wrong, or update.

Cases stay in the app's data folder, on that machine. A fix waits on a branch until you merge it. A maintain run makes at most two fixes. Nothing uploads what people typed, said, or asked.

Grokbot is the other door. Gitbot does not call it.
