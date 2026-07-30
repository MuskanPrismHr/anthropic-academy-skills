---
name: "agentic-coding-advanced"
description: "Reference guide to running long, unattended, and team-scale agentic coding sessions (Anthropic's Claude Code, advanced usage): steering long sessions (plan mode, directed /compact, rewind, /goal, /loop, worktrees), writing a CLAUDE.md that's actually followed, verification skills, permission modes (manual/accept-edits/plan/auto/don't-ask/bypass), hooks (PreToolUse/PostToolUse/Stop/SessionStart, exit codes, redacting secrets), automating with routines vs. headless mode (-p) vs. the Agent SDK, GitHub Code Review vs. claude-code-action for CI, verifying unsupervised runs, and packaging setups as plugins. Use whenever the user asks how to make a Claude Code session more autonomous or trustworthy, configure hooks/permissions, automate or schedule a task, wire up GitHub Actions/PR review, verify unsupervised work, or share a Claude Code setup as a plugin."
---

## Purpose

Condensed reference to Anthropic's "Claude Code in Action" course — the advanced/production layer on top of the basics covered in `agentic-coding-101`. This course is about running long, unattended, and team-scale sessions you can actually trust: steering, configuration, automation, and verification. Pull the relevant section rather than repeating the whole thing.

## Steering long sessions

Short tasks are easy to check by eye. Long ones (a multi-hour refactor, a new feature across a dozen files) need two habits: **scope before Claude starts, steer while it runs.**

**Scope first with Plan Mode.** Claude researches read-only and hands back a plan before writing anything. Actually read it — don't skim. The more thorough the plan, the fewer surprises during execution; iterating on a plan is far cheaper than letting a run go sideways and cleaning up after.

**Steer while it runs:**
- **`/compact`** — summarizes the conversation, uses the summary as new context, discards the old messages. Risk: something important gets dropped and Claude drifts. Don't run it bare — add direction after the command so you control what the summary keeps, e.g. `/compact Focus on the --version flag implementation`.
- **Rewind** — double-tap Escape on an empty prompt to open the rewind menu. Every user prompt creates a checkpoint. Options: restore code and conversation together, restore just the conversation, restore just the code, summarize from a checkpoint forward (compress a side-tangent), or summarize up to a checkpoint (compress a long setup phase while keeping the implementation intact).
- **`/goal`** — sets a completion condition instead of a set of steps; Claude keeps working across turns until a fast evaluator confirms the condition is met (e.g., `/goal all tests in src/billing pass, and the type checker reports zero errors`). Clear it with `/goal clear`. Constraint: the evaluator only reads the transcript, so your condition must be checkable from what Claude actually outputs (like test results), not something external.
- **`/loop`** — reruns a prompt on an interval (fixed or self-paced) between turns; useful for polling something external (a CI run, a deploy) and reacting when state changes. Stop it with Escape.

**Parallel work: git worktrees.** Running multiple agents against the same codebase risks file conflicts — two sessions editing the same files is unsafe. Worktrees give each session its own independent file tree so they can't clobber each other; a worktree is cleaned up automatically when its session exits. A `.worktreeinclude` file at the repo root lists git-ignored files (env files, local config) to copy into every new worktree without committing them.

## Writing a CLAUDE.md that Claude actually follows

The trap: CLAUDE.md keeps growing one rule at a time until it's a wall of text Claude starts partially ignoring. This isn't a bug — **CLAUDE.md is guidance, not enforced configuration.** Every line competes with every other line for attention; the longer the file, the less reliably any single rule gets followed. The goal is a *tight* file, not a complete one.

**First question: does this rule even belong in CLAUDE.md?** Split rules into two jobs: soft conventions (naming, file layout) belong in CLAUDE.md. Hard lines that must never be crossed ("never push to main") belong in a **hook** instead — a hook is code that runs before an action and can actually block it; CLAUDE.md can only ask nicely. Move anything genuinely dangerous out of CLAUDE.md and into enforcement.

**Four file locations, all loaded together at launch (nothing dropped, they stack):**
- **Managed policy** — org-level, controlled by the platform team, cannot be excluded.
- **User** — personal preferences, follows you across every project on your machine.
- **Project** — shared with the team, checked into the repo.
- **Local** — git-ignored, personal notes scoped to just this one repo (e.g., architectural decisions for a branch you're working alone, that shouldn't affect the shared project file).

