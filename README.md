# Claude Code workflow notes

Practical notes on Claude Code: memory, context and cost, commands and modes, skills, agents, and remote chat. With sources.

These notes are the written version of the talk "Inside My Real Claude Code Workflow" (CityJS Athens 2026). The slides show the flow. These pages hold the detail, the numbers, and the links.

## Who this is for

Developers who already use Claude Code every day. You want to spend less, get more reliable results, and know which claims have a source.

## How to read the status labels

Each page marks its claims.

- **V** (verified): the author read this in the official docs or in the primary source, on the date shown.
- **U** (unverified): a search snippet, a second-hand report, a vendor claim, or something the author did not run.

Claude Code changes fast. Version numbers and prices in these notes were checked on 2026-10-03. Check again before you rely on them.

## Contents

| Page | What it covers |
|---|---|
| [docs/memory.md](docs/memory.md) | CLAUDE.md and auto memory, the 200-line load limit, how to write good lines, and common pitfalls. |
| [docs/context-and-cost.md](docs/context-and-cost.md) | What fills the context window, `/context` and `/usage`, clear vs compact vs rewind, the "smart zone" rule, model prices, prompt caching, and a worked cost example. |
| [docs/commands-and-modes.md](docs/commands-and-modes.md) | Permission modes, output styles, effort, fast mode, `/focus`, commands people overlook, and voice dictation. |
| [docs/skills.md](docs/skills.md) | How to write skills, with advice from Anthropic, Jesse Vincent, Matt Pocock, and a seven-level talk on skills. |
| [docs/agents.md](docs/agents.md) | Workflow or agent, what agents cost, what goes wrong in parallel, how Claude sees a tool, and a worker agent that preloads a skill. |
| [docs/remote-and-voice.md](docs/remote-and-voice.md) | Ways to start Claude from your phone or a chat, their risks, and the Telegram bot approach. |
| [docs/sources.md](docs/sources.md) | Every source URL with author, date, and V or U status. |

## Related repositories

- [claude-code-agent-skills](https://github.com/YaroslavMatushevych/claude-code-agent-skills): the skills, agents, hooks, and CI workflow from the talk.
- [claude-telegram-bot](https://github.com/YaroslavMatushevych/claude-telegram-bot): a small script that turns a chat message into a Claude Code run.
- [claude-code-workflow-talk](https://github.com/YaroslavMatushevych/claude-code-workflow-talk): the slides.

These repositories were created at the same time as this one. A link can fail for a few minutes after publication.

## Limits of these notes

- Many examples are drafts. The author did not run every example from end to end. Each page says which.
- Some sources could not be read directly (for example posts on X, and some Reddit threads). Claims that depend on them are marked U or left out.
- Output examples labelled "example from the author's machine" are real, but they show one setup on one day.

## License

MIT. See [LICENSE](LICENSE).
