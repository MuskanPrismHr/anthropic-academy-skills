---
name: "agentic-coding-101"
description: "Reference guide to Anthropic's agentic coding tool (Claude Code): what it is, how the agentic loop/context window/permissions work, installing it (terminal, VS Code, JetBrains, Desktop, web), the explore-plan-code-commit workflow, context management (/compact, /clear, /context), subagents, the CLAUDE.md memory file, Skills, MCP servers, and Hooks. Use whenever the user asks how to use Claude Code effectively, how to set up or configure it, why a session is behaving a certain way, or how to structure CLAUDE.md, subagents, hooks, or MCP servers for a project — even if they don't name these features explicitly."
---

## Purpose

Condensed reference to Anthropic's "Claude Code 101" course. Use it to answer questions about how Claude Code works internally, help set up or troubleshoot a Claude Code environment, or explain a feature (CLAUDE.md, subagents, hooks, MCP, skills). Pull the relevant section rather than dumping the whole thing.

## What Claude Code is

An agentic coding tool: unlike Claude.ai, it has direct access to your files, terminal, and codebase, and does the work itself rather than handing back text to copy-paste. Available in the terminal, VS Code, JetBrains IDEs, the Claude Desktop app, and on the web (claude.ai/code).

An **agent** is software that interacts with its environment and takes actions toward a goal — an LLM operating in a loop, with access to tools (and sometimes other agents).

**What it can do:** read and understand a codebase (explain a feature, trace a bug), edit files across a project (refactor a function and update every caller), run terminal commands (build, test, install, and decide next steps from the output), search the web for docs/API references.

**Three things to keep in mind:**
1. **The context window** is Claude's working memory — large but finite. This is why it behaves "agentically," finding targeted ways to answer questions instead of loading the whole codebase.
2. **It asks permission** by default before editing files or running commands — you're in control of how hands-on you want to be.
3. **It can make mistakes** — misread intent, introduce a bug, over-engineer. Staying in the loop catches these early.

## The agentic loop

1. You enter a prompt.
2. Claude gathers context and the model returns text or a tool call.
3. It takes action (edits a file, runs a command).
4. It verifies the result against your goal.
5. If it's done, it stops and waits for the next prompt; if not, it loops back and keeps trying until the result is complete and verifiable.

You can add context, interrupt, or steer at any point in the loop.

**Context:** when the window fills up, Claude Code auto-compacts — summarizing and dropping tool-call results to reclaim space (this can lose detail, hence managing it proactively matters — see below).

**Tools** are what let Claude execute actions instead of just returning text (file read/write, web search, bash, etc.); Claude uses semantic understanding to decide when and how to call them.

**Permission modes:**
- **Default** — asks before editing a file or running a shell command.
- **Auto-accept** — file edits happen automatically; commands still need approval.
- **Plan mode** — read-only tools only; compiles a plan before doing any work.

Cycle between modes with **Shift+Tab**. All of this is configurable in settings — be cautious about giving free rein, since mistakes are harder to catch after the fact.

## Installing Claude Code

- **Terminal** (macOS/Linux/WSL): `curl` install (recommended — supports auto-update) or `brew install` (no auto-update). Windows: `Invoke-RestMethod` in PowerShell, `curl` in CMD, or `winget` (no auto-update). After install, run `claude` in your project directory — it gets access to that directory and all subfolders. First run walks you through color theme and sign-in (Pro/Max/Enterprise account or API key — pick Enterprise explicitly if your org has one).
- **VS Code:** install the "Claude Code" extension by Anthropic (blue verified check) from the Extensions panel, restart VS Code, open via Cmd/Ctrl+Shift+P → "Claude Code Open in New Tab," or the sidebar icon. Can opt out of the UI for a pure terminal experience in settings.
- **JetBrains:** install the plugin from the JetBrains Marketplace, restart the IDE, click the Claude logo.
- **Desktop app:** toggle "Code" at the top of the Claude Desktop app — same look as chat, but works in a specific folder, lets you change permissions, and can run in a cloud environment.
- **Web:** claude.ai/code, or the "Code" label in the desktop/web sidebar — restricted to GitHub repositories.

**Which to use:** terminal gets new features first if you want the cutting edge; IDE integrations feel more intertwined with your editor; Desktop is good for letting Claude run in the background while you do something else; web/Cloud is good for remote work against a GitHub repo.

## Your first prompt

