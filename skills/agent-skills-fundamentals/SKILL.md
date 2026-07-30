---
name: "agent-skills-fundamentals"
description: "Reference guide to how Agent Skills actually work under the hood in Claude Code - the SKILL.md structure and frontmatter fields (name, description, allowed-tools, model), how descriptions drive matching and confirmation-before-load, the Enterprise > Personal > Project > Plugins priority order for name conflicts, progressive disclosure via scripts/references/assets, how skills differ from CLAUDE.md/slash-commands/hooks/subagents/MCP, the gotcha that subagents don't inherit skills unless listed in their frontmatter, and a troubleshooting checklist for skills that won't trigger, won't load, or fail at runtime. Use whenever the user is authoring, debugging, sharing, or reasoning about Claude Code skills themselves - as opposed to skill-creator, which is a hands-on tool for generating and evaluating a specific skill's content."
---

## Purpose

This is a mechanics reference: how skills are structured, discovered, prioritized, and shared inside Claude Code, not how to write good skill content (that's `skill-creator`'s job). Reach for this skill when the question is about the system itself - why a skill isn't triggering, where to put it, how it compares to CLAUDE.md or a subagent, or why a subagent can't see a skill that works fine in the main conversation.

## What a skill is and why it's different from other customization

A skill is a directory containing a `SKILL.md` file with `name` and `description` in YAML frontmatter, plus instructions below. Claude loads only the name and description for every available skill at startup - the full body loads only when a request matches, and you get a confirmation before that happens. This is the core difference from the alternatives:

- CLAUDE.md loads into every conversation regardless of relevance - use it for always-on standards.
- Skills load on demand when the description matches - use them for task-specific expertise you don't want permanently in context.
- Slash commands require explicit typing - skills activate automatically when Claude recognizes the situation.
- Hooks are event-driven (fire on file saves, tool calls) - skills are request-driven (fire on what you're asking for).
- MCP servers provide external tools/integrations - a different category entirely, not a customization layer for Claude's own behavior.

These aren't competing options - a typical setup combines several. Don't force something into a skill just because skills are the newest mechanism.

## Where skills live and how conflicts resolve

Personal skills go in `~/.claude/skills` and follow you across every project. Project skills go in `.claude/skills` inside a repo and are shared with anyone who clones it (checked into version control). If a name collides across sources, priority resolves it in a fixed order: **Enterprise → Personal → Project → Plugins**. An enterprise-managed skill always wins over your personal one with the same name - if that's not what you want, rename your skill to something more distinct rather than fighting the priority order.

To update a skill, edit its `SKILL.md`. To remove one, delete its directory. Restart Claude Code after either change - skills are loaded at startup, not re-read live.

## Frontmatter fields

`name` and `description` are required; `allowed-tools` and `model` are optional. `description` is the field that matters most, because it's the only thing Claude sees before deciding whether to load the skill - it should answer both "what does this skill do" and "when should Claude use it," in language close to how you'd actually phrase the request, not just an abstract summary. `allowed-tools` restricts which tools are available while the skill is active, which is the mechanism for building read-only or otherwise constrained skills for security-sensitive workflows. `model` pins a specific Claude model for that skill's execution.

## Progressive disclosure for larger skills

`SKILL.md` shares the same context window as the conversation, so a 2,000-line file is expensive to keep loaded and unpleasant to maintain. The convention is to keep `SKILL.md` itself under roughly 500 lines with the essential instructions, and split anything larger into:

- `scripts/` - executable code (output consumes tokens, not the script source - so scripts stay cheap even if long)
- `references/` - supporting documentation, loaded only when the situation calls for it
- `assets/` - images, templates, or other static files

Link to these from `SKILL.md` with explicit instructions about when to read each one, so Claude only pulls in the reference material a given request actually needs rather than loading everything up front.

## Skills and subagents don't mix automatically

This is the most common surprise: subagents do not inherit your skills by default. A subagent starts from a clean context, and built-in agents (Explore, Plan, Verify-style agents) can't access skills at all, full stop, regardless of configuration. Only custom subagents you define in `.claude/agents` can use skills, and only the skills explicitly listed in that subagent's own frontmatter `skills` field:

```yaml
---
name: frontend-security-accessibility-reviewer
description: "Use this agent when you need to review frontend code for accessibility..."
tools: Bash, Glob, Grep, Read, WebFetch, WebSearch, Skill
model: sonnet
skills: accessibility-audit, performance-check
---
```

Skills listed this way load when the subagent starts, not on demand the way they do in the main conversation. If you're delegating review work to different specialized subagents (a frontend reviewer vs. a backend reviewer, for instance), this is the mechanism for giving each one its own fixed set of expertise rather than relying on prompt text to reproduce it every time.

## Sharing skills across a team or org

Project skills committed to `.claude/skills` share automatically through git - anyone who clones the repo gets them, no separate distribution step needed. Plugins package skills for distribution across repos via marketplaces, for broader reuse than a single repo. Enterprise managed settings push skills organization-wide with the highest priority, which is the right tool for mandatory standards and compliance rather than something individuals are expected to opt into.

## Troubleshooting checklist

Run the skills validator tool first - it catches structural problems (wrong file name, wrong location, malformed frontmatter) before you waste time debugging something else. Beyond that, match the symptom to the cause:

- Skill doesn't trigger at all → the description almost always is the problem; add trigger phrasing that matches how you actually ask for the task, not just a formal description.
- Skill doesn't load / doesn't show up → confirm `SKILL.md` sits inside a named subdirectory (not directly at the skills root) and that the filename is exactly `SKILL.md`.
- Wrong skill activates → two descriptions are too similar; make them more distinct rather than assuming Claude picked wrong.
- Plugin skill not appearing → clear cache and restart Claude Code first; if it's still missing, suspect the plugin's internal structure and run the validator.
- Runtime error after the skill loads → check for missing external dependencies (mention them in the description so Claude knows to expect them), missing execute permission on scripts (`chmod +x`), and path separators (use forward slashes everywhere, including on Windows).

`claude --debug` surfaces skill-loading and matching decisions directly, which is the fastest way to see what Claude actually did versus what you expected.

## Quick reference

- Deciding skill vs. CLAUDE.md vs. hook vs. subagent vs. MCP → skill if it's task-specific expertise that should apply automatically; CLAUDE.md if it should always be in context; hook if it's event-triggered; subagent if it needs an isolated execution context; MCP if it's an external tool/integration.
- Two skills with the same name conflict → check Enterprise > Personal > Project > Plugins; rename yours if you're not the one who should win.
- Skill isn't triggering → rewrite the description with phrasing closer to how you actually ask.
- Skill getting big and unwieldy → move detail into `references/`, code into `scripts/`, and keep `SKILL.md` under ~500 lines.
- Building a subagent that needs a skill → add a `skills:` field to that subagent's own frontmatter; don't assume it inherits from the main conversation.
- Anything not working as expected → run the validator, then `claude --debug`, before assuming the skill content itself is wrong.