**Splitting a large file with imports** — `@.claude/conventions/code-style.md` style references organize a big file into pieces, but Claude expands them all inline at launch. **Imports organize; they do not reduce how much context gets loaded.**

**Phrasing determines whether a rule sticks:**
- **Be specific and checkable.** "Follow best practices for API routes" can't be checked by you or Claude. "Put new API routes in `src/api/handlers`, one per file" can.
- **Name the replacement, don't just ban something.** "Don't use default exports" leaves the door open to anything else. "Use named exports, not default exports" closes it.
- **Emphasis is a budget.** "IMPORTANT" and "YOU MUST" raise a rule's priority only relative to the quieter rules around it. If everything shouts, nothing stands out — spend that budget on the two or three rules that really hurt when broken.

**Keep it under revision.** Treat CLAUDE.md like living code. When Claude does the wrong thing, that's a bug report against the file, not just something to fix by hand — you can tell Claude directly "add that to CLAUDE.md" and it will write the rule for you.

## Verification skills

If there's one skill worth building first, it's a skill that checks your own work — because manual checking depends on you remembering to ask for it every time, and skipping that once lets bad code through.

**The shape:** a task finishes, it matches the skill's trigger description, the skill fires automatically (no need to ask), and it runs the test suite, reads the diff, confirms no test was quietly weakened just to pass, and reports pass/fail with evidence attached. "Done" means the gates were run and the results stated explicitly — not "the diff looked fine when I read it." The rule of thumb for *any* skill: if you've typed the same multi-step instruction twice, that's a skill candidate (a release checklist, a migration recipe, a pre-PR check all follow the same shape).

**A skill folder is more than one file.** Keep `SKILL.md` itself lean — name, trigger description, procedure — and push weight into side files: a `reference.md` for detailed material Claude only reads when it needs that depth, and scripts (like a `check.sh` running all the gates) that Claude *executes* rather than loading into context.

**Which surface owns which rule:** always-on conventions (naming, file layout) → CLAUDE.md. A procedure/reference tied to a specific kind of task → a skill. A rule Claude must never be able to skip → a hook, because CLAUDE.md and skills are instructions Claude follows, while a hook is code that actually runs — don't leave a non-negotiable rule to instruction-following.

## Permission modes

Decide once what Claude can run without stopping to ask, instead of approving every action individually.

- **Manual** — reads only without prompting; everything else asks first.
- **Accept edits** — reads, file edits, and common filesystem bash commands run without asking; good for iterating on code you review afterward.
- **Plan** — reads only; researches and proposes without editing anything.
- **Auto** — accepts everything, but a separate classifier model reviews each action before it runs, watching for moves that escalate beyond what was asked (production deploys/migrations, force-pushing, piping downloaded code into a shell, sending sensitive data externally, destroying session files) while waving through everyday work (local edits, installing from a lockfile, reads, pushes to your own branch).
- **Don't ask** — only pre-approved tools run; everything else is auto-denied with no prompt. The right mode for unattended contexts (CI, scheduled jobs, overnight batches) where no human is present to answer a prompt.
- **Bypass permissions** — skips all checks (equivalent to `--dangerously-skip-permissions`). Only ever run this inside an isolated container or VM.

Cycle the everyday modes (manual/accept edits/plan/auto) with **Shift+Tab**; the status bar always shows the active mode.

**The classifier in auto mode checks intent, not correctness** — it will wave through code that's broken but not dangerous. That's why auto mode should be paired with a **stop hook that runs your tests**: the classifier guards intent during the run, the stop hook guards correctness at the end. Two different jobs, both needed.

## Hooks

A CLAUDE.md rule is a request; a hook is deterministic code that runs at a fixed point in the loop and can guarantee behavior. It turns "Claude usually listens" into "Claude can't skip it."