- **Shift+Tab** cycles between approval mode (ask every time) and auto-accept mode (files auto-approved, commands still ask).
- **Plan Mode** (also in the Shift+Tab menu) uses read-only tools to research the codebase and your proposed implementation, asks clarifying questions, and returns a plan before touching anything — ideal for multi-step feature work or a safe code review.
- Be as descriptive as possible in your prompt; review the plan before approving it, then let Claude ask for approval at each step if you want to stay in the loop.

## The Explore → Plan → Code → Commit workflow

The single most important habit in the course — skipping straight to "write the code" means more course-correcting later.

1. **Explore & Plan** — enter Plan Mode (Shift+Tab), describe the problem ("figure out where in the pipeline this should happen, whether we need new dependencies, and how to approach it"). Claude reads relevant files, may search the web, and returns a plan. This is the cheapest point to course-correct, since no code has been written yet. You can also run the "explore" subagent outside plan mode just to get a codebase summary with no intent to change anything.
2. **Code** — approve the plan and let Claude work through it, either auto-accepting edits or approving each one. Tips: define explicit success criteria up front so Claude knows what "correct" looks like; add tools that remove back-and-forth (e.g., the Claude in Chrome extension so Claude Code can drive a browser tab and test a UI directly); give it a reliable test suite to validate against (make sure the tests are trustworthy before handing them off, to avoid false positives). If Claude keeps hitting the same issue, ask it to save the fix to CLAUDE.md.
3. **Commit** — test the changes yourself, run a subagent code reviewer first (fresh eyes, no session bias), then have Claude generate a commit message in your style, and push.

## Context management

- **`/compact`** — manually compacts everything up to that point, keeping a summarized memory of the session. Use when you're still working the same feature and hitting the limit.
- **`/clear`** — wipes everything. Use when starting a new feature so the old conversation doesn't bias the new one. Anything you want remembered across sessions belongs in CLAUDE.md instead, so Claude doesn't have to rediscover it.
- **`/context`** — shows a breakdown of what's consuming your context window.

**Ways to save context space:**
- Be specific in prompts — a vague prompt costs *more* context in the long run because Claude has to explore and reason more to fill the gap.
- Manage MCP servers — they load all their tool definitions into context by default even when unused; disable ones unrelated to the current project, or prefer Skills (name + description only loaded upfront, full contents loaded on demand).
- Use subagents for anything where you only need the final answer, not the exploration trail (e.g., "where are the auth endpoints?") — they run in their own context window and return just a summary.

## Code review features

- **Subagent review before pushing a PR** — runs in its own context window with no bias from the session that wrote the code. Restrict a code-reviewer subagent to read-only tools (it should flag, not edit) and check its config into the repo so the whole team uses the same reviewer.
- **`/commit-push-pr` skill** — handles commit, push, and PR creation in one step. If a Slack MCP server with channels listed in CLAUDE.md is configured, it can auto-post the PR link to the team channel.
- **`claude --from-pr <PR_NUMBER>`** — when Claude creates a PR via `gh pr create`, the session is linked to it; use this to resume that session later (address review comments, fix a failing build).

## CLAUDE.md — persistent project memory

Without it, every session starts from zero: Claude re-explores the codebase, re-derives dependencies, and sometimes assumes things incorrectly. CLAUDE.md is a Markdown file at your project root that Claude Code reads automatically every session — its contents get appended to your prompt. Think of it as an onboarding script for the codebase.

Example shape:
```
# Project
This is a Next.js 15 app using the App Router, Tailwind, and Drizzle ORM.

# Commands
- Dev server: `pnpm dev`
- Run tests: `pnpm test`
- Lint: `pnpm lint`

# Code Style
- Use 2-space indentation
- Prefer named exports
- All API routes go in app/api/
- Use server actions instead of API routes where possible
```

**Memory hierarchy:**
- **Project-level CLAUDE.md** — repo root, checked into version control, shared with the team.
- **User-level CLAUDE.md** — lives in your own config folder, applies across all your projects, for personal preferences.

**Tips:**
- When you catch yourself correcting Claude on the same thing repeatedly, explicitly ask it to save that rule to memory.
- Reference other project docs with `@path/to/file.md` (e.g., `@README.md`) so Claude reads them.
- Start a new project *without* a CLAUDE.md so you can see where you actually have to course-correct — this keeps the eventual file compact and relevant instead of speculative. Run `/init` when ready to have Claude generate one from what it's learned.

## Subagents

Subagents let Claude delegate a task to a separate instance with its own isolated context window, running in parallel. The exploration work (reading files, searching) doesn't clutter your main context — only the subagent's final summary comes back.

