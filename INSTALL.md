# Install (instructions for Claude)

The user asked you to set up dreaming. Read `dreaming.md` in this repo first (download the exact text from https://raw.githubusercontent.com/danwilsonai/dreaming/main/dreaming.md, for example with `curl`, because web fetch tools may return a summary instead): it is the whole spec, and everything below is how to build it into this user's setup. The README's "How it's built" section shows the shape you're aiming for.

There is no code to download from this repo. You write the dream yourself, in the user's setup, so they can read every line of it. Follow the house style of whatever is already there, and keep it as small as the setup allows.

**0. Look, then ask.** Take stock of what this user actually has, reading only:
- which agents run here: Claude Code sessions (each one a JSONL file under `~/.claude/projects/<project folder>/`), headless or SDK runs, other agent tools with their own logs
- what runs on its own: `crontab -l`, systemd timers, launchd agents, Task Scheduler, CI schedules in their repos (`gh run list` if GitHub), cloud schedulers you can see from here, and anything their own docs say should be running
- where those processes log and alert, and where their output goes
- which repos agents commit to, and whether drafts an agent writes get edited by a person before they go out

Then tell the user in a few lines what you found and what the dream would read, that it reads on this machine with their existing access and their own Claude login, that it only proposes changes and never makes them, and that nothing gets scheduled without their say. Let them leave anything out. Wait for them to say go.

If the user is on claude.ai rather than Claude Code, there is nothing here to read: tell them so and stop. If their Claude Code already runs its own memory consolidation, tell them, and let them choose, so two don't run at once.

**1. Write the inventory.** One file listing every process that should run on its own: what it's for, how often it should run, what runs it, where it logs, what it produces, what a good run looks like and whether it writes nothing when there's nothing to do. Describe a good run by what it produced (an output file that's fresh, a new row, a sent message with every section in it), not by its exit code, because the failures worth finding exit cleanly. Where two jobs share one log, suggest splitting them. Fill in what you can see, and ask the user only about what you can't: which jobs still matter, and what "good" means where the logs don't say. A job nobody can name a purpose for is worth raising now.

**2. Write the collector.** A plain script, no model calls, that reads the inventory and writes the evidence to a temporary folder as plain text:
- for each process, over its last several runs (enough to see a pattern at its cadence, so more calendar time for weekly and monthly jobs): whether it ran when the inventory says it should, exit codes, retries, durations against its usual, whether its output exists, is fresh and changed, error-like lines in its log even when the run reported success, and the alerts it sent. Tell a missed run from a quiet one with run records, with whether the log file exists at all (`>>` creates it when a cron job starts), and with when the machine last restarted, since a reboot empties temporary folders
- anything the scheduler runs that isn't in the inventory, and anything in the inventory the scheduler no longer runs
- a digest of every agent session since the last dream: the user's words in full, the agent's replies trimmed, tool calls and tool errors, with a size cap so one huge session can't crowd out the rest. Session logs, CI output and log files are internal formats that change between versions, so open a recent example of each before you write anything that parses them
- where agents commit code: reverts and fix-ups soon after an agent's commit, changes requested on agent pull requests, CI results on agent commits, using read-only commands
- where a person edits agent drafts: the diff between the draft and what went out
- the instruction and memory files: every path and file they mention that no longer exists, links in the memory index that point nowhere, dated or conditional notes ("delete this once...", "watch until..."), and roughly how many tokens load into every session, file by file

Count, date and diff everything in code. The model's job is judgment, not arithmetic.

**3. Write the dream run.** A script that runs the collector, then makes one separate Claude call over the evidence with the spec from `dreaming.md`, the instruction and memory files to check against, the watch list and the decisions on earlier proposals. That call gets read-only tools and leaves no session log of its own (with the Claude Code CLI, `claude -p` with `--tools "Read,Grep,Glob"` and `--no-session-persistence`; check `claude --help` for the user's version). The script then writes a dated report (each proposal numbered, with its root cause, the exact change, its evidence and how to test it), saves the watch list and the time of the run so the next dream starts where this one stopped, and deletes the evidence folder. Sessions need a nightly pass; processes and code history only need the deeper pass weekly. A run with nothing to propose stays quiet, and one with proposals sends only their titles, with the details left in the report.

**4. Write the apply step.** A short skill, or an instruction in their CLAUDE.md, so the user can say "apply 2" or "skip 1" to Claude in any later session. It says where the reports live, how to apply a proposal (read the files as they are now, then make the change) and how to record the decision. Decisions stay recorded, so the dream doesn't raise a rejected proposal again without new evidence.

**5. Try it once.** Run it for real (not a dry run, so its findings are saved) with the process pass included over the user's recent history, show them the report, and fix the collector or the dream if it proposes something weak or misses something obvious.

**6. Schedule it only if they want.** Ask whether they want it on a schedule or on demand. If scheduled, use what their machine has, at an hour they choose, add the dream itself to the inventory, and tell them how to turn it off.
