# Dreaming: letting your AI agents learn from their own week

AI agents keep instruction files and memory so they don't start from zero every session. The weak spot is that every session only sees itself. If an agent makes the same mistake on Monday and again on Thursday, neither session knows about the other one, and the pattern only gets fixed if a person notices it.

The scheduled side is worse. A nightly job that quietly stops doing its job doesn't open a session or ask for help. It keeps "succeeding" until someone happens to look, which can take months.

Dreaming is a separate run, on a schedule, that looks across all of it at once. It reads the recent sessions, the record of every scheduled process and the code history, and checks the instruction and memory files the agents run on. It finds what keeps going wrong or quietly stopped working, and proposes the change that would fix the cause. Each proposal comes with its evidence and how often it showed up. Then you say yes or no.

The idea comes from a talk Anthropic gave on context engineering ([here](https://www.youtube.com/watch?v=tTcxVv8HHNw), the dreaming part is near the end). This repo extends it from agent sessions to the processes around them.

## What a dream reads

- **Agent sessions** since the last dream: what you said, what the agent said, the tools it used and the errors it hit. Headless and scheduled agent runs too, not just the ones you typed in.
- **Scheduled processes** over their last several runs, however often they run: cron jobs, systemd timers, launchd, CI workflows, cloud schedulers. Their exit codes, retries, durations, log tails, output and the alerts they sent.
- **Code history**, where agents write code: reverts and quick fix-ups right after an agent's commit, changes reviewers asked for on agent pull requests, CI failures on agent commits.
- **Human edits to agent output**: what a person changed in a draft before it went out. It's the strongest signal there is, because it says exactly what the agent got wrong.
- **The instruction and memory files themselves**: files and paths they mention that no longer exist, dated notes whose date or condition has passed, two files that say different things, and how much of it loads into every session.

## What it looks for

- Corrections, repeated instructions and frustration
- The same mistake or tool failure across sessions
- Agents spending many steps rediscovering something their instructions should have told them
- Facts in memory that turned out to be wrong, and notes whose date or condition has passed
- An agent that noticed a problem and moved on without fixing it
- Processes that "succeed" but hit an error along the way and carried on
- Processes that succeed but produce nothing, the same thing every time, or something nobody uses
- Processes that retry most nights, keep getting slower or more expensive, or depend on a credential that's about to expire
- Checks that can't actually see what they claim to check
- Alerts that keep firing and never get acted on
- Processes that no longer serve a goal, which is a proposal to delete them, not to fix them

## How it decides

Anything that was a one-off, or already got fixed, gets dropped. Something real that showed up only once goes on a watch list, and becomes a proposal if it shows up again, so a weekly job's problem can still add up to a pattern. The exception is a bug in a scheduled process: it repeats on every run, so one run is enough evidence. For what's left, the dream proposes the smallest change that removes the cause, usually deleting or rewording an instruction rather than adding one (the [build-and-fix](https://github.com/danwilsonai/build-and-fix) approach), along with how to test it: on the case that failed and on one that already worked.

Sessions get a light nightly pass. Scheduled processes and code history get a deeper weekly pass, since a pattern in a weekly job needs weeks of runs to show. A quiet night produces no report, and that's fine.

## How it's built

Five pieces, all written by Claude in your own setup:

1. **An inventory of every process.** Claude takes stock of everything that runs on its own (crontab, systemd timers, launchd, CI schedules, cloud schedulers, scheduled agent runs) and writes it into one file. For each process: what it's for, how often it should run, where it logs, what it produces, and what a good run looks like, described by what it produced rather than its exit code, because the worst failures exit cleanly. You go through it once and confirm, because you're the one who knows which jobs still matter. It's the list of what *should* be running, so the dream catches a job that dropped off the schedule, not just one that's failing on it.
2. **A collector script.** Plain Python, no AI in it. It reads the inventory and gathers the facts: whether each process ran when it should have, its exit codes, retries and durations against its usual, whether its output exists and changed, the error lines in its log (including from runs that reported success) and the alerts it sent. It also builds a digest of each agent session since the last dream, checks every path the instruction and memory files mention, lists their dated notes, measures what loads into every session and, for code, pulls the reverts, fix-ups and CI results after agent commits. The script does all the counting, so the model only has to judge.
3. **The dream run.** A script that runs the collector, then makes one Claude call with read-only tools over that evidence, next to the instructions, the memory, the watch list and your past decisions. It writes a dated report, updates the watch list and deletes the evidence. The notification you get carries only the titles; the evidence and the exact changes live in the report.
4. **An apply step.** A short skill, so you can tell Claude "apply 2" or "skip 1" in any later session. It makes the change, and the decision gets recorded.
5. **A schedule.** Nightly for sessions, weekly for the deeper process pass, at an hour you pick. Or only when you ask.

Laid out, it can be as small as this:

```
dreaming/
  processes.yaml        the inventory: what should run, and what a good run looks like
  collect.py            gathers the evidence, no AI
  dream.py              runs collect.py, then the read-only Claude call, writes the report
  reports/2026-11-05.md proposals with their evidence and your decisions
  watch.json            things seen once, waiting to see if they come back
```

One inventory entry might look like:

```yaml
- name: nightly-backup
  purpose: push config and scripts to the private backup repo
  schedule: daily 03:00
  runs_from: crontab
  log: ~/logs/backup.log
  output: a new commit on the backup repo
  good_run: a new commit within 26 hours, and no error lines in the log
  quiet: false        # true for jobs that write nothing when there's nothing to do
```

When the scheduler and the inventory disagree (a new job nobody listed, a listed job that's gone), the dream proposes updating one or the other.

Two details matter more than they look. Some jobs write nothing when there's nothing to do, so an old log doesn't mean a job stopped; mark those as quiet and lean on run records instead. And logs kept somewhere temporary disappear on a reboot, so the collector notes when the machine last restarted. Give each job its own log, too, because two jobs sharing one muddies the evidence for both.

## From a real run

On one real setup, a dream over 5 agent sessions and 56 scheduled processes took about two and a half minutes and proposed three changes. The biggest was a daily report that had been "succeeding" every morning: it exited cleanly and said it was sent, while an error it caught and logged meant a whole section was missing. No exit-code check would have found it. The other two were memory notes the record had overtaken, one waiting on something that had already happened and one describing a bug that was already fixed. The instruction-file check, added after that run, found 23 references to files that no longer existed.

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
