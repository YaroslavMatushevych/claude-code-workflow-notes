# Agents

Status labels: **V** = read at the source on 2026-10-03. **U** = unverified. Quotes from the Anthropic engineering posts came through a summarising fetch tool. Check them at the link before you reuse them.

## Workflow or agent?

Anthropic uses two words. (V, https://www.anthropic.com/engineering/building-effective-agents)

- A **workflow** uses an LLM and tools on paths that your code defines.
- An **agent** lets the LLM choose its own steps and tools.

Build an agent only when both are true: you cannot set the steps in advance, and you can check whether the result is good (tests, clear criteria). One LLM call with retrieval is often enough. Start simple. Add complexity only when it measurably helps.

Try a workflow pattern first: prompt chaining, routing, parallelisation, orchestrator-workers, or evaluator-optimizer.

## What agents cost

Anthropic reports that agents use about 4x the tokens of a chat. Multi-agent systems use about 15x. (V, https://www.anthropic.com/engineering/multi-agent-research-system, 2025-06-13) In the same post, token use alone explained 80% of the variance on one benchmark (BrowseComp). Multi-agent work was 90.2% better than a single agent on Anthropic's internal research eval. It fits breadth-first, parallel work. It fits most coding work poorly, because coding tasks share state.

Agent teams in Claude Code use about 7x the tokens of a standard session when teammates run in plan mode. Idle teammates keep using tokens. Use Sonnet for teammates. Keep teams small. Shut teammates down when they finish. (V, https://code.claude.com/docs/en/costs)

The average Claude Code cost is about $13 per developer per active day. (V, same page) Subagent requests come on top of the main conversation.

## What goes wrong in parallel

| Problem | Evidence |
|---|---|
| Review is the bottleneck | Simon Willison: "the natural bottleneck on all of this is how fast I can review the results." (V, https://simonwillison.net/2025/Oct/5/parallel-coding-agents/, 2025-10-05) |
| Code generation outruns review | Armin Ronacher: "if input grows faster than throughput, you have an accumulating failure." (V, https://lucumr.pocoo.org/2026/2/13/the-final-bottleneck/, 2026-02-13). PR queues grow, PRs go stale, and people lose knowledge of their own code. |
| Overwrites | The Claude Code docs: "Two teammates editing the same file leads to overwrites." (V, https://code.claude.com/docs/en/agent-teams) |
| Wasted effort when unattended | The same page: "Letting a team run unattended for too long increases the risk of wasted effort." |
| Idle token burn | Idle teammates keep using tokens. (V, costs page above) |
| Too many agents for a simple job | Anthropic's research system spawned too many subagents for simple queries, searched without end, and duplicated work after vague task descriptions. (V, multi-agent post) |
| Agent teams are experimental | The docs list: task status can lag, the lead may start implementing itself or declare done early, `/resume` does not restore teammates, and plan approval is granted by the lead. (V, agent-teams page) |
| Single investigator anchors | The docs say one investigator tends to anchor on the first plausible theory. They suggest teammates that argue against each other's theories. (V) |

Advice from the sources:

- Start with 3 to 5 teammates. "Three focused teammates often outperform five scattered ones." (V, agent-teams page)
- Work in separate git worktrees (`claude -w <name>`). Merge one branch at a time. (the worktree advice comes from third-party guides, U)
- Code that began with your own specification is easier to review. (Willison, V)
- Write precise task descriptions for each worker. Vague delegation causes duplicate work.

Claims we could not confirm (U): Steve Yegge's "Gas Town" runs 20 to 30 instances and spends much on a merge queue (the page returned an error, only snippets exist). The "Ralph loop" by Geoffrey Huntley, a bash loop with a plan file on disk, was also seen only in snippets.

## A pre-build checklist (V, from the Anthropic posts)

1. Workflow or agent? See above.
2. One call with retrieval may be enough.
3. Does the task value justify 4x to 15x tokens?
4. Tools are the interface. Few, well-named, high-signal output, errors that say what to try next. (https://www.anthropic.com/engineering/writing-tools-for-agents)
5. Evaluate with about 20 realistic cases. Read the transcripts.
6. Guardrails: turn cap, tool allowlist, read-only by default, budget cap. Treat tool results as untrusted input.

## How Claude sees a tool

Claude sees only three things: the tool **name**, the **description**, and the **input schema** (JSON Schema). It picks a tool from that text alone. The API docs call the description "by far the most important factor". (V, https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools)

A good description says:

- what the tool does,
- when to use it and when not to,
- what each parameter means,
- what the tool does not return.

A bad description can cause a wrong tool call, a missing or invalid parameter, retries, or no call at all. (V for the effects. The exact behaviour depends on the model.)

### Bad and good (adapted from the official docs example)

Bad:

```ts
{
  name: "get_stock_price",
  description: "Gets the stock price for a ticker.",
  input_schema: {
    type: "object",
    properties: { ticker: { type: "string" } },
    required: ["ticker"],
  },
}
```

Good:

```ts
{
  name: "get_stock_price",
  description:
    "Retrieves the current stock price for a given ticker symbol. " +
    "The ticker must be a valid symbol for a publicly traded company on a " +
    "major US exchange like NYSE or NASDAQ. Returns the latest trade price " +
    "in USD. Use when the user asks about the current or most recent price " +
    "of a specific stock. It will not provide any other information about " +
    "the stock or company.",
  input_schema: {
    type: "object",
    properties: {
      ticker: { type: "string", description: "The stock ticker symbol, e.g. AAPL for Apple Inc." },
    },
    required: ["ticker"],
  },
}
```

A test you can run (not from the docs, and the result depends on the model, so run it several times): ask "What is Apple trading at?" and "How is Tesla doing this year?" against both versions. With the bad version Claude may pass `"Apple"` instead of `"AAPL"`, or call the tool for a question about history that the tool cannot answer. (U, the author has not recorded a result)

Other facts (V): tool names must match `^[a-zA-Z0-9_-]{1,128}$`. Return errors with `is_error: true` and say what to try next. On the newest models, `tool_choice` of `"any"` or `"tool"` returns a 400 error. Use `"auto"`.

The author drafted a small agent loop with the TypeScript SDK (a turn cap of 10, one tool, `tool_use` and `tool_result` blocks). The author did not run it end to end. (U) Check the current SDK types before you copy such code.

## A worker agent that preloads a skill

An agent is a role with limited tools. A skill is the procedure it follows. You can run a planner, a worker, and a reviewer, each with its own skills. This example shows one worker.

`.claude/agents/worker.md`:

```markdown
---
name: worker
description: Implements one task end to end and opens a
  draft pull request. Use for any coding task sent
  from chat.
tools: Read, Edit, Write, Grep, Glob, Bash
model: sonnet
isolation: worktree
maxTurns: 30
skills:
  - ticket-to-pr
---
Follow the preloaded ticket-to-pr skill.
Report the pull request link and a 3-line summary.
```

What each field does (V, https://code.claude.com/docs/en/sub-agents):

- `tools`: an allowlist. If you leave it out, the agent inherits all tools.
- `model`: `sonnet`, `opus`, `haiku`, `fable`, `inherit`, or a full model ID. The order is: per-call parameter, then front matter, then `CLAUDE_CODE_SUBAGENT_MODEL`, then the main model.
- `isolation: worktree`: the agent works in its own git worktree.
- `maxTurns`: a turn cap.
- `skills`: a list of skills. The docs say the full skill content is injected into the subagent's context at startup, so the agent does not have to find the skill. **Verified in the docs.**

The skill, `.claude/skills/ticket-to-pr/SKILL.md`:

```markdown
---
name: ticket-to-pr
description: Turn a ticket description into a draft pull
  request with tests. Use only when asked to implement
  a ticket or task.
argument-hint: [ticket-id] [ticket text or path]
---
1. Restate the goal and acceptance criteria in 5 lines.
2. Write a failing test first. Run it.
3. Implement the smallest change that passes.
4. Commit on claude/<ticket-id>-<slug>. Never push to main.
5. Open a DRAFT pull request.
6. Stop if tests fail twice.
```

Notes:

- The skill is thin. Each step is checkable. It ends with a stop rule.
- The description says when to use the skill. It does not summarise the steps.
- The author removed `disable-model-invocation` from this skill, because it is unclear whether a preloaded skill can have it. Test on your version. (U)
- Neither file was run end to end. (U)
- A subagent starts with a fresh context. It gets its own prompt and the preloaded skill, not the parent history. Only its final message returns to the parent. (V)
- The description of an agent decides when Claude delegates to it. A vague description means no delegation. (V)

The same files are in https://github.com/YaroslavMatushevych/claude-code-agent-skills.
