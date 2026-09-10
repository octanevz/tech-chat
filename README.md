# tech-chat

A chat notebook for talking through technical topics with AI coding agents (Claude Code,
Codex CLI, OpenCode) from one directory. Not a software project: no code, no build, nothing
to run.

Not meant to be used as-is. Fork it and adjust models, effort, harnesses and rules to your
taste. See [Fork and customize](#fork-and-customize).

## The problem

Coding agents are built to create and edit code. Used as a plain chat partner, they get
unpredictable. In my experience they tend to:

- answer at wildly different lengths and shapes from one session to the next
- name a library without a link one day and give a full comparison the next
- "help" by creating files, offering to save the answer, running installers, or committing,
  when all I wanted was an answer
- quietly store what we talked about as agent memory

I got tired of nudging them back every time.

Plan mode doesn't fix this either, and neither does a read-only sandbox. Both restrict what
the agent may do, and both can be made the default, but neither says anything about answer
length, links or memory. Plan mode is also designed for the step before coding: explore, then
produce a plan for me to approve. I don't want a plan. I want an answer, and a notebook that
behaves the same way for every agent I start in it.

## The solution

Every agent launched from a checkout of this repo gets the same instructions and the same
guard rails. One shared `AGENTS.md` (imported by `CLAUDE.md`) spells out how I want answers shaped and when an
agent may act. Per-agent settings back the rules that matter most (file writes, git, memory)
with mechanisms that don't depend on the agent reading the prose.

| Concern | Instructions | Supporting settings |
| --- | --- | --- |
| No file writes unless explicitly asked | `AGENTS.md` | Claude sandbox configured to deny repo writes; Codex read-only sandbox; OpenCode edits ask |
| No commits or pushes by agents | `AGENTS.md` | command deny rules in all three configs |
| No agent memory from chats | `AGENTS.md` | Claude `autoMemoryEnabled: false`; Codex `generate_memories = false` |
| Notes only on request, under `notes/` | `AGENTS.md` | write approvals where required; the note scope comes from instructions |
| Concise, cited answers with URLs for recommended tools | `AGENTS.md` | verbosity settings support brevity; citations and links come from instructions |
| Consistent model and effort defaults | settings | configured models at medium effort |

Two kinds of enforcement are at work here:

- **Hard:** sandbox restrictions block operations; deny rules reject matching commands;
  approval prompts stop the agent until I say yes.
- **Soft:** answer style and where a note ends up rely on the agent reading and following the
  instructions.

All of it relies on the agent loading the settings in the first place; see
[Before you start](#before-you-start).

## What changes in practice

The examples below are illustrative, not verbatim transcripts, and instructions aren't
guarantees: the exact shape of an answer still varies, and an agent can still ignore a rule now
and then.

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
With them, the agent should check [angular.dev](https://angular.dev), tell me standalone has been the default since
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
