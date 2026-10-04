# Remote triggers and voice

Status labels: **V** = read in the official docs on 2026-10-03. **U** = unverified. Several of these features are in preview or beta. Names and limits change. Check the docs before you set anything up.

## Ways to start Claude remotely

| Way | Setup effort | Main risk | Best use |
|---|---|---|---|
| Headless `claude -p` | Low | Without `--bare`, it runs project hooks and `.mcp.json` with no trust prompt. You own the host. | Cron jobs, scripts, your own bot |
| GitHub Action `anthropics/claude-code-action@v1` | Low to medium | Needs repo secrets. Runs on your runners. | PR review, issue to PR |
| Cloud sessions (claude.ai/code, `claude --cloud`, `--teleport`) | Low | Isolated VM. Git credentials stay outside the VM. | Long async work, checked from a phone |
| Remote Control (`claude --remote-control`) | Low | Code runs on your own machine. Outbound HTTPS only. No API keys. | Steer a local session from a phone |
| Claude Tag in Slack (beta, Team and Enterprise) | Admin needed | Shared identity. Anyone in the channel can use its credentials. | Team triage in a channel |
| Claude Code in Slack (older, per user) | Low | Claude may follow instructions found in the thread. Channels only, not DMs. | Personal Slack to PR |
| Routines (cloud, research preview) | Low | No approvals. Acts under your identity. All connectors are on by default. | Morning brief, alert triage |
| Channels (research preview): Telegram, Discord, iMessage | Medium | Chat drives your live local session. Needs a sender allowlist. | Chat bridge to your laptop |
| Agent SDK | High | You build auth and allowlists. | A custom bot for many users |
| Hooks (`Notification`, `Stop`) | Low | The shell command runs with your rights. | Push a message when Claude is done |

Sources (V): https://code.claude.com/docs/en/headless, /github-actions, /claude-code-on-the-web, /remote-control, /channels, /routines, /slack, /hooks-guide, /agent-sdk/overview, https://claude.com/docs/claude-tag/overview

Useful facts (V):

- `claude -p` with `--output-format json` returns `result`, `session_id`, and `total_cost_usd`. Resume with `--resume <session_id>`. `--bare` is the recommended mode for scripts and needs `ANTHROPIC_API_KEY`.
- `/install-github-app` sets up the GitHub Action.
- Routines have a minimum interval of 1 hour. Avoid on-the-hour times, because those runs can start late.
- Channels: Pro and Max users opt in per session with `--channels`. Events arrive only while the session is open.
- Claude Tag has no per-seat charge. It bills against an organisation usage balance. It is not available on Pro or Max.
- Remote Control works on Pro, Max, Team, and Enterprise. API keys are not supported.

## Organisations can switch features off

An administrator can disable features by policy. Examples: Remote Control, Channels, Claude Tag, and voice (voice is off with an API key, Bedrock, Vertex, or Foundry). Before you build a setup, check that your organisation allows it on your work login. A feature that works on a personal account may be blocked on a work account. If it is blocked, ask your administrator. Do not look for a workaround.

## Security cautions (V, from the docs)

- **Prompt injection.** Slack threads, error-tracker events, tickets, and PR bodies can carry hidden instructions. The Slack docs say Claude "may follow directions from other messages in the context". Use trusted channels only. Avoid piping untrusted content into Claude. (https://code.claude.com/docs/en/slack)
- **Shared credentials.** In Claude Tag, anyone in the channel can make Claude use the connected credentials. Keep the channel private. Use read-only connections and dedicated service accounts.
- **Routines act as you.** They commit, post, and file tickets under your identity, with no approvals. Limit the repos, the network, and the connectors.
- **Channels.** Allowlist only yourself. If permission relay is on, anyone on the list can approve tool use. Use `--dangerously-skip-permissions` only in a trusted VM.
- **Secrets.** Use GitHub Secrets. Cloud environment variables are visible to anyone who uses that environment.
- **`claude -p` without `--bare`** loads repo hooks and `.mcp.json` without a trust prompt. Run it on trusted repos only.
- **Routine API tokens** show once. If one leaks, anyone can start your routine.
- **Agent SDK.** Do not ask third parties to log in with a claude.ai account. Use API keys.
- **Cost.** Set a spend limit, `--max-turns`, `--max-budget-usd`, a job timeout, and concurrency limits.

## Not verified (U)

- A Jira MCP URL and a Slack MCP URL were inferred by a tool and are not in the docs. Check the vendor docs.
- Phone push through ntfy, and a relay from an error tracker to a routine, are not documented.
- The mobile Code tab got only a one-line mention in the docs.

## The Telegram bot approach

The author uses a small bot instead of a live session. It works like this.

1. Dictate a task on your phone into a Telegram chat with your own bot. Use the dictation button on the phone keyboard.
2. A small script on your laptop receives the message.
3. The script starts a headless run: `claude -p` with JSON output, a budget cap, and a short list of allowed tools.
4. Claude hands the task to a worker agent. The worker follows a skill. See [agents.md](agents.md).
5. You get a draft pull request and a reply in the chat.

Why a bot and `claude -p`: no live session has to be open, and each message starts a new run.

Rules that matter:

- The only access control is an allowlist of chat IDs. Accept messages from one chat ID only. Keep the bot token secret.
- Cap the cost for each message with a budget flag.
- Open draft pull requests only. Never push to the main branch.
- Treat the message text as untrusted input if anyone else can write to the chat.
- The laptop must be on, with the script running. A small server with the repo checked out also works.

The script, the setup steps, and a README are in https://github.com/YaroslavMatushevych/claude-telegram-bot. The author has not run the script end to end. (U)

There is also an official Telegram Channel plugin for a live session. (V, https://code.claude.com/docs/en/channels) It needs the session to stay open.

## Voice

Built-in `/voice` works only at a local terminal with a microphone. For the settings, the keybinding, and the macOS permission steps, see [commands-and-modes.md](commands-and-modes.md#voice). On a phone, use the keyboard dictation button and send the text through your chat.