**Events worth knowing** (of ~30 total):
- **PreToolUse** — fires before a tool call; the enforcement primitive, can stop something before it happens.
- **PostToolUse** — fires after a successful tool call; typical home for auto-formatting/linting.
- **Stop** — fires when Claude wants to end its turn; can refuse ("you're not done yet") if a condition isn't met. **SubagentStop** is the equivalent for a finishing subagent.
- **PreCompact / PostCompact** — around compaction.
- **InstructionsLoaded** — fires when a CLAUDE.md/rule file loads; useful for auditing what actually made it into context.
- **SessionStart** — fires at session start to prime the environment (use the `startup` source to target only fresh starts). **To re-inject context after a compaction, use `SessionStart` with the `compact` matcher — not `PostCompact`** — that's the one whose output actually lands back in the conversation.

**PreToolUse returns a decision as JSON**, exit code 0, with the key field `permissionDecision`:
- `allow` — let it through.
- `deny` — stop it.
- `ask` — hand the decision back to the user.
- `defer` — rare; only for non-interactive `-p` runs where a calling process pauses and resumes the tool later.

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "...",
    "updatedInput": { "command": "..." }
  }
}
```

`updatedInput` lets you **rewrite** a call instead of just blocking it — e.g., redact a secret out of a bash command and let the rest of it run. Caveat: `updatedInput` replaces the *whole* input object, so echo back every field you're not changing or you'll lose it.

**Exit codes for hooks that skip JSON:**
- `0` — success; stdout is parsed as JSON if present, otherwise plain text is ignored on most events — except `SessionStart`, `UserPromptSubmit`, and `UserPromptExpansion`, where plain text on stdout gets added to context (this is what makes a state-preserving hook work).
- `2` — blocking error; stderr is fed back to Claude as context. The blocking code almost everywhere, and the one that can even block `Stop` to say "not done yet." (`PostToolUse` fires after the tool already ran, so exit 2 there is too late to stop the call, though it can still feed text back.) A few events (`Notification`, `SessionStart`) ignore blocking entirely regardless of exit code.
- Anything else (including **1**, which feels like an error but is *not* blocking) — non-blocking; stderr gets logged, Claude carries on. If you meant to stop something, use `2`, not `1`.

**Real guardrail pattern — redact instead of block.** A `PreToolUse` hook on the Bash tool (matcher picks the tool; an optional `if` clause narrows to a specific command) can detect something like a live-looking secret (`sk_live_...`) in a command and swap it for a placeholder via `updatedInput` before the command runs. The work still happens; the secret never makes it through — a strictly better outcome than an outright block when the goal is safety, not obstruction.

**Preserving state across a compact:** a `SessionStart` hook with the `compact` matcher can print a short summary of what you were working on right after compaction, feeding it back into context so Claude picks up where it left off instead of starting cold.

## Automating repeat work: routines vs. headless mode vs. the Agent SDK

A spectrum from "build nothing" to "full control from your own code."

**Routines — a saved prompt that runs in the cloud on Anthropic's infrastructure.** Bundles a prompt, the target repo, and any connectors, and runs on a trigger — no server of yours to keep on, no workflow file to maintain. Triggers: a cron schedule, an HTTP POST to its API endpoint, or a GitHub event (e.g., a new PR). Good fits: a morning dependency audit, a PR triager, a daily ticket-priority scan. Create one at claude.ai/code/routines, or from inside a session with `/schedule daily dependency audit at 9am`.

**Three limits to know before relying on routines:** they're a research preview (behavior will keep shifting); a recurring schedule runs at most hourly; and each run starts from a fresh clone of the default branch, only able to push to `claude/`-prefixed branches unless that's loosened per-repo — the guardrail against an autonomous run rewriting `main`.

**Headless mode (`-p` / `--print`)** — for when the job needs your own environment or logic wrapped around the run. Runs Claude Code as a one-shot command with no interactive UI, reading stdin and writing stdout like any shell tool: `claude -p "summarize the changes in this diff"`. Note: `-p` **skips auto-discovery** of hooks, skills, plugins, MCP servers, and CLAUDE.md — you get Claude plus only what you allow explicitly (faster startup, but nothing ambient loads).

**Structured output:** pair a JSON schema with `--output-format json` to constrain output into a `structured_output` field you can pull with `jq` and pipe elsewhere:
```bash
claude -p "Extract the exported function names from src/core/style.js" \
  --output-format json \
  --json-schema '{"type":"object","properties":{"functions":{"type":"array","items":{"type":"string"}}},"required":["functions"]}' \
  | jq '.structured_output.functions'
