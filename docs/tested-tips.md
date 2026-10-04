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
