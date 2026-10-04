# Sources

Every source used in these notes. Status: **V** = the author read it (date read: 2026-10-03 unless noted). **U** = unverified: only a search snippet, a second-hand report, or a page that failed to load. "Undated" means the page shows no date. Some pages were read through a fetch tool that summarises text, so quotes may differ from the original. Check before you quote.

## Anthropic documentation (all V, undated, current on 2026-10-03)

| URL | Used for |
|---|---|
| https://code.claude.com/docs/en/memory | Memory, CLAUDE.md, load limits |
| https://code.claude.com/docs/en/best-practices | Context, memory, habits |
| https://code.claude.com/docs/en/costs | Cost figures, agent team cost |
| https://code.claude.com/docs/en/context-window | Startup load |
| https://code.claude.com/docs/en/checkpointing | `/rewind`, summarise |
| https://code.claude.com/docs/en/commands | Command list |
| https://code.claude.com/docs/en/interactive-mode | Keys, modes |
| https://code.claude.com/docs/en/permission-modes | Permission modes |
| https://code.claude.com/docs/en/output-styles | Output styles |
| https://code.claude.com/docs/en/model-config | Model aliases, effort |
| https://code.claude.com/docs/en/fast-mode | Fast mode |
| https://code.claude.com/docs/en/fullscreen | Display |
| https://code.claude.com/docs/en/settings-reference | Settings |
| https://code.claude.com/docs/en/prompt-caching | Caching in Claude Code |
| https://code.claude.com/docs/en/voice-dictation | `/voice` |
| https://code.claude.com/docs/en/skills | Skill levels, scopes |
| https://code.claude.com/docs/en/sub-agents | Agent files, `skills:` field |
| https://code.claude.com/docs/en/agent-teams | Agent teams, failures |
| https://code.claude.com/docs/en/agent-sdk/overview | Agent SDK |
| https://code.claude.com/docs/en/headless | `claude -p` |
| https://code.claude.com/docs/en/github-actions | GitHub Action |
| https://code.claude.com/docs/en/claude-code-on-the-web | Cloud sessions |
| https://code.claude.com/docs/en/remote-control | Remote Control (V for the opening only) |
| https://code.claude.com/docs/en/channels | Channels |
| https://code.claude.com/docs/en/routines | Routines |
| https://code.claude.com/docs/en/slack | Claude Code in Slack |
| https://code.claude.com/docs/en/hooks-guide | Hooks |
| https://code.claude.com/docs/en/security | Security |
| https://code.claude.com/docs/en/mcp | MCP |
| https://claude.com/docs/claude-tag/overview | Claude Tag |
| https://claude.com/docs/claude-tag/concepts/security-and-data | Claude Tag security |
| https://platform.claude.com/docs/en/about-claude/pricing | Prices (checked 2026-10-03) |
| https://platform.claude.com/docs/en/models/overview | Model lineup |
| https://platform.claude.com/docs/en/build-with-claude/prompt-caching | Caching rules |
| https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools | How Claude sees a tool |
| https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices | Skill best practices (V, summarising fetch tool) |

## Anthropic engineering posts

| URL | Author, date | Status |
|---|---|---|
| https://www.anthropic.com/engineering/building-effective-agents | Anthropic, undated | V (summarised fetch) |
| https://www.anthropic.com/engineering/writing-tools-for-agents | Anthropic, undated | V (summarised fetch) |
| https://www.anthropic.com/engineering/multi-agent-research-system | Hadfield, Zhang, Lien, Scholz, Fox, Ford, 2025-06-13 | V (summarised fetch) |
| https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents | Anthropic, undated | V |

## Skills sources

| URL | Author, date | Status |
|---|---|---|
| https://github.com/anthropics/skills | Anthropic, undated | V |
| https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md | Anthropic, undated | V |
| https://github.com/obra/superpowers/blob/main/skills/writing-skills/SKILL.md | Jesse Vincent, undated | V (raw file) |
| https://blog.fsck.com/2025/10/16/skills-for-claude/ | Jesse Vincent, 2025-10-16 | V (summarised fetch) |
| https://blog.fsck.com/2025/10/09/superpowers/ | Jesse Vincent, 2025-10-09 | V (summarised fetch) |
| https://github.com/mattpocock/skills | Matt Pocock, head commit 2026-09-29 | V |
| https://github.com/mattpocock/skills/blob/main/README.md | Matt Pocock, undated | V |
| https://github.com/mattpocock/skills/blob/main/skills/productivity/grill-me/SKILL.md | Matt Pocock, undated | V |
| https://github.com/mattpocock/skills/blob/main/skills/productivity/writing-for-agents/SKILL.md | Matt Pocock, undated | V |
| https://github.com/mattpocock/dictionary-of-ai-coding/blob/main/dictionary/Smart%20zone.md | Matt Pocock, undated | V |
| Posts by Matt Pocock on X about the dumb zone and ticket size | Matt Pocock, dates unverified | U (search snippets only; the page returned HTTP 402) |
| "Як правильно працювати з Agent Skills" (YouTube, BeerCode channel) | Kyrylo, 2026-08-09 | V for the captions (auto-generated, auto-translated); on-screen content U; SkillsBench numbers not checked. Video URL not recorded. |
| https://gist.github.com/Danm72/467f6d6cd193d19c0042371866d53b75 | Thariq Shihipar, "Lessons from Building Claude Code: How We Use Skills", 2026-03-17 (gist mirror of an X article) | V (summarised fetch) |
| https://simonwillison.net/2025/Oct/16/claude-skills/ | Simon Willison, 2025-10-16 | V (summarised fetch) |
| https://github.com/travisvn/awesome-claude-skills | travisvn, updated 2026-02 | V (index only) |