```

**Multi-step automation across sessions:** capture the `session_id` from JSON output and resume it later — `claude --resume "$(jq -r .session_id /tmp/plan.json)"` — so one script produces a plan and a second carries it out with full context.

**`--bare`** — deterministic mode for CI, when you need repeatable output run to run rather than anything that varies.

**The Agent SDK** — embeds Claude Code as a library inside your own TypeScript/Python app. Exposes a `query` function and the same primitives as the CLI (`allowedTools`, a system prompt, a permission mode); you iterate over streamed messages and handle them however your app needs. Same engine as the CLI, callable from inside your product.

**Decision guide:** start with **routines** for anything that's the same prompt on a recurring trigger. Drop to **headless (`-p`)** when the job needs your pipeline and you want to pipe data through a script. Use **`--bare`** when CI needs identical results every run. Reach for the **Agent SDK** when the work needs to live inside your own product.

## GitHub Actions and Code Review

Two different tools for pull-request work, solving different problems.

**Managed path — Code Review.** An Anthropic-hosted service through the Claude GitHub app; nothing to build or host. An org admin enables it (Claude Code admin settings → Code review → Configure), installs the GitHub app, picks watched repos, and sets timing: once when a PR opens, on every push to the PR, or only on `@claude review`. Review agents analyze the diff against the *full* codebase (not just the changed lines in isolation), then post inline comments tagged by severity plus a summary table, deduplicated and ranked so you see real issues instead of a wall of nitpicks.

**Boundaries:** it never approves or blocks a PR — the judgment call stays human. There's no managed autofix; it only posts findings. It's a research preview on Team/Enterprise plans, so behavior will keep moving. To actually apply a finding, pull it down locally and run `/code-review --fix` against your working tree.

**DIY path — the GitHub Action.** Reach for this when the job goes beyond review: implementing a change from a comment, running scheduled reports, anything you'd normally write a CI workflow for. Set up with `/install-github-app` inside a session (requires repo admin) — walks through installing the GitHub app and setting the Anthropic API key secret.

The action is `anthropics/claude-code-action@v1`. Key inputs: `anthropic_api_key`, `github_token` (defaults to `secrets.GITHUB_TOKEN`), `trigger_phrase` (default `@claude`), `use_bedrock`/`use_vertex` for those providers, `prompt`, and `claude_args` — a string of CLI arguments passed straight through.

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    github_token: ${{ secrets.GITHUB_TOKEN }}
    trigger_phrase: "@claude"
    prompt: "Your instructions here"
    claude_args: "--max-turns 5 --model claude-sonnet-5"
```

