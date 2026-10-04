# Context and cost

Status labels: **V** = read in official docs or the primary source. **U** = unverified. Prices and limits were checked on **2026-10-03**. They change often. Check the pricing page before you decide anything.

## What fills the context window

The context window holds every message, file read, command output, and tool result. The terminal may show one line, but the model sees all of it. Performance drops as the window fills. Anthropic calls the effect "context rot". (V)

Sources: https://code.claude.com/docs/en/best-practices, https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents

What loads at session start (V, docs):

- System prompt.
- Auto memory index (first 200 lines or 25 KB).
- CLAUDE.md files.
- Skill descriptions (one line each). The skill body loads only when the skill runs.
- MCP tool names. Tool schemas are deferred by default (tool search). They load when Claude needs them.
- Path-scoped rules load when Claude reads a matching file.

What fills it during work: file reads (about 1 to 2.5k tokens each in the docs simulation), grep output, test output, and your own messages.

Subagents start with a fresh context. Only their final summary returns to you. (V)

Hooks add to context only through `additionalContext`. Plain stdout does not. (V)

### Example from the author's machine

This is real output from a fresh session. One command: `claude -p "/context"`.

```
Model: claude-sonnet-5-5
Tokens: 13.7k / 1m (1%)

System prompt            2.1k
System tools              427
System tools (deferred)  13.1k
Skills                    9.8k
Messages                  1.3k
Autocompact buffer         33k
```

Before the first message, 13.7k tokens were in use. The biggest part is skills: 9.8k tokens, and that is only the one-line descriptions of all installed skills. Run `/context` at the start of a session, and again after you install a plugin. You may find things you forgot.

### `/usage`

`/usage` shows where your limits go, based on local sessions. It also shows prompt cache stats (v2.1.251 or later). (V) Example from the author's machine, last 7 days:

```
Last 7d - 1463 requests - 14 sessions
  74% of your usage was at >150k context
  33% came from sessions active for 8+ hours
```

This is a bad habit, not a target. Long sessions are where context gets noisy and cost goes up.

On API billing, the dollar figure in `/usage` is a local estimate at list price. It is not a bill. (V)

## Clear, compact, or rewind

| Situation | Use | Why |
|---|---|---|
| New, unrelated task | `/clear` | Costs nothing. Starts clean. |
| Same task, long history | `/compact <focus>` | Keeps continuity. Compacting a big context is itself a large request. |
| One noisy detour | Esc Esc, then "Summarize from here" | Targeted summary. The originals stay in the transcript. |
| Wrong path taken | `/rewind` or Esc Esc | Restores code, conversation, or both. Returns to a cached prefix. |
| Quick side question | `/btw` | The answer never enters history. |
| Task spans several sittings | `--continue` or `--resume` | Keeps the session. |

