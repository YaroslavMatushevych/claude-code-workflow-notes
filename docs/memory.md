# Memory and CLAUDE.md

Status labels: **V** = read in official docs or the primary source (2026-10-03). **U** = unverified.

## Two systems

Claude starts every session with an empty context. Two systems carry knowledge into it. (V)

| | CLAUDE.md | Auto memory |
|---|---|---|
| Who writes it | You | Claude |
| What it holds | Instructions and rules | Claude's notes: your corrections, preferences, decisions |
| Where it lives | In the repo or in `~/.claude/` | `~/.claude/projects/<project>/memory/` |
| When it loads | Session start | Session start (only the index) |

Both are context. Neither is enforcement. Claude reads them as a user message after the system prompt. The docs say there is no guarantee of strict compliance. (V)

Source: https://code.claude.com/docs/en/memory

## CLAUDE.md locations (V)

- Managed policy file: set by an administrator. You cannot exclude it.
- User: `~/.claude/CLAUDE.md`
- Project: `./CLAUDE.md` or `./.claude/CLAUDE.md` (share it through git)
- Local: `./CLAUDE.local.md` (add it to `.gitignore`)
- Files in the working directory and its parents load at launch. Files in subdirectories load when Claude reads files there.
- `@path/to/file` imports load at launch too. They do not save context.
- `.claude/rules/*.md` files with a `paths:` glob load only when Claude touches a matching file.
- If a folder has no CLAUDE.md, Claude can use `AGENTS.md`. This needs v2.1.277 or later.

## The load limit for MEMORY.md (V)

Auto memory has an index file, `MEMORY.md`. At session start Claude loads only the first 200 lines or the first 25 KB, whichever comes first. The rest is dropped. Topic files are not loaded at start. Claude reads them when it needs them.

So keep `MEMORY.md` short. One line per memory. Put the detail in topic files. A line past the limit does not exist for Claude.

Example index (illustrative, in the same format Claude writes):

```markdown
# Memory Index

## Feedback
- [Message drafting tone](./feedback_message_drafting_tone.md) - short, direct, no hedging
- [Minimize clarifying questions](./feedback_minimize_questions.md) - once autonomy is given, decide and go

## Projects
- [Talk slides app](./project_talk_slides.md) - Next.js app, runs on port 3000
```

Each line is a title, a link, and a one-line hook. The hook tells Claude when to open the file.

## What goes where (V)

| Need | Put it in |
|---|---|
| Rules for every session, shared with the team | CLAUDE.md |
| Personal preferences | `~/.claude/CLAUDE.md` or `CLAUDE.local.md` |
| Rules for one area of the code | `.claude/rules/` with `paths:` |
| A procedure you need only sometimes | A skill |
| Something that must happen every time | A hook or `permissions.deny` |
| Claude's own notes on you and the project | Auto memory |

## How to write good lines

The test for each line, from the official docs: "Would removing this cause Claude to make mistakes?" If not, cut it.

Write lines that are concrete and checkable. (V, docs)

| Bad | Good |
|---|---|
| Format code properly. | Use 2-space indentation. |
| Keep files organized. | API handlers live in `src/api/handlers/`. |
| Test your changes. | Run `npm test` before committing. |
| Never use default exports. | Use named exports. Default exports break our codemods. |

More rules:

- Give a reason or an alternative. A rule that says only "never do X" can leave the agent stuck. (Shrivu Shankar, V: https://blog.sshh.io/p/how-i-use-every-claude-code-feature)
- Point to a document and say when to read it. Do not embed it with `@`, because imports load at launch. (same source, V)
- Do not use a prompt for a linter's job. (HumanLayer, V: https://www.humanlayer.dev/blog/writing-a-good-claude-md)
- Use `IMPORTANT` on one stubborn line at most. Many emphasised lines cancel each other.
- Add a line when Claude makes the same mistake twice, or when a review finds a repo-specific miss. (docs, V)
- Add a compaction hint: "When compacting, keep the list of modified files and the test commands." (docs, V)

Do not put these in CLAUDE.md (docs list, V): things Claude can learn from the code, standard language conventions, long API docs (link them), fast-changing facts, tutorials, file-by-file descriptions, and "write clean code" advice.

A short template (V):

```markdown
# Project: <name>
<one line: what it is and the stack>

## Commands
- Dev: `pnpm dev`  Test one file: `pnpm test <path>`  Typecheck: `pnpm typecheck`
- Run typecheck and related tests before saying work is done.

## Conventions
- ES modules, named exports, 2-space indent.

## Gotchas
- <a non-obvious thing that caused a bug>
- IMPORTANT: never edit `src/generated/`.

## Compaction
- When compacting, keep modified files and test commands.
```

## Pitfalls

### 1. Instruction bloat

More lines mean less compliance with each line. HumanLayer: "As instruction count increases, instruction-following quality decreases uniformly." (V, https://www.humanlayer.dev/blog/writing-a-good-claude-md, 2025-11-25). They keep the root file under 60 lines. The Anthropic target is under 200 lines. (V)

The cost is real too. Shrivu Shankar reports a 13 KB monorepo CLAUDE.md that used about 20,000 tokens in a fresh session. (V, 2025-11-02, link above)

### 2. Contradictions

The docs say that if two instructions contradict each other, Claude may pick one arbitrarily. (V) Run `/doctor prompt-audit` to audit your instruction files. In a monorepo, use `claudeMdExcludes`. (V)

### 3. Claude ignores a rule

CLAUDE.md is a request, not a lock. Two GitHub issues describe a large memory index that Claude did not check (#62812, #48783). (V) If a rule must hold every time, use a `PreToolUse` hook or `permissions.deny`.

### 4. Lost after compaction

Issues #4017 and #11545 report rules ignored after `/compact`. (V, issues read) The docs now say the project-root CLAUDE.md is re-read from disk after `/compact`. (V) What can still disappear: instructions you gave only in chat, nested CLAUDE.md files that were not yet reloaded, and path rules that do not match. Put important rules in the file, not in the chat.

### 5. Stale memory

Auto memory grows. Old decisions become wrong. One post says memory without criteria "ends up as useless as a 300-line CLAUDE.md". (V, dev.to, https://dev.to/ohugonnot/persistent-memory-in-claude-code-whats-worth-keeping-54ck, Odilon Hugonnot, 2026-04-05; check the exact wording at the source.) A vendor article reports contradictory entries after 20 to 30 sessions. (U, vendor source)

What to do: open `/memory` now and then. Delete old entries. Keep the index under 200 lines.

### 6. Memory poisoning

Memory files are text that Claude trusts. Cisco showed an npm `postinstall` script that rewrote `MEMORY.md` and settings. (V, 2026-04-01, https://blogs.cisco.com/ai/identifying-and-remediating-a-persistent-memory-compromise-in-claude-code) Treat memory as untrusted input after you install unknown packages. Do not store secrets in memory. (This is advice, not a finding from a source.)

### 7. The `#` shortcut

Old posts say you can type `#` to add a line to memory. The current memory docs do not mention it. (U) Ask Claude "add X to CLAUDE.md" or edit the file through `/memory`.

### 8. Claims about how others use CLAUDE.md

Posts say the Claude Code team shares one CLAUDE.md and edits it several times a week. We found this only in second-hand summaries. (U)

## Sources

See [sources.md](sources.md) for the full list with dates.
