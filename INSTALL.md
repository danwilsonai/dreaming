# Install (instructions for Claude)

The user asked you to set up dreaming. Read `dreaming.md` in this repo first (download the exact text from https://raw.githubusercontent.com/danwilsonai/dreaming/main/dreaming.md, for example with `curl`, because web fetch tools may return a summary instead): it is the whole spec, and everything below is how to build it into this user's setup.

There is no code to download from this repo. You write the dream yourself, in the user's setup, so they can read every line of it.

**0. Ask first.** Before you change anything, tell the user in a few lines what you're about to build (the parts below), that the dream reads their session logs on this machine through their own Claude login, that it only proposes changes and never makes them, and that nothing gets scheduled without their say. Wait for them to say go.

If the user is on claude.ai rather than Claude Code, there are no session logs to read: tell them so and stop. If their Claude Code already runs its own memory consolidation, tell them, and let them choose which one runs, so two don't run at once.

**1. Find the sessions.** Claude Code keeps each session as a JSONL file under `~/.claude/projects/<project folder>/`, one JSON event per line with a timestamp. The format is internal and changes between versions, so open a recent file and confirm what the user's messages, the assistant's replies, tool calls and tool errors look like before you write anything that parses them. Note which sessions are the user typing and which are scheduled or headless runs; a headless run usually only matters when a tool failed.

**2. Write the dream.** Pick the simplest form that fits the user's setup (a script, a skill or both) and follow the house style of whatever is already there. It:
- builds a digest of every session since the last dream, with a size cap so a huge session can't crowd out the rest, in a temporary folder it deletes afterwards
- runs a separate Claude call over the digests with the spec from `dreaming.md`, the user's instruction and memory files to check against, and the decisions on earlier proposals. That call gets read-only tools and leaves no session log of its own (with the Claude Code CLI, `claude -p` with `--tools "Read,Grep,Glob"` and `--no-session-persistence`; check `claude --help` for the user's version)
- writes a dated report the user can read: each proposal numbered, with its root cause, the exact change and its evidence
- records the time of the run, so the next dream starts where this one stopped

**3. Let the user decide.** Give them a simple way to say "apply 2" or "skip 1" to Claude in a later session: a short skill or an instruction in their CLAUDE.md that says where the reports live, how to apply a proposal (read the files as they are now, then make the change) and how to record the decision. Rejected proposals stay recorded so the dream doesn't raise them again without new evidence.

**4. Try it once.** Run it on the last day or two of sessions, show the user the report, and fix the dream if it proposes something weak or misses something obvious.

**5. Schedule it only if they want.** Ask whether they want it nightly or on demand. If nightly, use what their machine has (cron, launchd, Task Scheduler), at an hour they choose, and tell them how to turn it off. A night with no proposals should stay quiet.