A workflow can listen for `@claude` on PR/issue comments (Claude pushes commits and comments back), or run on a cron schedule for something like a daily rollup (add `workflow_dispatch` too, for a manual trigger from the Actions tab). Tune the unattended run through `claude_args`: `--max-turns N` caps the loop so it can't run forever; set a permission mode that won't stop to ask (nobody's there to answer); and restrict allowed tools to exactly what the job needs (read-only for a report).

**Decision guide:** PR review → managed **Code Review**, fix findings locally with `/code-review --fix`. Anything beyond review (implement, report, scheduled work) → the **GitHub Action**, tuned via `claude_args`.

## Verifying unsupervised runs

The core principle: **verify in proportion to how much rope you gave the run.** A short session you watched scroll by needs a glance. A run that fired unattended, or in CI with no one in the loop, needs a real reconstruction after the fact, because no one saw what actually happened.

**Keep unattended runs in auto mode rather than bypass permissions** — the classifier still screens each action for danger, which is a safety net worth keeping even unsupervised. But remember it judges intent, not correctness, so your verification bar doesn't move just because the classifier was watching.

**Start with the diff, not the summary.** Run `/code-review` to walk the changes and flag issues, then look at `git diff` yourself. A tidy, well-written summary is not proof of clean code — it can read perfectly while the actual diff touched a file you never expected. Read the files that were part of the plan first, then check for anything outside it.

**Turn tests into a gate, not a promise.** Don't trust a claim that tests passed — wire it as a hook so it can't be skipped: a **stop hook** that runs the test suite and refuses to end the turn on failure, and a **post-tool-use hook** that lints/type-checks after every edit. The exit code matters — `2` feeds the failure straight back to Claude, which then fixes it without being asked, and it fires on every run whether or not you remember to request it.

**Get a cold second opinion.** Open a fresh session or subagent with no memory of how the code was built and have it review the changed code. Because it has no stake in the approach taken, it catches things the original run rationalized past.

## Plugins (packaging and sharing a setup)

A `.claude` setup (skills, subagents, hooks, MCP configs) that works well is worth far more once the whole team runs it — but copy-pasting files between machines doesn't stay in sync. A **plugin** is one installable unit that moves the whole setup as a package.

**Using plugins someone else published.** Install one directly by name inside a session: `/plugin install org-name@plugin-name`, then `/reload-plugins` to apply it. For a team, add a **private marketplace** once — `/plugin marketplace add your-org/claude-plugins` — so every subsequent install resolves through it, giving centralized discovery, version tracking, and updates in one place. Browse what's available from the Discover tab.

**Read before you install — this is the part that matters most.** A plugin runs code on your machine with your privileges. Its hooks fire on every matching tool call whether or not you read them — a community plugin's Stop hook could call an external endpoint on every run with nothing in your config warning you. Claude Code shows what a plugin will install and an estimated context cost, plus an explicit warning that Anthropic doesn't control third-party plugin contents. Community-marketplace submissions get an automated review before listing; the official marketplace is curated on a separate track — but **reviewed is not the same as trusted.** Only install plugins, and add marketplaces, from sources you actually trust, and check what a plugin does before enabling it.

**How installed components interact with yours:** they run *alongside*, not over, your own setup. Hooks **stack** — a plugin's `PreToolUse` and your own both fire on every matching tool call, neither replacing the other (exactly why reading first matters). Skills, agents, and commands are namespaced under the plugin name so they can't collide with yours. A plugin's `settings.json` only has effect through two honored keys: the agent and subagent status-line keys — and the `agent` key can promote one of the plugin's subagents to the main thread, along with its system prompt, tool restrictions, and model, meaning a plugin install can change Claude Code's default behavior. Another reason to look before enabling.

**Packaging your own plugin.** No restructuring needed — a plugin uses the same `.claude` shape already in use: one folder per skill, one markdown file per subagent under `agents/`, `hooks/hooks.json` and `.mcp.json` at the plugin root. Claude Code discovers components by directory convention alone.

**The manifest** (optional) lives at `.claude-plugin/plugin.json`:
```json
{
  "name": "svg-splitter-review",
  "version": "0.1.0",
  "description": "Reviews the SVG Splitter repo",
  "author": { "name": "Lewis Menelaws" }
}
```
`name` is the only required field — it namespaces skills as `company-name:skill-name` to avoid collisions with anyone else's plugin. Version it like any dependency so updates and version tracking work across the team.

## Quick reference: what to reach for

- A run is going to take hours across many files → **Plan Mode** first, then steer with directed `/compact`, **rewind**, `/goal`, or `/loop` as needed; use **worktrees** if running multiple agents at once.
- CLAUDE.md is getting ignored → shrink it, move hard rules to **hooks**, split with imports (they organize, not shrink), make every rule specific/checkable and name the replacement, spend emphasis on only the top few rules.
- You keep manually re-checking the same thing after a task → build a **verification skill**, keep `SKILL.md` lean, push depth into `reference.md`/scripts.
- Deciding how much autonomy to grant → **manual/accept-edits/plan** for hands-on work, **auto** (+ a stop-hook test gate) for hands-off-but-supervised, **don't ask** for unattended CI/scheduled jobs, **bypass** only inside an isolated container.
- A rule must never be skippable → a **hook** (`PreToolUse` to block/redact, `Stop` to gate turn completion), not CLAUDE.md.
- Automating a recurring prompt → **routine** by default; drop to **headless (`-p`)** if it needs your pipeline; **`--bare`** for CI determinism; the **Agent SDK** if it needs to live inside your own product.
- PR work → managed **Code Review** for review/comments; the **GitHub Action** (`claude-code-action@v1`) for anything that needs to actually do something in CI.
- A run finished unsupervised → **verify in proportion to how unsupervised it was**: read the diff yourself, gate tests via a hook (not a claim), get a cold second-opinion review.
- A working `.claude` setup is worth sharing → package it as a **plugin**; read any third-party plugin's hooks/agents/MCP configs before installing one yourself.

