---
name: "subagent-fundamentals"
description: "Reference guide to designing and using Claude Code subagents well - what a subagent actually is (isolated context window, custom system prompt, summary-only return), built-in vs. custom subagents and how to create one via /agents, why the description field shapes both when a subagent triggers and what input prompt it's given, defining an output format so a subagent knows when it's done instead of running long, scoping tool access to just what the role needs, and the three anti-patterns that make subagents worse than doing the work in the main thread (fake expert personas, dependent sequential pipelines, test runners that hide full output). Use whenever the user is creating a subagent, debugging why one isn't triggering or is underperforming, or deciding whether a task should be delegated to a subagent at all."
---

## Purpose

Subagents trade visibility for a clean main context: the work happens somewhere else, and only a summary comes back. That trade is worth it for some tasks and actively harmful for others. This skill is about making that call correctly and configuring subagents so they actually deliver the clean-summary benefit instead of quietly losing information.

## What a subagent actually is

A subagent runs in its own conversation context window, separate from the main thread. It receives two inputs: a system prompt from its configuration file (its role and behavior), and a task description the main agent writes based on what you asked for. It then works independently - reading files, running searches, editing code - and when it finishes, only a summary returns to the main conversation. The subagent's full transcript is discarded.

This gives three concrete benefits: focused work (each subagent concentrates on one task), a clean main context (the intermediate noise - 15 file reads to answer one question - never accumulates there), and a concise result (you get the finding, not the search process). The cost is that you lose visibility into how the subagent got there.

## Built-in vs. custom subagents

Claude Code ships with built-in subagents you can use immediately: a general-purpose subagent for multi-step tasks needing both exploration and action, Explore for fast codebase search and navigation, and Plan for research during plan mode. Beyond those, custom subagents let you define your own role, system prompt, and tool access - a code reviewer, a test writer, a documentation generator, whatever your workflow needs.

Create one with the `/agents` slash command: choose project-level scope (available only in the current project) or user-level scope (shared across everything on your machine), then either write the configuration by hand or describe what you want and let Claude generate the name, description, and system prompt for you. During creation you also choose tool access from categories: read-only tools, edit tools, execution tools, MCP tools, and other tools.

## Frontmatter fields

- `name` - the identifier.
- `description` - does double duty: it's what the main agent matches against your request to decide *whether* to delegate, and it's also what the main agent uses as guidance for *what task description to write* when it does delegate. A vague description produces a vague input prompt for the subagent, not just an unreliable trigger - "use get diff to find the current changes" leaves the subagent guessing, versus a description with concrete example scenarios that shapes a specific, actionable task.
- `tools` - the tool list selected during creation; editable afterward.
- `model` - `sonnet`, `opus`, `haiku`, or `inherit`.
- `color` - UI identification only.

Everything below the frontmatter is the system prompt - what the subagent should focus on, how it should analyze, how it should report back. A specific system prompt is the difference between a subagent that's actually useful and one that misses the point.

## Designing a subagent that finishes cleanly

Two configuration choices determine whether a subagent behaves predictably:

**Define an output format.** Without one, a subagent struggles to decide when it's done and tends to run longer than necessary. Give it a structured template - for a code-review subagent, something like Summary / Critical Issues / Major Issues / Minor Issues / Recommendations / Approval Status - so it has a checklist to fill in and a clear stopping point. Add a section like "Obstacles Encountered" (setup issues, workarounds, unusual flags, dependency problems) so the main thread doesn't have to rediscover things the subagent already ran into.

**Scope tool access to the role.** A research/read-only subagent needs Glob, Grep, and Read - nothing that could modify anything. A reviewer might additionally need Bash to run checks. Only an agent that should actually change code needs Edit/Write. This isn't just safety - it also makes each subagent's role unambiguous when you're running several.

## When delegating actually helps

Subagents are worth the overhead when exploration is separable from execution: you need a result, not the play-by-play of finding it, and the exploratory work would otherwise clutter the main thread. Research is the clean case - tracing how authentication works in an unfamiliar codebase can mean reading dozens of files, and the main thread only needs "JWT validation happens in middleware/auth.js line 42," not the search history.

A custom system prompt earns its keep when it changes behavior in a way the main thread's default prompt doesn't support - a copywriting subagent with explicit tone/audience/style instructions (Claude Code's default leans concise and technical, wrong for a landing page), or a styling subagent pointed at your design-system files so those conventions load into its context automatically. Generic "you are a Python expert" or "you are a Kubernetes specialist" framing adds nothing, though - Claude already has that knowledge without the persona, and a fake-expert subagent can't do anything the main thread couldn't do directly.

## Three anti-patterns

- **Persona subagents with no real capability difference.** If nothing about the system prompt, tools, or loaded context actually changes what the subagent can do, the "expert" framing is decoration, not delegation.
- **Sequential pipelines where each step depends on the last.** A three-agent flow (reproduce a bug → debug it → fix it) looks appealing but fails in practice, because information gets lost in the handoff between agents - and bug fixing in particular almost always needs each step to see what the previous one actually found. Pipelines work only when the stages are genuinely independent.
- **Test runners that compress away the output you need.** A subagent that returns "tests failed" instead of the full failure output forces you to go build separate debug tooling to recover information that would have just been visible in direct output. This pattern has measured worse than running tests directly.

## Quick reference

- Task needs a result but not the search process, and the exploration would clutter your main thread → delegate to a subagent.
- Each step of a task depends on what the previous step discovered → keep it in the main thread; don't chain subagents.
- Subagent isn't triggering when expected → the description is too vague; add concrete trigger phrasing and example scenarios.
- Subagent runs unpredictably long → give it a defined output format/template so it has a stopping point.
- Subagent's summary is missing detail you needed → add an explicit section (like "Obstacles Encountered") to the output format rather than assuming it'll report it unprompted.
- About to add a "you are an expert in X" system prompt → ask what capability that actually adds beyond what the main thread already has; if the answer is nothing, skip the subagent.
- Running tests via a subagent → don't; run them directly so you keep full failure output.
