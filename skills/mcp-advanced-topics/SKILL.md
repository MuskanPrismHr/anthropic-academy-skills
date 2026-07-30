---
name: "mcp-advanced-topics"
description: "Reference guide to advanced Model Context Protocol (MCP) features and transports, building on mcp-fundamentals. Covers sampling (letting a server borrow the client's LLM connection instead of integrating its own), log and progress notifications for long-running tools, roots (scoping filesystem access and helping Claude locate files without full paths), the JSON message taxonomy (request/result pairs vs. one-way notifications, client vs. server messages), the stdio transport as the bidirectional baseline, and the StreamableHTTP transport's SSE-based workaround for HTTP's client-initiates-only limitation (session IDs, dual SSE connections, and the stateless_http/json_response flags that trade away sampling, notifications, and subscriptions for horizontal scalability). Use whenever the user is building, deploying, or debugging an MCP server/client beyond the basics — especially sampling, progress reporting, roots, or choosing/debugging a transport for production deployment."
---

## Purpose

Condensed reference to Anthropic Academy's "Model Context Protocol: Advanced Topics" course. Builds directly on `mcp-fundamentals` (tools, resources, prompts, the basic client/server loop) — this skill covers the features and transport-layer mechanics that come up once you're deploying a real MCP server rather than just prototyping one locally. Pull the relevant section rather than repeating the whole thing.

## Sampling — letting the server borrow the client's model connection

**The problem:** an MCP server sometimes needs to generate text (e.g., summarizing research it just gathered), but giving the server its own API key and Claude integration means it owns authentication, cost, and maintenance for every request — a real problem for a *public* server anyone can connect to, since you don't want to be billed for every user's usage.

**Sampling's fix:** the server does its own work, builds a prompt, and asks the *client* — which already has a model connection — to run the generation on its behalf via `create_message`. The client pays for the tokens; the server never needs credentials.

**Server side** — inside a tool function, call `ctx.session.create_message(messages=[...], max_tokens=..., system_prompt=...)` and read `result.content.text` back.

**Client side** — implement a `sampling_callback` that receives the server's request, calls your own model client, and returns a `CreateMessageResult`; pass it into `ClientSession(..., sampling_callback=sampling_callback)`.

**When to reach for it:** any public-facing or multi-tenant MCP server where you don't want to absorb everyone's AI costs and don't want to manage API credentials server-side — cost and complexity both shift to whoever's already paying for the model connection.

## Log and progress notifications — UX for long-running tools

Without any signal, a tool call that takes a while just looks frozen to the end user. The `Context` object automatically passed into an MCP tool function gives you two calls to fix that:

- `await context.info("message")` — send a log message to the client.
- `await context.report_progress(current, total)` — update progress.

**Client side:** provide a `logging_callback` at session creation (receives `LoggingMessageNotificationParams`) and a `progress_callback` per tool call (receives progress/total/message) — the server just emits notifications; your client decides how to render them (print to a terminal, update a progress bar, push over WebSockets to a browser, etc.).

**This is entirely optional** — skip it, show only some notification types, or present them however fits your application. It's a UX layer, not a functional requirement.

## Roots — scoped filesystem access without full paths

**The problem:** if a tool takes a file path argument, Claude has no way to search a user's filesystem to find where a file actually lives — requiring users to type full paths every time is bad UX.

**Roots solve this two ways at once:** they tell Claude which directories it's allowed to look in (so it can call `list_roots` then `read_dir` to actually locate a file by name), *and* they establish a security boundary (a server given access only to a Desktop root cannot read files in Documents or Downloads, even if asked).

**Implementation note:** the SDK does **not** enforce root restrictions automatically — you write that yourself, typically an `is_path_allowed()` helper that checks a requested path against the approved root list, called at the top of every tool that touches the filesystem.

**Net effect:** users can say "convert biking.mp4" without a full path, Claude's search is faster because it's scoped, and sensitive directories outside the approved roots stay inaccessible even by accident.

## The JSON message taxonomy

All MCP communication is JSON messages, falling into two shapes:

- **Request-result pairs** — every request expects exactly one matching result: `CallToolRequest`/`Result`, `ListPromptsRequest`/`Result`, `ReadResourceRequest`/`Result`, `InitializeRequest`/`Result`.
- **Notifications** — one-way, no response expected: progress notifications, logging notifications, tool-list-changed, resource-updated.