## Memory and CLAUDE.md sources

| URL | Author, date | Status |
|---|---|---|
| https://www.humanlayer.dev/blog/writing-a-good-claude-md | Kyle, HumanLayer, 2025-11-25 | V (summarised fetch) |
| https://blog.sshh.io/p/how-i-use-every-claude-code-feature | Shrivu Shankar, 2025-11-02 | V (summarised fetch) |
| https://blogs.cisco.com/ai/identifying-and-remediating-a-persistent-memory-compromise-in-claude-code | Cisco, 2026-04-01 | V |
| dev.to post on memory growth | Odilon Hugonnot, 2026-04-05 | V (summarised fetch); URL not recorded |
| GitHub issues #4017, #11545, #62812, #48783 in the Claude Code issue tracker | various | V (summarised fetch) |
| https://simonwillison.net/2026/Sep/18/thariq-shihipar/ | Simon Willison, 2026-09-18 (`AGENTS.md` fallback, v2.1.277) | V |
| https://github.com/ykdojo/claude-code-tips | ykdojo | V |
| https://www.frenxt.com/cables/claude-code/cherny-03-claude-md-postmortem | frenxt, secondary report on Boris Cherny | U |
| https://howborisusesclaudecode.com/ | Secondary roundup of Boris Cherny tips; links to X posts | U |
| https://www.threads.com/@boris_cherny/post/DHq60G7vkNz/ | Boris Cherny, `#` shortcut announcement | U (search result only) |
| https://x.com/bcherny/status/1977163445205450783 | Boris Cherny, auto-compact near 155k | U (snippet only; HTTP 402) |
| Vector-store vendor article on memory contradictions | vendor | U |

## Agents and parallel work

| URL | Author, date | Status |
|---|---|---|
| https://simonwillison.net/2025/Oct/5/parallel-coding-agents/ | Simon Willison, 2025-10-05 | V |
| https://lucumr.pocoo.org/2026/2/13/the-final-bottleneck/ | Armin Ronacher, 2026-02-13 | V |
| https://addyosmani.com/blog/future-agentic-coding/ | Addy Osmani, 2026-01-02 | V (the "90% of engineers" figure is not checked) |
| https://steve-yegge.medium.com/welcome-to-gas-town-4f25ee16dd04 | Steve Yegge, probably 2026-01 | U (fetch returned 403) |
| https://github.com/ghuntley/how-to-ralph-wiggum | Geoffrey Huntley, mid-2025 | U (snippets) |
| Worktree guides from several blogs | various | U (snippets) |

## Voice

| URL | Author, date | Status |
|---|---|---|
| https://code.claude.com/docs/en/voice-dictation | Anthropic | V |
| https://dev.to/dmitriusan/code-dictator-vs-wispr-flow-vs-claude-code-vs-copilot-cli-free-voice-dictation-for-ai-coding-in-vs-a8m | Dmytro Lisnichenko, 2026-09-13 | V; the author may sell a competing tool |
| https://willowvoice.com/blog/voice-ai-dictation-claude-code | Willow Voice, 2026-10-01 | V; vendor marketing |
| https://wisprflow.ai/vibe-coding | Wispr Flow, undated | V; vendor marketing |
| https://aquavoice.com/blog/voice-dictation-claude-code | Aqua Voice | V; vendor claims |

## Other

| URL | Author, date | Status |
|---|---|---|
| https://simonwillison.net/guides/agentic-engineering-patterns/red-green-tdd/ | Simon Willison | listed, not used in these notes |
| https://newsletter.pragmaticengineer.com/p/how-claude-code-is-built | Gergely Orosz | listed, not used in these notes |
| Posts by @ClaudeDevs, @bcherny, and @trq212 on X | various | U (the site was blocked; only search-result text) |
| Reddit threads | various | not read (blocked) |
