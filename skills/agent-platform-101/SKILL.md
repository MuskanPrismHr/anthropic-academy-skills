---
name: "agent-platform-101"
description: "Reference guide to Anthropic's developer API platform for building with Claude programmatically: the Messages API, choosing a model tier (Fable/Opus/Sonnet/Haiku), the agent loop, custom tool use vs. built-in server tools (web search, code execution, web fetch) vs. client tools, extended thinking, Skills vs. MCP vs. tools, context management patterns (just-in-time context, server-side compaction, prompt caching, the memory tool), and Managed Agents (agents/environments/sessions/events) for long-running hosted agent workloads. Use whenever the user asks about calling the Claude API, writing agentic code against the SDK, picking a model for a workload, designing tool schemas, setting up MCP servers or Skills for API use, managing context/token costs in an API integration, or building/deploying agents at scale — even if they don't name these features explicitly."
---

## Purpose

Condensed reference to Anthropic's "Claude Platform 101" course — the developer-facing API platform, as distinct from the consumer apps (Chat/Cowork) or Claude Code as an interactive tool. Use this when the conversation is about writing code against the Claude API/SDK, not about using the desktop or web chat product. Pull the relevant section rather than repeating the whole thing.

## What the Claude Platform is

Anthropic's infrastructure for building with Claude programmatically: send structured requests from your own code, get structured responses back, with full control over model, token budget, tools, and system instructions. Made up of a REST API, SDKs (TypeScript, Python, Ruby, etc.), CLIs, and the Claude Console (API keys, usage monitoring, managed agent deployment, prompt testing).

**Three layers**, and the shorthand for how they relate: **build with primitives, scale on infrastructure, run with control.**
- **Primitives** — the API building blocks: Messages API, tool use, files, web search, code execution, MCP servers, Skills.
- **Infrastructure** — what takes an agent from prototype to scale: managed agents, retries, queues, observability.
- **Controls** — running it in production: dashboards, evals.

A single `messages.create` call is enough to wire Claude into an existing product feature (e.g., a "draft a reply" button in a help desk app) — you're not building a chatbot, you're adding Claude to something that already exists.

## Your first API call

