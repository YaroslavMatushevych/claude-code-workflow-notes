# How to write skills

Status labels: **V** = the author read the source on 2026-10-03. **U** = unverified. Some pages were read through a fetch tool that summarises text. Quotes from those pages may differ a little from the original. Check a quote at its link before you reuse it.

This page links to other people's work. It does not copy their files. Each short quote has a link.

## The basics (V)

A skill is a folder with a `SKILL.md` file. The file has YAML front matter (`name`, `description`) and a body of instructions. Source: https://github.com/anthropics/skills

Skills load in three levels (V, https://code.claude.com/docs/en/skills):

| Level | What loads | Cost |
|---|---|---|
| 1. Name and description | Every request | Small, always paid |
| 2. SKILL.md body | When the skill triggers, or you type `/name` | Stays in context |
| 3. Bundled files and scripts | Only when read or run | Zero until used |

Which tool for which job (V):

| Need | Use |
|---|---|
| Always-on rules, under 200 lines | CLAUDE.md or `.claude/rules` |
| A procedure you need sometimes | A skill |
| Must run every time | A hook |
| Noisy side task | A subagent, or a skill with `context: fork` |
| Access to an outside system | MCP. A skill can teach how to use it. |
| Share across repos | A plugin |

A rule of thumb: make a skill the third time you paste the same playbook.

Other facts (V): keep `SKILL.md` under 500 lines. Link files one level deep. Add `disable-model-invocation: true` to any skill with side effects, such as deploy or push. Broken YAML silently disables triggering. Run `/doctor` and `claude plugin validate` to check.

## Advice from Anthropic

Source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices (V, read through a summarising fetch tool)

- Assume Claude is smart. Add only what it does not know. The page says "The context window is a public good."
- Write the description in the third person. Say what the skill does and when to use it. Bad: "Helps with documents."
- Limits: name up to 64 characters, description up to 1,024 characters.
- Keep references one level deep from `SKILL.md`.
- Match freedom to fragility. Give exact commands for fragile steps. Give room to adapt for judgment work.
- Put deterministic steps in scripts. Say whether Claude should run the script or read it.
- Write evaluations before you write long documentation. Run the task without the skill first. Test on Haiku, Sonnet, and Opus.
- Give one default with an escape hatch, not a menu of options.

Anthropic's own `skill-creator` skill (V, https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md) adds:

- Claude tends to under-trigger skills, so make descriptions a little "pushy". Put all "when to use" text in the description.
- Explain why a rule matters. Do not shout. All-caps ALWAYS and NEVER is a yellow flag.
- Read transcripts, not only final outputs.
- Do not overfit to your test examples.

The `docx` skill in the same repo is a good model. Its description lists triggers and also an exclusion ("Do NOT use for ..."). Its body is a table plus a list of footguns. (V)

## Jesse Vincent: the description says WHEN, not HOW

Source: https://github.com/obra/superpowers/blob/main/skills/writing-skills/SKILL.md (V, raw file)

- "Writing skills IS Test-Driven Development applied to process documentation."
- A description should say when to use the skill. It should not summarise the workflow. If it does, Claude may follow the description and skip the body.
- His evidence: a description that said "code review between tasks" caused one review. The skill body specified two.
- Bad: "Use when executing plans - dispatches subagent per task with code review between tasks."
- Good: "Use when executing implementation plans with independent tasks in the current session."
- Run a baseline without the skill. Write down how the agent talks itself out of the rules. Add those cases to the skill.
- Test under pressure that combines time, sunk cost, and tiredness. Repeat each wording 5 or more times.
- Do not write a skill for one-off fixes, standard practice, project conventions (use CLAUDE.md), or anything a validator can enforce.

Background: https://blog.fsck.com/2025/10/16/skills-for-claude/ (2025-10-16, V)

## Matt Pocock: small, composable, checkable

Repository: https://github.com/mattpocock/skills (V, head commit 2026-09-29). The repo changes often. Names below may move.

His README says other frameworks "take away your control". He prefers skills that are "small, easy to adapt, and composable". (https://github.com/mattpocock/skills/blob/main/README.md)

- **grill-me**: a user-invoked skill that interviews you until you share an understanding of the plan. It asks the whole set of open questions in one round and gives a recommended answer for each. It is done only when no open questions remain. https://github.com/mattpocock/skills/blob/main/skills/productivity/grill-me/SKILL.md
- **Completion criteria**: every step ends on a check. His `writing-for-agents` skill says a vague bound invites "premature completion". The best criteria are checkable and exhaustive. https://github.com/mattpocock/skills/blob/main/skills/productivity/writing-for-agents/SKILL.md
- **Prompt the positive**: his skill says that steering by prohibition drags the banned behaviour into context. Say what to do.
- **Remove no-ops**: ask "does this sentence change the behaviour versus the default?" "Be thorough" is a no-op.
- **The pointer wording decides when Claude reads a file**: a must-have file behind a weak pointer is a variance bug.
- **User-invoked or model-invoked**: a model-invoked skill costs its description on every turn. A user-invoked skill costs you the effort to remember it. Pick model invocation only when the agent must reach the skill by itself.
- **Tickets sized to one fresh context window**: see his `to-tickets` skill.

(U) His posts on X about sizing tickets by the smart zone were seen only as search snippets.

## BeerCode talk: seven levels of a skill

Talk: "Як правильно працювати з Agent Skills" (How to work correctly with Agent Skills). URL: https://www.youtube.com/watch?v=3t7VVZp2si8. BeerCode channel, speaker Kyrylo (Head of Engineering). Published 2026-08-09, about 19 minutes. Language: Ukrainian. Search the title on YouTube to find it. The author did not record the video URL.

How the author read it: from the **auto-generated captions**, using the auto-translated English track. Wording may be slightly off. The author could not see the screen. Anything about on-screen content is U. The author did not check the "SkillsBench" numbers from the talk, so they are left out. All claims below are the speaker's.

| Level | What it means |
|---|---|
| 1. Bare SKILL.md | It works but is unstable. Different runs can give different results. Adding words like "analyse carefully" changes nothing. The text grows and the behaviour stays the same. |
| 2. Fixed process | Replace "figure it out" with steps. Add a hard gate: "no confirmed cause, no fix". Add a failing test that must fail before the fix. The speaker warns that rules in a file do not guarantee that the agent follows them. |
| 3. Description as a router | The speaker calls the description the most important line. Without a trigger the skill never runs. Write three parts: which tasks trigger it, the signals, and where it must not interfere. Watch for over-triggering. |
| 4. Split into files | Every link in the body needs a condition, for example "If there are network issues, read the log conventions". Move routine work into scripts. Do not split a skill you can read in a minute. |
| 5. Limits in the environment | A prompt rule such as "do not touch the code" is a request, not a prohibition. Enforce limits with allowed tools, a model per skill, and subagents. |
| 6. Evals | A case has a realistic prompt, a flag for "should trigger", and a list of checks. Use 10 to 20 cases, about 5 that should trigger and 5 that should not. Run each case in a clean directory. Run each case 3 to 5 times. Run deterministic checks first. Then use a separate judge agent with a rubric. |
| 7. Plugin | Package the skill with its evals. Run the evals on every change. A marketplace is a git repo with a manifest. |

Points the speaker made that fit the other sources:

- Description as a trigger agrees with Anthropic and Vincent.
- Checkable steps agree with Pocock.
- An agent that grades its own work will almost always pass itself. Use a separate judge. (Paraphrase of the speaker.)

Where the talk differs (author's reading, U): the speaker does not mention Vincent's finding that a workflow summary in the description can make Claude skip the body. He answers "do not touch the code" with environment limits, not with a positive rewording.

Tools the speaker named for evals: Promptfoo, Inspect AI, DeepEval, Braintrust. The speaker's advice: start with a file and a script.

## A checklist that combines the sources

1. Prove a gap first. Run the task without the skill and note the failures. (Anthropic, Vincent)
2. Check that a skill is the right tool. A CLAUDE.md line, a hook, or a linter may be better. (Vincent, HumanLayer)
3. Write the description as a trigger, in the third person, with words a user would say. Add an exclusion line. (Anthropic, Vincent, BeerCode)
4. Do not summarise the workflow in the description. (Vincent)
5. Write a fixed process with a gate and a checkable end. (Pocock, BeerCode)
6. Write only what Claude does not know: gotchas and local facts. (Anthropic, Thariq Shihipar's article, below)
7. Keep the body under 500 lines. Split with conditional links, one level deep. (Anthropic, BeerCode)
8. Put fragile steps in scripts. Enforce limits in the environment: `allowed-tools`, `disable-model-invocation`, a model per skill. (Anthropic, BeerCode)
9. Explain why. Avoid all-caps rules. (Anthropic)
10. Build 10 to 20 cases. Run each 3 to 5 times. Use a separate judge. Test on more than one model. (BeerCode, Anthropic, Vincent)
11. Grow a "Gotchas" section from real failures. Thariq Shihipar calls it the highest-signal part of a skill (V, via a gist mirror of the original post, 2026-03-17: https://gist.github.com/Danm72/467f6d6cd193d19c0042371866d53b75).
12. Store data outside the skill folder. The folder can be deleted on upgrade. (same article)

## Common mistakes

- A workflow summary in the description.
- A vague or first-person description.
- Explaining things Claude already knows.
- Deep chains of reference files.
- No test at all.
- Rigid all-caps rules with no reason.
- Using a skill where a CLAUDE.md line or a plain slash command is enough.

## Unverified claims

Posts say skills trigger only about 50% of the time, and that a forced-activation hook fixes it. These came from community snippets. (U) Do not rely on those numbers. Measure your own trigger rate with the evals above.
