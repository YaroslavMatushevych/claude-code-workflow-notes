# Three tips, tested

I ran each of these on my own machine on 2026-10-04 with Claude Code 2.1.288 and Sonnet 5.5. Each was one run, in a small throwaway folder. Results vary, so treat them as examples and not as benchmarks.

## 1. Every skill costs tokens in every session

A skill that Claude can start on its own puts its name and description into the context of every session.

Test: `claude -p "/context"` in three folders.

| Folder | Skills line in `/context` |
|---|---|
| No project skill | 9.8k tokens |
| One new skill, 50-word description | 9.9k tokens, and `release-notes` is listed at about 110 tokens |
| Same skill with `disable-model-invocation: true` | 9.8k tokens, and the skill is not listed |

The first line (9.8k) comes from the skills already installed on my machine. What to do: add `disable-model-invocation: true` to skills that you only call by hand. The docs say such a skill stays out of the context until you call it. It also cannot be preloaded into an agent with the `skills:` field.

## 2. Use Claude as a Unix command

Claude Code reads standard input.

```
git diff main...HEAD | claude -p "List the 3 biggest risks in this diff. One line each, no intro."
```

On a three-commit demo repo it printed three risks. The first one: a value with only spaces is not null, so the saved value is lost. My own demo did not test that case. The model did not keep to "one line each" for the third risk, so check the format if a script parses the output.

Add `--output-format json` to get the result with token counts and cost. Add `--max-budget-usd` to cap one run.

## 3. Audit your instruction files

```
claude -p "/doctor prompt-audit"
```

I used a toy `CLAUDE.md` with two planted problems: one line says to use tabs and another says to use 2-space indentation, and the last line is all-caps `IMPORTANT`, `ALWAYS`, `NEVER`. The audit found both:

- "Lines 2 and 3 give opposite rules. One says tabs, the other says 2-space indentation. The model cannot obey both."
- "Line 7 is pressure language. It is all-caps IMPORTANT, ALWAYS, NEVER, and it repeats the other rules."

It also said that without git history it cannot tell which of two conflicting lines is newer, so you must decide. The audit also reads the skills in your home folder and counts plugin files. Check the report before you share it, because it can list names from your own setup. One audit run used about 4,000 output tokens (about $0.42 on Sonnet in my run).

## 4. When Claude ignores a rule, make it a hook

A rule in `CLAUDE.md` is a request. A hook is code, so it enforces.

This `PreToolUse` hook for Bash blocks `git commit --no-verify`. Exit code 2 blocks the command and sends the message on stderr back to Claude.

```js
// .claude/hooks/block-no-verify.mjs
let input = "";
process.stdin.on("data", (d) => (input += d)).on("end", () => {
  const command = JSON.parse(input).tool_input?.command ?? "";
  if (/git\s+commit\b.*--no-verify/.test(command)) {
    console.error("Blocked: do not skip git hooks. Fix the failing check instead.");
    process.exit(2);
  }
});
```

```json
{
  "hooks": {
    "PreToolUse": [
      { "matcher": "Bash", "hooks": [{ "type": "command", "command": "node .claude/hooks/block-no-verify.mjs" }] }
    ]
  }
}
```

Test: `echo '{"tool_input":{"command":"git commit --no-verify -m x"}}' | node .claude/hooks/block-no-verify.mjs; echo $?` prints the message and exit code 2.

Real run: I asked `claude -p "Run this exact command and tell me what happened: git commit --allow-empty --no-verify -m test"` with the hook in project settings. The command did not run. Claude reported that the hook blocked it, did not try to get around it, and no commit was created. One run.

The idea comes from the Everything Claude Code repository (github.com/affaan-m/ECC), which has hooks that block the same flag and block edits to lint configs. Notes on that repository, checked on 2026-10-04:

- It is large: about 293 skills, 68 agents, 94 commands and 54 hook scripts, and it has a large star count. It also promotes paid plans.
- Its savings claims (for example "about 60% cost") have no benchmark in the repository.
- The 293 skill descriptions alone are about 22k tokens by estimate, which works against its own advice on context budget.
- About 24 hook entries run on nearly every tool call. One observation hook logs all prompts and tool use to a folder outside `~/.claude`. Two hooks start `claude -p` on their own and spend tokens.
- Read the hooks before you install it. Borrow single ideas, as in the example above, and do not install the whole set.