**Creating one:** run `/agents` → "Create new agent" → choose scope, define its purpose, pick which tools it can access, pick a color. Claude generates the name, description, and prompt, and the description is what Claude uses to decide when to invoke it automatically.

**Further customization:**
- Persistent memory — lets a subagent retain memory across conversations (useful if you reuse it on the same project repeatedly).
- Preloaded skills — add a `skills` key listing skill names; note that unlike skills in your main conversation, the entire skill content loads into the subagent's context upfront, not on demand.

## Skills (in Claude Code)

Skills are folders of instructions (and optionally scripts/resources) that Claude loads only when relevant — cheaper on context than an MCP server, since only the name + description sit in context until invoked, and Claude decides whether to pull in the rest. Useful as an alternative to an MCP server when you just need repeatable domain instructions rather than live tool access. For deeper coverage of building and structuring skills, see the dedicated "Introduction to agent skills" course/skill.

## MCP (Model Context Protocol)

An open standard connecting Claude Code to external tools and data sources — bridges the gap for context that lives outside your codebase (databases, productivity apps, public repos). Claude decides on its own when a query calls for one of these tools.

**Adding a server:** `claude mcp add`. Two transport types:
- **HTTP** — remote services hosted by the provider, connected over the network.
- **Stdio** — local processes running on your machine.

Manage active servers with **`/mcp`** inside a session (see what's connected, check status, disable unused ones).

**Scoping:**
- **Local** — current project only, just for you.
- **User** — across all your projects.
- **Project** — via a `.mcp.json` checked into version control, so everyone on the repo gets the same servers automatically.

**Context cost:** every configured MCP server adds its tool definitions to context, even unused ones — disable what you're not actively using. If a CLI equivalent exists (`gh` for GitHub, `aws` for AWS), prefer it — no persistent tool definitions. A Skill can also be a lighter-weight alternative to an MCP server for the same reason. If MCP tools exceed 10% of the context window, Claude Code auto-switches to a tool-search mode that discovers tools on demand (can be less reliable).

## Hooks

Hooks are the one thing in this course that's **deterministic** — they always run, unlike an instruction in CLAUDE.md ("run Prettier after every edit") which Claude usually but not always follows.

**Common uses:** auto-formatting after file edits, logging every executed command for compliance, blocking dangerous operations (writes to production config, `rm -rf`, commits to `main`), sending yourself a notification when Claude finishes.

**Events:**
- `PreToolUse` — before a tool call (can block it).
- `PostToolUse` — after a tool call completes.
- `UserPromptSubmit` — when you submit a prompt, before Claude processes it.
- `Stop` — when Claude finishes responding.
- `Notification` — when Claude sends a notification.

Configure via **`/hooks`** inside a session, or directly in `settings.json`. Each hook picks an event, an optional matcher (which tools it applies to), and a command to run.

**Example:** a `PostToolUse` hook matching `"Edit|MultiEdit|Write"` that runs the right formatter (Prettier, gofmt, etc.) based on file extension whenever Claude modifies a file.

**Blocking with `PreToolUse`:** the hook receives the tool name and input as JSON on stdin; its exit code controls behavior — `0` proceeds normally, `2` blocks the action and feeds the stderr message back to Claude as feedback so it can adjust, any other code is a non-blocking error shown to you but doesn't stop anything. This is how you enforce a hard rule instead of a suggestion.

**Sharing with a team:** hooks in `.claude/settings.json` are project-level and can be checked into the repo so everyone gets them. Use the `CLAUDE_PROJECT_DIR` environment variable in hook commands to reference project-relative scripts regardless of Claude's current working directory.

**Rule of thumb:** if something must happen every time without exception, put it in a hook, not a prompt.

## Quick reference: what to reach for

- Starting any non-trivial change → **Explore → Plan → Code → Commit**, using Plan Mode for the first two steps.
- Context filling up mid-feature → **`/compact`**; starting a new feature → **`/clear`**; want to see what's using space → **`/context`**.
- Want Claude to stop re-learning your stack every session → **CLAUDE.md** (project-level, checked in; user-level for personal preferences).
- Need an answer without polluting main context, or want a second, unbiased pass before a PR → **subagent**.
- Need live data/actions from an external system → **MCP server** (scope it to the project via `.mcp.json` if the team needs it too); if it's really just repeatable instructions, consider a **Skill** instead — cheaper on context.
- Need a rule that must never be skipped → **hook**, not a CLAUDE.md instruction.

