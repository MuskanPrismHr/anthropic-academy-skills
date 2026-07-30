# Anthropic Academy Skills

Seven [Claude Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) distilled from Anthropic Academy's Skilljar courses. Each `SKILL.md` is an original, condensed reference guide (not a verbatim copy of course text) covering the course's key concepts, organized for quick lookup and written to trigger automatically in Claude Code / Claude Desktop / Cowork when a relevant question comes up.

## Skills

| Skill | Source course | Covers |
|---|---|---|
| [`ai-fluency-101`](skills/ai-fluency-101/SKILL.md) | [Claude 101](https://anthropic.skilljar.com/claude-101) | Choosing chat vs. hand-off vs. coding mode, prompt writing basics, Projects, Artifacts, Skills, Connectors, Enterprise Search, Research |
| [`agentic-coding-101`](skills/agentic-coding-101/SKILL.md) | [Claude Code 101](https://anthropic.skilljar.com/claude-code-101) | The agentic loop, context window/permissions, install options, explore-plan-code-commit workflow, `/compact`/`/clear`, subagents, CLAUDE.md, Skills, MCP, Hooks |
| [`agent-platform-101`](skills/agent-platform-101/SKILL.md) | [Claude Platform 101](https://anthropic.skilljar.com/claude-platform-101) | Messages API, model tiers, the agent loop, tool use (custom/server/client tools), extended thinking, Skills vs. MCP vs. tools, context management, Managed Agents |
| [`cowork-fundamentals`](skills/cowork-fundamentals/SKILL.md) | [Introduction to Claude Cowork](https://anthropic.skilljar.com/introduction-to-claude-cowork) | What Cowork is, folders/connectors/permissions, writing a good delegation prompt, scheduled tasks, projects, skills, plugins, working safely |
| [`agentic-coding-advanced`](skills/agentic-coding-advanced/SKILL.md) | [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action) | Steering long sessions, CLAUDE.md that's actually followed, permission modes, hooks, headless mode/Agent SDK, CI review actions, packaging as plugins |
| [`ai-fluency-framework`](skills/ai-fluency-framework/SKILL.md) | [AI Fluency Framework Foundations](https://anthropic.skilljar.com/ai-fluency-framework-foundations) | Automation/Augmentation/Agency, generative AI fundamentals, the 4D competencies (Delegation, Description, Discernment, Diligence) |
| [`building-agents-with-the-api`](skills/building-agents-with-the-api/SKILL.md) | [Claude with the Anthropic API](https://anthropic.skilljar.com/claude-with-the-anthropic-api) | Messages API mechanics, prompt evaluation/engineering, custom tool-use loops, RAG, extended thinking/images/PDFs/citations/caching, building an MCP client & server, workflow vs. agent patterns |

## Using these skills

**Claude Code / Claude Desktop / Cowork:** copy the skill folder(s) you want into your skills directory (or install via your platform's skill/plugin manager), or point your assistant at this repo and ask it to install one.

**Anywhere else:** each `SKILL.md` is a self-contained Markdown file with YAML frontmatter (`name`, `description`) followed by the reference content — readable and usable as plain documentation even without a Skills-aware client.

## License

MIT — see [LICENSE](LICENSE). Use, modify, and redistribute freely.
