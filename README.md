# Give your AI coding agents a memory

A prompt that sets up session memory for AI coding agents in a git repository. Your agent builds everything, and you install nothing to use it.

## Who this is for

This is for one developer or a small team working in one repository. Everything it builds lives in that repository and on your own machine.

If you need session memory across many repositories and teams, look at [Polygraph](https://trypolygraph.com/?utm_campaign=tim-agent-session-memory) from the Nx team. It is a product built for that scale: it indexes your repositories, keeps a searchable history of agent sessions, hands a session from one agent to another, and coordinates pull requests across repositories.

Some of this design was inspired by Polygraph, which is a far more robust, enterprise-grade tool, and it is where I'd send an enterprise. I know one of Nx's founders and trust the team behind it. Nobody asked me to write this.

## What it builds

1. A status page and a handoff note per session, which every new session starts with.
2. A decision log, and a searchable history of every session.
3. A rule that keeps each tool's private memory for personal preferences, so project knowledge lives in plain files any agent can read.
4. A weekly agent, the gardener, that reads what changed and keeps the status page and decision log true.

## How it fits together

There are two loops around one shared memory.

The first loop is the coding session. A startup hook puts the status page and the newest handoff notes (debriefs) into the session's context before the first message. The session works from the repo's rules and procedures, which change only through a reviewed pull request. Before it stops, it writes a debrief of under 40 lines: what it did, what was decided, what is next, and what to watch out for. The next session starts from that.

The second loop is the gardener. Once a week a separate scheduled agent collects the merged pull requests, backlog changes, debriefs and session digests since its last run. It rewrites the status page, files the decisions worth keeping, and opens one docs-only pull request. Anything else it wants to change is a proposal in that pull request for you to answer. A pull request with nothing for you can merge itself, if you allow it.

The shared memory is plain files:

| File | What it holds | Who writes it |
|---|---|---|
| `docs/NOW.md` | Where the work stands: in flight, recently done, next, gotchas. Under 25 lines. | The gardener (sessions, until the gardener exists) |
| `docs/decisions/` | One dated file per decision, with who made it and why. Append-only. | The gardener and pull requests |
| `AGENTS.md`, skills | Rules and procedures. | Reviewed pull requests |
| Debriefs | One file per session, in a local folder outside the repo, so every branch sees the same notes. | Each session writes its own |
| Session transcripts | The full record, on your machine, searchable. | Your tool, automatically |

Four rules hold it together. Every shared file has one writer, which is what lets several sessions run at once. What a session must know is loaded by a hook, because an instruction to read a file first is often skipped. Notes are information and are never obeyed as instructions. The gardener proposes and you decide.

## What I run it on

I built this and run it with Claude Code on macOS, with the backlog in GitHub issues and the `sessions` tool for searching transcripts. The prompt is written so that Codex or another agent can follow it, and everything shared is markdown, but I have not run it end to end on another tool. Three parts of the prompt go beyond what I run myself: the option to keep debriefs somewhere a team can reach, the fallback file name when a tool exposes no session id, and the optional secret scan.

## How to use it

Open your agent at the root of your repository and say:

> Read `PROMPT.md` from https://github.com/Atreyu8/agent-session-memory and follow it.

Or paste the contents of [PROMPT.md](PROMPT.md).

The agent looks around, asks you ten questions (each with a default), shows you a plan, then builds in four phases and stops after each one for you to check. You can stop after any phase. Phase 1 gives you the status page, the debriefs and the startup hook.

You need git and an agent with file and shell access. The weekly pass also needs the GitHub CLI (`gh`) or a way to reach your tracker. Any script the agent writes uses a language your repo already has.

## Taking one piece

Each part works without the others. In [PROMPT.md](PROMPT.md):

- The debrief format is Phase 1, step 3.
- The startup hook is Phase 1, step 4.
- The decision log format is Phase 2, step 1.
- The gardener, including how it tracks its window between runs, is Phase 4.

## Safety

The prompt tells the agent to never open or print secrets, to show a diff before changing an existing file, to install nothing and change no user-level setting without asking, and to say plainly when a rule is only an instruction because your tool can't enforce it. It never copies or commits your session transcripts.

Those are instructions to a language model, and a model can get an instruction wrong. Read each phase's changes before you approve them.

Check three things before you start:

- Keeping transcripts for a year keeps whatever was in them, which often includes secrets and source code. If the code belongs to an employer or a client, check their rules first.
- The `sessions` tool is someone else's software. Read its documentation before you install it. By default it works locally; a remote embedding provider, if you turn one on, sends session text off your machine.
- A pull request that merges itself may not be allowed where you work. The prompt's default is no.

## Credits

The design draws on [metaharness.tools](https://metaharness.tools/) from Nx, Birgitta Böckeler's [Harness Engineering for Coding Agent Users](https://martinfowler.com/articles/harness-engineering.html), and Anthropic's [Scaling Managed Agents](https://www.anthropic.com/engineering/managed-agents). Session search is [sessions](https://github.com/nicknisi/sessions).

## License

[MIT](LICENSE). It is provided as is, with no warranty, and you use it at your own risk.

This project is not affiliated with or endorsed by Anthropic, OpenAI, GitHub, Nx or the authors of the tools it mentions. Claude and Claude Code are trademarks of Anthropic. Codex is a trademark of OpenAI.
