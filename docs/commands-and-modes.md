# Commands and modes

Status labels: **V** = read in official docs on 2026-10-03. **U** = unverified. Version numbers are as the docs stated them. Check `claude --version` and the docs for your own version.

Sources: https://code.claude.com/docs/en/permission-modes, /output-styles, /model-config, /fast-mode, /fullscreen, /commands, /interactive-mode, /settings-reference, /voice-dictation

## Permission modes (V)

Press Shift+Tab to cycle. A mode change keeps the cache.

| Mode | What it does |
|---|---|
| `default` (Manual) | Reads files without asking. Asks for everything else. |
| `acceptEdits` | Reads, edits files, and runs common filesystem commands. |
| `plan` | Read-only until you approve a plan. |
| `auto` | Runs everything. A classifier model reviews each action. It is the start mode in the terminal and VS Code since v2.1.283. |
| `dontAsk` | Launch flag only. Denies anything not pre-approved. Good for CI. Not in the cycle. |
| `bypassPermissions` | Launch flag or setting. No checks. Use only in a container. |

The extra cost of the `auto` classifier is not stated in the docs. (U)

Start in a mode with `claude --permission-mode plan`. Enter plan mode in a session with `/plan <task>`.

## Output styles (V)

An output style changes how Claude writes its answers. Built-in styles: Default, Proactive, Concise, Explanatory, Learning. Concise needs v2.1.237 or later.

- **Concise**: the result goes in the first sentence. No preamble. No step narration. No closing recap. Simple answers take 1 to 3 sentences. Full detail stays for errors, failing tests, security warnings, and destructive actions. It does not reduce engineering effort.
- **Explanatory** and **Learning** add "Insight" blocks. They use more output tokens.
- The docs publish no exact token saving for Concise. (U)

Select a style with `/output-style concise`, with `/config`, or with `"outputStyle": "Concise"` in settings. The value in the file is case-sensitive. A wrong case gives Default.

A switch applies from your next message. It keeps the cache. A style applies to the main conversation, not to other subagents. A style is an instruction, not enforcement.

### A custom style: Terse

Save this as `~/.claude/output-styles/terse.md` or `.claude/output-styles/terse.md`. Restart Claude Code after you edit it. Then run `/output-style terse`.

```markdown
---
name: Terse
description: Shortest useful answers; full detail only for risk or on request
keep-coding-instructions: true
---

Answer first, in the first sentence. No greeting, no restating the question,
no narration of steps, no closing summary.

- Simple questions: one to three sentences.
- After code changes: list changed files and one line on what changed. Do not repeat the diff.
- Use short plain words and short sentences. Prefer bullets over paragraphs. No emoji.
- Ask a question only when blocked. Otherwise pick the sensible option and state it in one line.

Write full detail when:
- The user asks for explanation, reasoning, or more detail.
- Reporting errors, failing tests, or security risk.
- Confirming destructive actions.

Keep engineering quality unchanged: verify work, read code before editing, run tests.
```

Front-matter fields (V): `name`, `description`, `keep-coding-instructions` (default false), `force-for-plugin`. Set `keep-coding-instructions: true`. If you do not, a custom style drops the built-in coding instructions. The built-in Concise style already does most of what Terse does. Terse adds your own rules, such as how to report diffs. The file is a draft. The author did not measure its token effect. (U)

## Effort (V)

Levels: low, medium, high, xhigh, max. Set it with `/effort`, `--effort`, `CLAUDE_CODE_EFFORT_LEVEL`, or the `effortLevel` setting.

- Opus 5.5 and Sonnet 5.5 default to medium.
- You cannot switch thinking off on Opus 5.5, Sonnet 5.5, or Fable. Collapsed thinking is still billed as output.
- Higher levels use more thinking tokens.
- An effort change keeps the cache. A model change does not.

## Fast mode (V)

`/fast` or Option+O. Up to 2.5x faster output at the same Opus quality. Available on some Opus models only. On Opus 5.5 it costs $8 input and $40 output per million tokens. Turning it on in the middle of a conversation costs one full uncached input read, so turn it on at the start.

