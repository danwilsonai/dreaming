# Dreaming: letting your AI agents learn from their own week

AI agents keep instruction files and memory so they don't start from zero every session. The weak spot is that every session only sees itself. If an agent makes the same mistake on Monday and again on Thursday, neither session knows about the other one, and the pattern only gets fixed if a person notices it.

The scheduled side is worse. A nightly job that quietly stops doing its job doesn't open a session or ask for help. It keeps "succeeding" until someone happens to look, which can take months.

Dreaming is a separate run, on a schedule, that looks across all of it at once. It reads the recent sessions, the record of every scheduled process and the code history, next to the instructions and memory the agents run on. It finds what keeps going wrong or quietly stopped working, and proposes the change that would fix the cause. Each proposal comes with its evidence and how often it showed up. Then you say yes or no.

The idea comes from a talk Anthropic gave on context engineering ([here](https://www.youtube.com/watch?v=tTcxVv8HHNw), the dreaming part is near the end). This repo extends it from agent sessions to the processes around them.

## What a dream reads

- **Agent sessions** since the last dream: what you said, what the agent said, the tools it used and the errors it hit. Headless and scheduled agent runs too, not just the ones you typed in.
- **Scheduled processes** over their last several runs, however often they run: cron jobs, systemd timers, launchd, CI workflows, cloud schedulers. Their exit codes, retries, durations, log tails, output and the alerts they sent.
- **Code history**, where agents write code: reverts and quick fix-ups right after an agent's commit, changes reviewers asked for on agent pull requests, CI failures on agent commits.
- **Human edits to agent output**: what a person changed in a draft before it went out. It's the strongest signal there is, because it says exactly what the agent got wrong.

## What it looks for

- Corrections, repeated instructions and frustration
- The same mistake or tool failure across sessions
- Agents spending many steps rediscovering something their instructions should have told them
- Facts in memory that turned out to be wrong
- Processes that succeed but produce nothing, the same thing every time, or something nobody uses
- Processes that retry most nights, keep getting slower or more expensive, or depend on a credential that's about to expire
- Checks that can't actually see what they claim to check
- Alerts that keep firing and never get acted on
- Processes that no longer serve a goal, which is a proposal to delete them, not to fix them

## How it decides

Anything that was a one-off, or already got fixed, gets dropped. Something real that showed up only once goes on a watch list, and becomes a proposal if it shows up again, so a weekly job's problem can still add up to a pattern. For what's left, the dream proposes the smallest change that removes the cause, usually deleting or rewording an instruction rather than adding one (the [build-and-fix](https://github.com/danwilsonai/build-and-fix) approach).

Sessions get a light nightly pass. Scheduled processes and code history get a deeper weekly pass, since a pattern in a weekly job needs weeks of runs to show. A quiet night produces no report, and that's fine.

## Why it only proposes

It would be easy to let the dream edit the instructions itself. It doesn't, for two reasons.

The first is drift. Agents that rewrite their own rules every night with nobody looking end up somewhere you didn't choose. Reading a few proposals is cheap by comparison.

The second is security. Session logs and process logs are full of text that came from outside: comments, emails, web pages, API responses. If the dream could edit instructions, anyone who got a sentence into one of those could try to steer it. So the dream can read but not write, it treats everything it reads as data, and nothing changes until you say so.

## Install

Paste this into Claude Code:

```
Set up dreaming from https://github.com/danwilsonai/dreaming
```

Claude tells you what it's about to build and waits for your go. There's no code in this repo to download and run. Claude looks at what you actually have (which agents, which schedulers, which repos) and writes the dream itself, in your setup, so you can read every line of it. Then it runs one dream on your recent history, shows you the report, and asks whether you want it on a schedule or only when you ask.

Everything is read on your machine, through your own Claude login and the access you already have.

*If you're Claude and the user asked you to set this up, follow `INSTALL.md`. The plain spec is `dreaming.md`.*
