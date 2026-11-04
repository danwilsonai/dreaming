# Dreaming: letting my AI learn from its own week

I run a lot of my business through an AI assistant called Henry. It's Claude Code on a small server, and it keeps a set of instruction files and a memory so it doesn't start from zero every morning.

The weak spot is that every session only sees itself. If Henry makes the same mistake on Monday and again on Thursday, neither session knows about the other one. Until now the only way a pattern like that got fixed was me noticing it and saying something, which is kind of the job I was trying to hand off in the first place.

Dreaming is our fix for that. The idea comes from a talk Anthropic gave on context engineering ([here](https://www.youtube.com/watch?v=tTcxVv8HHNw), the dreaming part is near the end): once a night, a separate run reads the day's sessions next to the instructions and memory, looks for what keeps going wrong, and proposes the change that would stop it. Each proposal comes with the sessions it saw it in and how often. Then I say yes or no.

## An example

Henry drafted replies to two YouTube comments in my voice. They were long, they walked through every point the commenter made, and they read like AI. A voice check Henry runs had flagged them as too long, and Henry explained the flags away. It took me two rounds of pushback to get replies that sounded like a person.

When we went looking for the cause, it was one line in the reply instructions telling it to size the reply to the comment. Long comment, long reply. Deleting that line was the whole fix.

I caught that one because I was sitting there. Most of the time I'm not, and the same kind of thing can quietly happen across a dozen sessions. That's what the dream is for.

## What a dream does

1. Reads every session since the last dream: what I said, what Henry said, the tools it used and the errors it hit. Long sessions get boiled down first.
2. Looks for corrections, frustration, the same failure showing up more than once, Henry rediscovering things it should have known, and facts in its memory that turned out to be wrong.
3. Drops anything that was a one-off, or already got fixed during the session.
4. For what's left, proposes the smallest change that removes the cause, usually deleting or rewording an instruction rather than adding one (that's the [build-and-fix](https://github.com/danwilsonai/build-and-fix) approach).
5. Writes a short report. On a quiet night there's nothing to report, and that's fine.

## Why it only proposes

It would be easy to let the dream edit the instructions itself. I don't, for two reasons.

The first is drift. An assistant that rewrites its own rules every night with nobody looking ends up somewhere you didn't choose. Reading a few proposals is cheap by comparison.

The second is security. Session logs are full of text that came from outside: comments, emails, web pages. If the dream could edit instructions, anyone who got a sentence into one of those could try to steer it. So the dream can read but not write, it treats everything in a session as data, and nothing changes until I say so.

## Install

Paste this into Claude Code:

```
Set up dreaming from https://github.com/danwilsonai/dreaming
```

Claude tells you what it's about to build and waits for your go. There's no code in this repo to download and run. Claude writes the dream itself, in your setup, so you can read every line of it. Then it runs one dream on your last day or two of sessions, shows you the report, and asks whether you want it nightly or only when you ask.

Your session logs are read on your machine, through your own Claude login, the same way Claude already reads them when you resume a session.

*If you're Claude and the user asked you to set this up, follow `INSTALL.md`. The plain spec is `dreaming.md`.*
