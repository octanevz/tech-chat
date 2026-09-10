# tech-chat

This is a chat notebook, not a software project. I keep it around so I can talk through
technical topics with AI coding agents (Claude Code, Codex CLI, OpenCode) from one directory,
where every agent starts with the same instructions and the same guard rails.

It's not meant to be used as-is. If you like the idea, fork it and adjust the models, effort,
harnesses and rules to your own taste and needs. See [Fork and customize](#fork-and-customize).

## Why it exists

Coding agents are built for creating and editing code. Used as a plain chat partner, they get a
bit unpredictable. In my experience they tend to:

- answer at wildly different lengths and shapes from one session to the next
- name a library without a link one day and give a full comparison the next
- "help" by creating files, offering to save the answer, running installers, or committing,
  when all I wanted was an answer
- quietly store what we talked about as agent memory

I got tired of nudging them back every time. One shared `AGENTS.md` (imported by `CLAUDE.md`)
plus per-agent settings spell out what I expect and block the things I never want:

| Concern | Instructions | Supporting settings |
| --- | --- | --- |
| No file writes unless explicitly asked | `AGENTS.md` | Claude sandbox configured to deny repo writes; Codex read-only sandbox; OpenCode edits ask |
| No commits or pushes by agents | `AGENTS.md` | command deny rules in all three configs |
| No agent memory from chats | `AGENTS.md` | Claude `autoMemoryEnabled: false`; Codex `generate_memories = false` |
| Notes only on request, under `notes/` | `AGENTS.md` | write approvals where required; the note scope comes from instructions |
| Concise, cited answers with URLs for recommended tools | `AGENTS.md` | verbosity settings support brevity; citations and links come from instructions |
| Consistent model and effort defaults | settings | configured models at medium effort |

Not all of these are equally hard. Sandbox restrictions and deny rules actually block
operations, and approval prompts stop the agent until I say yes. Answer style and where a note
ends up still rely on the agent reading and following the instructions, and all of it relies
on the agent loading the settings in the first place.

## What changes in practice

The examples below are illustrative, not verbatim transcripts. Instructions aren't guarantees
either: the exact shape of an answer still varies, and an agent can still ignore a rule now and
then. What they do is make it far more likely that an answer stays short, links the libraries
it recommends, and never offers to write a file. The parts that actually block things (the
sandbox, the git deny rules, memory switched off) come from the settings, not from the prose.

**"Which Python library for parsing TOML?"**

Without instructions, one session might say "use `tomllib`" and the next might write six
paragraphs on `tomli`, `tomlkit` and `toml` with no links and offer to install one. With them,
I get something like:

- Python 3.11+: `tomllib` (stdlib, read-only).
- Need to write TOML or keep comments: `tomlkit`, https://github.com/python-poetry/tomlkit,
  MIT, actively maintained.

**"How would I add a global exception handler to my ASP.NET Core API?"**

Without instructions, the agent might scaffold a project right here or go hunting for a
`Program.cs` to edit. With them, it should state which framework version it assumes (or ask,
if it matters), look at the official docs, and put an `IExceptionHandler` implementation and
the two registration lines in the chat with a link to the page it read. It should touch
nothing, because I asked a question, not for a change.

**"Do Angular components still need `standalone: true`?"**

Without instructions, the answer might come straight from training data and be years stale.
With them, the agent should check angular.dev, tell me standalone has been the default since
Angular 19, and cite the page it read.

**"Save a note on that."**

Without instructions, the agent might write the file, `git add` it, and commit. With them, it
should write `notes/angular-standalone-components.md`, ask for approval on that one file if
the tooling wants it, and stop. Notes are the one write I expect from an agent in here. Anything
else needs an explicit ask, and git stays mine to handle outside agent sessions.

## Before you start

- Install and authenticate the agents you want to use.
- Look over their configs and pick models your accounts can actually use.
- Make sure each agent loads the project instructions and settings. In Claude Code, `/memory`
  lists the loaded `CLAUDE.md` files and `/permissions` shows the effective rules. In Codex,
  `/status` shows the session's model, sandbox and approval settings, and the config and rules
  in this repo only apply once the project is trusted in `~/.codex/config.toml`. For OpenCode,
  `opencode debug config` prints the merged config.
- If you want to see the restrictions bite, do it in a throwaway copy of this repo. Ask the
  agent to create a file (should be blocked or require approval) and to run `git commit`
  (should be denied outright). An agent politely declining because `AGENTS.md` told it to
  proves nothing about the settings, so tell it to try anyway and report the tool error.

## Workflow

- Start any of the agents in this directory and ask away. Answers stay in the chat.
- Ask explicitly to save a note and it lands in `notes/`.
- Notes are scratch output, not part of this repo. Move them to the project they belong to.
  Git operations are yours to run outside agent sessions.

## Fork and customize

- Models and effort: `.claude/settings.json`, `.codex/config.toml`, `opencode.json`.
- Rules for answers, research and translations: `AGENTS.md`.
- Other harnesses: add their config, point them at `AGENTS.md`, and set up whatever the
  harness offers for write approvals, sandboxing, git deny rules, and memory.
- `notes/` is gitignored. Drop that entry if you do want notes tracked in your fork.