Messages are also categorized by **who sends them** — client messages (tool calls, and any client-originated notifications) versus server messages (server-initiated requests like sampling or roots listing, and server-originated notifications like progress/logging). The important implication: **MCP is bidirectional** — both sides can initiate communication, not just the client — and that fact is exactly what makes some transports harder to implement than others (see StreamableHTTP below). The canonical, authoritative list of every message type lives in the MCP specification repository (written in TypeScript for clarity of data shapes, not because it runs as TypeScript).

## Transports: stdio as the baseline

A transport is just the channel messages travel over — stdio, HTTP, WebSockets, in principle anything that can carry JSON both ways.

**stdio** is the default for local development: the client launches the server as a subprocess and they talk over stdin/stdout. Either side can write at any time, so all four communication patterns work without friction: client→server request, server→client response, server→client request, client→server response. Because both directions are equally easy, stdio is the *ideal* case — every MCP feature works exactly as specified. You can even test a server directly from a terminal (`uv run server.py`) by piping JSON messages to stdin and reading stdout.

**Every MCP connection, regardless of transport, opens with the same three-message handshake:** `InitializeRequest` (client → server) → `InitializeResult` (server → client, advertises capabilities) → `InitializedNotification` (client → server, no response expected). Nothing else can be sent before this completes.

**Limitation:** stdio only works when client and server run on the same machine — it's a non-starter for a server other people connect to remotely.

## StreamableHTTP — the remote-hosting transport, and its trade-offs

**Why HTTP is harder than stdio:** plain HTTP is built for clients to request and servers to respond — a server has no way to *initiate* a request to a client, because clients don't have stable, reachable addresses the way servers do. That breaks exactly the server-to-client message types MCP needs for sampling, roots requests, and most notifications.

**The workaround: Server-Sent Events (SSE).** After the usual init handshake (during which the server hands back an `mcp-session-id` that must be included on every subsequent request), the client opens a long-lived GET request that becomes an SSE stream — a channel the server can push messages down at any time, effectively faking server-initiated communication over an inherently client-initiated protocol.

**Dual connections per tool call:** a **primary SSE connection** stays open indefinitely and carries progress notifications and other server-initiated messages; a **tool-specific SSE connection** opens per tool call and closes once that tool's result is delivered (logging messages and the tool result itself route through this one).

**The two flags that break the workaround:**
- `stateless_http=True` — removes per-client session tracking entirely. Gains: no initialization handshake required, and (crucially) horizontal scalability — multiple server instances behind a load balancer no longer need to coordinate which instance is holding which client's SSE connection. Losses: no session IDs, no server-to-client requests at all, therefore **no sampling, no progress reports, no subscriptions**.
- `json_response=True` — disables SSE streaming for POST responses specifically; a tool call returns one plain JSON result instead of a stream of intermediate progress/log messages, then the final result.

**Why this matters in practice:** an MCP server that works flawlessly with stdio locally can silently lose functionality the moment it's deployed behind a load balancer with `stateless_http=True` — sampling calls fail, progress bars vanish, logging goes silent. If you must scale horizontally, decide up front whether your server genuinely needs server-initiated features; if it does, the coordination problem (routing a client's POST and GET-SSE connections to the same backend instance, or sharing sampling state across instances) has to be solved some other way, not just flagged around.

**The practical rule:** develop and test with the *same* transport and flags you intend to run in production. "Works locally over stdio" tells you nothing about whether it'll still work once deployed over StreamableHTTP with scaling flags enabled.

## Quick reference

- Server needs to generate text but shouldn't own API keys or absorb model costs → **sampling** (`create_message` server-side, a `sampling_callback` client-side).
- A tool call takes a while and users have no feedback → **log/progress notifications** via the `Context` object; entirely optional, purely UX.
- A tool needs a file by name without the user typing a full path, or you need to scope what a server can touch → **roots**, plus your own `is_path_allowed()` enforcement — the SDK won't do this for you.
- Wondering whether a message needs a response → check whether it's a **request** (paired with a result) or a **notification** (one-way, no response, no `id`).
- Prototyping locally → **stdio**; it's the bidirectional baseline where every MCP feature works without compromise.
- Need a server other people can connect to remotely → **StreamableHTTP**; expect an SSE-based workaround with a required `mcp-session-id` and dual per-call connections.
- Deploying behind a load balancer / need to scale horizontally → `stateless_http=True`, but budget for losing sampling, progress, logging, and subscriptions entirely — there's no free lunch here, only a trade you make deliberately.
- Only need the final tool result, not streamed intermediate messages → `json_response=True`.
- Something works locally but breaks in production over HTTP → check `stateless_http` and `json_response` first; this is the single most common cause.