(All V: https://code.claude.com/docs/en/checkpointing, https://code.claude.com/docs/en/costs)

`/rewind` does not track Bash edits or most subagent edits. It is not a replacement for git. (V)

Habits (V, from the docs):

1. `/rename`, then `/clear`, between tasks.
2. After two failed corrections, `/clear` and write a better prompt.
3. Send research and log reading to subagents.
4. Prefer CLI tools (`gh`, `aws`) to MCP servers. Switch off MCP servers you do not use.
5. Name the file and function in the prompt.
6. Include a way to verify the work, such as a test command.

## The smart zone rule (Matt Pocock)

Matt Pocock's dictionary of AI coding says the "dumb zone" on frontier models commonly begins around 125K to 150K tokens, "though this is debated". (V, https://github.com/mattpocock/dictionary-of-ai-coding/blob/main/dictionary/Smart%20zone.md; the page shows no date)

Key points from the same entry (V):

- The zones do not follow the context window limit. A 1M window does not mean 1M smart tokens.
- Do one task per session.
- If a task is bigger than one smart zone, split it. Hand off or compact at a natural boundary.

Pocock's own posts on this are on X. The author could not open them, so only search snippets exist. (U)

Boris Cherny is said to have written that auto-compact starts near 155k tokens. Only a search snippet exists. (U) Do not quote it.

Practical use: show context usage in your status line (the `/statusline` command exists, V), and clear or hand off before about 150K. Colour thresholds on that line are a community habit, not an official feature. (U)

## Prices per model

USD per million tokens. Checked 2026-10-03. Source: https://platform.claude.com/docs/en/about-claude/pricing (V)

| Model | Input | 5-min cache write | 1-hour cache write | Cache read | Output |
|---|---|---|---|---|---|
| Fable 5.1 | 10 | 12.50 | 20 | 0.25 | 50 |
| Opus 5.5 | 4 | 5 | 8 | 0.20 | 20 |
| Sonnet 5.5 | 2 | 2.50 | 4 | 0.20 | 10 |
| Haiku 4.5 | 1 | 1.25 | 2 | 0.10 | 5 |

Notes (V):

- Opus 5.5, Sonnet 5.5, and Fable 5.1 have a 1M-token window at the standard price. Haiku 4.5 has 200K.
- Batch API gives 50% off input and output. It stacks with caching.
- Models from 4.7 on use a tokenizer that makes about 30% more tokens for the same text. A price per token is not a price per task.
- Fast mode on Opus 5.5 costs $8 input and $40 output.
- Claude Code costs about $13 per developer per active day on average, and $150 to $250 per month. (docs, https://code.claude.com/docs/en/costs)
- The numeric limits for Pro and Max plans were not found on the pages the author read. Check https://claude.com/pricing. (U)

Which model for which job (docs advice plus the author's reading):

- Haiku: exploration, summaries, log triage. (V)
- Sonnet: daily scoped coding. (V)
- Opus: ambiguous bugs, architecture, long runs. (V)
- Fable: only if Opus at a higher effort fails your own evals. (V, models overview)
- A stronger model can cost less per finished task if it avoids retries. This is the author's inference, not an Anthropic claim. Measure it on your tasks.
- Lower `/effort` before you change model. An effort change keeps the cache. A model change does not. (V)

## Prompt caching rules

Sources: https://platform.claude.com/docs/en/build-with-claude/prompt-caching, https://code.claude.com/docs/en/prompt-caching (V)

- Cache write costs 1.25x base input (5-minute) or 2x (1-hour). A read costs 0.1x or less. A read refreshes the timer.
- A 5-minute cache pays off after 1 read. A 1-hour cache pays off after 2 reads.
- Order of the cache: tools, then system prompt, then messages. A change at one level invalidates everything after it. The match is an exact prefix.
- Short prompts are not cached. The minimum is 512 tokens on Opus 5.5, Sonnet 5.5, and Fable 5.x. It is 4,096 on Haiku 4.5.
- Claude Code uses a 1-hour cache for the main conversation on a subscription. It uses 5 minutes on an API key, in cloud sessions, and on usage credits. Subagents and compaction use 5 minutes by default.
- `/usage` shows a "Prompt cache (main)" line with the hit share.

Do:

- Choose model and effort at the start of a session.
- Use `/clear` between unrelated tasks and `/compact` at task boundaries.
- Keep CLAUDE.md and the tool set stable.
- Use `/rewind` instead of compaction to drop a wrong path.
- Use cheap subagents for noisy work.

Do not:

- Switch models in the middle of a task. Each model has its own cache. The next turn rereads everything at full price.
- Toggle `opusplan` back and forth. Each toggle is a model switch.
- Turn on fast mode late in a session. It costs one full uncached read.
- Add or remove MCP servers during a session when tool search is off.
- Leave a session idle past the cache time, then send a one-line question. It reprocesses the whole context.
- Upgrade Claude Code in the middle of a task. The system prompt changes.

## Worked example

This is the author's arithmetic from the table above. It is an illustration, not a measurement.

Assumptions: 40 turns. A 20k-token starting prefix. On average, 60k tokens read from cache per turn. 3k new tokens written per turn. 1k output tokens per turn. 5-minute cache, no breaks.

Totals: 2.4M tokens read from cache, 140k written to cache, 40k output.

| Case | Cache read | Cache write | Output | Total |
|---|---|---|---|---|
| Opus 5.5, caching | 2.4M x $0.20 = $0.48 | 0.14M x $5 = $0.70 | 0.04M x $20 = $0.80 | about $1.98 |
| Sonnet 5.5, caching | $0.48 | 0.14M x $2.50 = $0.35 | 0.04M x $10 = $0.40 | about $1.23 |
| Opus 5.5, no caching | none | none | $0.80 | 2.54M x $4 = $10.16, plus $0.80, about $10.96 |
| Sonnet 5.5, no caching | none | none | $0.40 | 2.54M x $2 = $5.08, plus $0.40, about $5.48 |

What to learn from it:

- Caching cuts the cost by about 80 to 90% in this case.
- The gap between Opus and Sonnet is only about 1.6x, because the cache read price is the same. New tokens and output drive the gap.
- A cache miss after an idle break costs a full reprocess: 60k tokens x $4 per million = $0.24 on Opus 5.5, plus the write.
