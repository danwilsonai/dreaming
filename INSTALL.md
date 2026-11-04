# Install (instructions for Claude)

The user asked you to set up dreaming. Read `dreaming.md` in this repo first (download the exact text from https://raw.githubusercontent.com/danwilsonai/dreaming/main/dreaming.md, for example with `curl`, because web fetch tools may return a summary instead): it is the whole spec, and everything below is how to build it into this user's setup.

There is no code to download from this repo. You write the dream yourself, in the user's setup, so they can read every line of it.

**0. Look, then ask.** Take stock of what this user actually has, reading only:
- which agents run here: Claude Code sessions (each one a JSONL file under `~/.claude/projects/<project folder>/`), headless or SDK runs, other agent tools with their own logs
- what runs on a schedule: `crontab -l`, systemd timers, launchd agents, Task Scheduler, CI workflows in their repos (`gh run list` if GitHub), cloud schedulers you can see from here
- where those processes log and alert, and where their output goes
- which repos agents commit to, and whether drafts an agent writes get edited by a person before they go out

Then tell the user in a few lines what you found and what the dream would read, that it reads on this machine with their existing access and their own Claude login, that it only proposes changes and never makes them, and that nothing gets scheduled without their say. Let them leave anything out. Wait for them to say go.

If the user is on claude.ai rather than Claude Code, there is nothing here to read: tell them so and stop. If their Claude Code already runs its own memory consolidation, tell them, and let them choose, so two don't run at once.

**1. Learn the formats.** Session logs, CI output and log files are internal formats that change between versions. Open a recent example of each source you'll read and confirm what the user's messages, the agent's replies, tool calls, tool errors and a process's runs look like before you write anything that parses them.

**2. Write the dream.** Pick the simplest form that fits the user's setup (a script, a skill or both) and follow the house style of whatever is already there. It:
- builds the evidence with code: a digest of every session since the last dream, and for each scheduled process its last several runs (enough to see a pattern at its cadence), with the counts, dates, durations and diffs computed, not left to the model. Cap the size so one huge source can't crowd out the rest, and keep it in a temporary folder deleted afterwards.
- runs a separate Claude call over that evidence with the spec from `dreaming.md`, the instruction and memory files to check against, the watch list and the decisions on earlier proposals. That call gets read-only tools and leaves no session log of its own (with the Claude Code CLI, `claude -p` with `--tools "Read,Grep,Glob"` and `--no-session-persistence`; check `claude --help` for the user's version). Anything it reads with outside tools, such as `gh`, uses read-only commands.
- writes a dated report the user can read: each proposal numbered, with its root cause, the exact change and its evidence. A run with nothing to propose stays quiet.
- keeps the watch list between runs, and records when it last ran, so the next dream starts where this one stopped. Processes and code history only need the deeper pass weekly.

**3. Let the user decide.** Give them a simple way to say "apply 2" or "skip 1" to Claude in a later session: a short skill or an instruction in their CLAUDE.md that says where the reports live, how to apply a proposal (read the files as they are now, then make the change) and how to record the decision. Decisions stay recorded, so the dream doesn't raise a rejected proposal again without new evidence.

**4. Try it once.** Run it with the process pass included over the user's recent history, show them the report, and fix the dream if it proposes something weak or misses something obvious.

**5. Schedule it only if they want.** Ask whether they want it on a schedule or on demand. If scheduled, use what their machine has, at an hour they choose, and tell them how to turn it off.