Get an API key from platform.claude.com (requires purchasing credits), store it in `.env.local` (never hardcode it — that's how keys end up leaked on GitHub), then `npm install @anthropic-ai/sdk` (or the equivalent for your language).

Every call goes through `messages.create` with three required pieces: a **model**, a **max_tokens** cap, and a **messages** array of `{role, content}` objects (`user`/`assistant`).

```js
const msg = await client.messages.create({
  model: "claude-opus-4-7",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello, Claude" }],
});
```

Add a **system** prompt to shape persona/behavior (e.g., "You are a terse senior code reviewer. Give feedback in one paragraph."). The response's `content` is an **array of blocks** — not a plain string — since Claude can return text, tool calls, and thinking blocks together; always loop and check `block.type` rather than assuming a single string.

## Choosing the right model

Four tiers, selected via the `model` parameter:

- **Claude Fable** — sits above Opus, for the toughest problems, significantly higher cost — reserve it for work where that extra capability is worth paying for.
- **Claude Opus** — most capable of the three core families, but slowest and highest cost. Deep reasoning, complex analysis, multi-step coding, nuanced writing.
- **Claude Sonnet** — the sweet spot of intelligence, speed, and cost for most production work.
- **Claude Haiku** — fastest and cheapest, optimized for speed/cost over maximum intelligence. High-volume, low-complexity work: classification, extraction, routing.

**Process:** before writing production code, build a simple eval — 20-30 representative examples from your actual workload, scored against what "good" means for your use case. Run them through Haiku first; if quality holds, stop there. If not, step up to Sonnet. Only reach for Opus (or Fable) when the task actually needs it. `response.usage` reports input/output tokens, which is what your bill is based on — useful for comparing tiers side by side on the same prompt.

**In production, route per-task, not per-app**: e.g., an operations dashboard might classify every incoming file with Haiku, draft client updates with Sonnet, and reserve Opus only for RFP responses — one queue, three models.

## The agent loop

An **agent** is Claude running both sides of the messaging loop without a human in the middle: it receives a task, picks a tool, executes in a loop until it decides the task is done.

**The loop:**
1. Send a message with `tools` available.
2. Claude responds with either a final answer or a tool-use request.
3. Your code executes that tool.
4. Send the result back to Claude as a `tool_result` block.
5. Repeat until `stop_reason` is `end_turn`.

```python
while True:
    response = client.messages.create(model=..., max_tokens=..., tools=tools, messages=messages)
    if response.stop_reason == "end_turn":
        # print text blocks and break
        break
    if response.stop_reason == "tool_use":
        tool_results = [...]  # run each tool_use block, wrap as tool_result
        messages.append({"role": "assistant", "content": response.content})
        messages.append({"role": "user", "content": tool_results})
```

You own the loop and the tools; Claude owns the reasoning. This same shape scales from a toy weather demo to a production compliance agent that reads a report, looks up relevant codes via a tool, and writes findings to a database — only the tools and plumbing change. When you don't want to own the loop yourself, **managed agents** run this exact loop on Anthropic's infrastructure (see below).

## Tool use

A tool is a function you define and expose to Claude via a JSON schema (`name`, `description`, `input_schema`) passed in the `tools` array. **Claude decides when to call it; your code executes it** — Claude never runs the tool itself.

**The description is what Claude reads to decide whether to call the tool.** A vague description is the single biggest cause of agents misfiring or ignoring an available tool — be specific.

With multiple tools, Claude reads each description and matches it to the right piece of the request (e.g., `get_weather` for today, `get_forecast` for the next few days) — sometimes calling several in the same turn. Adding a tool is just: add it to the array, add a dispatch case for it, done.

**Skip the boilerplate — use the tool runner:** shipped in the SDK for TypeScript, Python, and Ruby. Pass your actual functions in; the runner builds the JSON schema from your function signatures/docs and handles the whole tool-use/tool-result loop internally. `runner.untilDone()` returns the final assistant message once everything settles — no manual while-loop, no stop-reason switch, no hand-written schema.

## Built-in tools: server tools vs. client tools

**Server tools** are pre-built and run on Anthropic's infrastructure, not yours — no agent loop needed, since Claude calls them and the result comes back in the same response:
- **Web search** — searches the internet, returns results with citations.
- **Code execution** — writes and runs Python in a sandbox.
- **Web fetch** — retrieves full content from URLs.

Look for `server_tool_use` blocks (the tool call) and tool-result blocks (e.g., `bash_code_execution_tool_result`) alongside regular text blocks — there's no `stop_reason` switching to write. Reminder: something being backed by a live web search doesn't make it true — verify before shipping high-stakes claims.

**Client tools** run where your code runs, but ship with the SDK so you don't write the schema yourself — e.g., **memory** (Claude reads/writes memory across sessions) and **bash** (a persistent shell).

## Extended thinking

Lets Claude reason step-by-step before answering — the internal chain of thought is visible in the response, not hidden. Helps with math/multi-step logic, code debugging, regulatory analysis, and anything involving trade-offs; skip it for simple classification/extraction/boilerplate, where it only adds latency and cost.

With **Opus 4.7**, thinking is **adaptive** — no token budget to pick; just turn it on with `thinking: {"type": "adaptive"}` and control depth with the **`effort`** parameter (inside `output_config`, not next to the thinking block): `low`, `medium`, `high` (default), `xhigh`, `max`.

```python
response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=16000,
    thinking={"type": "adaptive"},
    output_config={"effort": "high"},
    tools=[weather_tool],
    messages=[...],
)
```

## Skills (API context)

A Skill is a `SKILL.md` file (plus optional scripts/resources) that teaches Claude a **procedure** — your status report format, your review checklist — as opposed to a tool, which connects Claude to **data or actions**. Rule of thumb: *tools are what Claude can do; Skills are how you want it done.*

Skills load progressively — only name + description load at startup; the full Skill loads into context only once the agent decides it's relevant, keeping context lean even with many Skills available.

**Upload once, reference by ID:**
```python
skill = client.beta.skills.create(display_title="Status Report Generator", files=files_from_dir("status-report-skill"))
```

**Attach via `container.skills`** on a `messages.create` call (currently beta — pass the `skills-*` beta header):
```python
response = client.beta.messages.create(
    model="claude-sonnet-4-5", max_tokens=4096,
    betas=["skills-2025-10-02", "code-execution-2025-08-25"],
    container={"skills": [{"type": "custom", "skill_id": skill.id, "version": "latest"}]},
    tools=[{"type": "code_execution_20250825", "name": "code_execution"}],
    messages=[...],
)
```
`container.skills` is a list, so multiple Skills can layer onto one call. Skills often pair with code execution, since a Skill's procedure may need to actually run something.

## MCP (API context)

MCP exists to shift integration maintenance to the **service provider**. Without it, connecting to Asana + Slack + Google Calendar means writing and then perpetually maintaining three API wrappers yourself. With MCP, each provider publishes and maintains their own server — when their API changes, they update the server, you change nothing.

**Three-way split:** tools are for *your* systems (you own the maintenance), Skills are for *your* processes (instructions, not necessarily integrations), MCP is for *everyone else's* systems (they own the maintenance).

**Connecting:** declare the connection in `mcp_servers` (type, URL, name, optional auth token), then grant access via an `mcp_toolset` entry in `tools`. Claude introspects the server and discovers its tools on its own — you write zero tool schemas. Currently beta (`mcp-client-*` header required).

```python
response = client.beta.messages.create(
    model="claude-opus-4-8", max_tokens=1000,
    messages=[{"role": "user", "content": "What tools do you have available?"}],
    mcp_servers=[{"type": "url", "url": "https://mcp.linear.app/mcp", "name": "linear", "authorization_token": os.environ["LINEAR_MCP_TOKEN"]}],
    tools=[{"type": "mcp_toolset", "mcp_server_name": "linear"}],
    betas=["mcp-client-2025-11-20"],
)
```

**Scoping down access** (e.g., read-only): disable everything by default, then enable specific tools:
```python
{"type": "mcp_toolset", "mcp_server_name": "slack",
 "default_config": {"enabled": False},
 "configs": {"search_messages": {"enabled": True}, "list_channels": {"enabled": True}}}
```
Useful when you trust a service for reads but don't want Claude writing on your behalf by accident. See modelcontextprotocol.io for the server directory and protocol docs.

## Context management

Context is everything Claude sees on a turn — system prompt, message history, tool definitions/results, attached files/skills, thinking blocks. You pay for it both in and out, and once the window is full the request fails. The goal isn't fitting everything in; it's fitting the *right* things in. Four patterns (three API features, one design pattern):

1. **Just-in-time context** (design pattern, no special API) — don't preload everything; load what's needed now and let tools pull more in on demand (e.g., a compliance agent calls `lookup_building_code` for a specific section instead of loading the whole code book upfront).
2. **Server-side compaction** — opt in with a `context_management` key:
   ```python
   context_management={"edits": [{"type": "compact"}]}
   ```
   The API auto-summarizes old turns once input crosses the trigger threshold — no manual conversation-length tracking needed.
3. **Prompt caching** — mark stable parts of a request (system prompt, tool definitions, a long reference document) for reuse across calls at a fraction of the cost. Matters a lot at volume: a 4,000-token system prompt called 100x/hour is the difference between a normal bill and an unpleasant one.
4. **The memory tool** — for context that must survive across sessions (user preferences, running notes, prior decisions). Claude reads/writes a memory directory via tool calls; you implement the storage backend (filesystem, DB, encrypted store — your choice); Anthropic auto-injects an instruction telling Claude to check memory before starting work.

In production these layer together — e.g., a compliance agent caches its system prompt/tools and pulls code sections in just-in-time. Managed agents (below) ship with caching and compaction on by default.

## Managed Agents

A suite of APIs for building and deploying agents **at scale, hosted on Anthropic's infrastructure** instead of your own server — enabled by default on every API account, no special access needed. Same familiar agent loop (reason → tool call → read result → repeat) but running inside an isolated container with file system access, bash execution, and web search, resumable across network hiccups, and streamed back to your app in real time.

**Reach for managed agents when** the loop would run too long (minutes to hours), touch too many tools, need to keep state, or need to survive interruption. **Reach for a manual loop when** you want full control and the task is short-lived.

**Four primitives, in order:**
1. **Agent** — the reusable persona: model, system prompt, toolset.
2. **Environment** — where it runs: cloud or local, networking config, packages.
3. **Session** — one run of an agent inside an environment; the unit of work.
4. **Events** — everything flowing in/out: actions, tool calls, results, replies.

**Minimal example (create → environment → session → stream):**
```python
agent = client.beta.agents.create(
    name="Line Counter", model="claude-opus-4-8",
    system="You are a helpful agent that completes small file tasks.",
    tools=[{"type": "agent_toolset_20260401", "default_config": {"enabled": True}}],
)
environment = client.beta.environments.create(
    name="line-counter-env", config={"type": "cloud", "networking": {"type": "unrestricted"}},
)
session = client.beta.sessions.create(agent=agent.id, environment_id=environment.id, title="Count lines demo")

with client.beta.sessions.events.stream(session_id=session.id) as stream:
    # open the stream BEFORE sending the kickoff — it only delivers events after it opens
    client.beta.sessions.events.send(
        session_id=session.id,
        events=[{"type": "user.message", "content": [{"type": "text", "text": "Create a file, count its lines, report back."}]}],
    )
    for event in stream:
        if event.type == "agent.message":
            ...  # Claude's text
        elif event.type == "agent.tool_use":
            ...  # which tool it picked
        elif event.type == "session.status_idle":
            break  # done
```

**What managed agents add on top of the loop:** rubrics + a separate grader (its own context window) that scores output against your success criteria, with Claude iterating until it passes; memory that persists across runs (so a recurring agent can say "costs are down 15% since last week" instead of re-deriving everything); MCP connections to third-party services; multi-agent coordination (a coordinator delegating to specialists sharing a file system, synthesizing their findings); permissions policies that hold sensitive actions (like posting to Slack) for human approval before they fire. Sessions run in parallel — you can kick off a second one while the first is still working.

## Building Claude-API code with Claude Code

Rather than hand-writing the tool-definition/runner boilerplate yourself, stub the file (function signatures only) and hand it to Claude Code. It has a built-in **Claude API skill** (`/claude-api`) that auto-invokes when it detects the TypeScript SDK in use, or install it via `/plugin marketplace add AnthropicsSkills` (note the "s" at the end) if missing.

**A good prompt names three things:** the file to change, the pattern to use (e.g., "use the tool runner"), and the end state you expect. Claude Code then fills in the stub, appends a call, runs the script, and patches errors it hits along the way — review the diff rather than writing it from scratch.

**The recurring shape to recognize across almost everything in this course:** define a tool → hand it to a runner → return the result.

## Quick reference: what to reach for

- Wiring Claude into one existing feature/endpoint → plain **`messages.create`**.
- Task needs external data/actions from *your* systems → **custom tool** (write a tight, specific description).
- Task needs a common capability (search, run code, fetch a URL) → **server tool**, no loop required.
- Task needs to follow *your* procedure/template every time → **Skill**.
- Task needs a *third party's* live data/actions → **MCP server** (they maintain it, not you).
- Multi-step trade-off reasoning (not simple extraction) → turn on **extended thinking**.
- Context keeps overflowing → combine **just-in-time loading**, **server-side compaction**, **prompt caching**, and the **memory tool** as needed.
- The loop needs to run long, touch many tools, persist state, or survive a hiccup → **Managed Agents** instead of a hand-rolled loop.
- Writing the API integration code itself → stub it and hand it to **Claude Code** with the file/pattern/end-state named explicitly.