## Display and focus (V)

- `/focus`: shows the last prompt, a one-line tool summary, and the final response. It persists. It changes the display only.
- Ctrl+O: transcript view with full tool output.
- Option+T: toggle thinking display.
- `/recap`: one-line summary. It runs by itself after 3 or more minutes away. Set `awaySummaryEnabled: false` to stop it.
- `/tui fullscreen`: alternate-screen renderer.
- `/vim` was removed in v2.1.92. Use `/config` and set the editor mode.
- `/config key=value` sets a value directly, for example `/config model=sonnet`.

The author did not read the full `/config` menu, the changelog, or the status line page. (U)

## Overlooked commands

All of these are in the command docs. (V)

| Command | Use |
|---|---|
| `/btw <question>` | Side question. No tools. Never enters history. |
| `/rewind` (also `/undo`), or Esc Esc | Restore code, conversation, or both. |
| `/fork`, `/branch` | Try another direction. Return with `/resume`. |
| `/goal <condition>` | Claude keeps working until the condition is met. |
| `/batch <instruction>` | Splits a large change into parallel background subagents, after you approve the plan. |
| `/code-review high --fix` | Reviews the diff and applies the findings. `/review` is an alias. |
| `/simplify` | Reviews changed code for reuse and simplification, and applies fixes. |
| `/loop [interval] [prompt]` | Repeats a prompt. Without an interval, Claude sets its own pace. |
| `/schedule` | Creates a cloud routine. |
| `/insights` | HTML report on your own recent sessions. |
| `/usage` (alias `/cost`) | Tokens, cache stats, and what used your limits. |
| `/doctor` | Checks your setup. `/doctor prompt-audit` audits instruction files. |
| `claude -w <name>` | New session in a git worktree. |
| `claude -p` | Headless mode. Use `--output-format json`, `--max-turns`, `--max-budget-usd`. |

Keys: Ctrl+G opens the prompt in your editor. Ctrl+S stashes a draft. Ctrl+R searches history. Ctrl+B backgrounds a running command. Option+P switches model and keeps your prompt.

Things that do not exist or changed (V): `/vibe` does not exist. `/vim` was removed. `/agents` only prints a reminder on v2.1.198 and later. `/hooks` is view-only.

## Voice

Built-in dictation. Source: https://code.claude.com/docs/en/voice-dictation (V)

- Turn it on with `/voice [hold|tap|off]`.
- Hold mode (default): hold Space, speak, release. There is a short warmup.
- Tap mode: `/voice tap`. Tap Space on an empty prompt, speak, tap again. It sends by itself at 3 or more words. Recording stops after 15 seconds of silence or 2 minutes in total.
- Settings: `{"voice":{"enabled":true,"mode":"tap","autoSubmit":true}}`
- It needs a claude.ai login and a local microphone. It does not work with an API key, Bedrock, Vertex, or Foundry. It does not work over SSH or in cloud sessions.
- Audio goes to Anthropic servers for transcription.
- Transcription does not use your tokens or usage limits.
- It supports 20 languages. Set `language` in `/config`.
- The docs say it is tuned for coding words. The project name and git branch are sent as hints.

### Remove the warmup: bind a modifier key

Add this to `~/.claude/keybindings.json`:

```json
{ "bindings": [ { "context": "Chat", "bindings": { "meta+k": "voice:pushToTalk" } } ] }
```

### macOS microphone permission

Give the microphone permission to your terminal in System Settings, Privacy, Microphone. If the terminal is missing from the list, reset and relaunch:

```bash
tccutil reset Microphone com.apple.Terminal
# iTerm2: com.googlecode.iterm2
```

### Not verified

Wispr Flow, SuperWhisper, and macOS Dictation as alternatives were not tested. (U) Vendor pages claim speed gains such as "3x faster than typing". Those are marketing claims. (U) System-wide dictation tools help if you dictate outside Claude Code or use an API key.

For voice from a phone, see [remote-and-voice.md](remote-and-voice.md).
