---
name: "building-agents-with-the-api"
description: "Reference for building directly on Anthropic's Messages API: request anatomy, multi-turn state, system prompts, temperature, streaming, forced structured output via prefill+stop sequences, prompt evaluation (datasets, model/code graders), prompt engineering technique, custom tool use end-to-end (schemas, message blocks, multi-turn loops, text-editor and web-search built-in tools), RAG (chunking, embeddings, hybrid BM25+vector search), extended thinking, image/PDF input, citations, prompt caching, code execution + Files API, building an MCP client and server, and agent design patterns (workflows vs agents, parallelization, chaining, routing, environment inspection). Use when writing code against the API/SDK, debugging a tool-use loop, designing RAG, setting up evals, choosing workflow vs agent, or building MCP servers/clients."
---

## Purpose

Condensed reference to Anthropic's "Building with the Claude API" course — a from-scratch, hands-on walkthrough of the Messages API using the Python SDK. This is the most implementation-level of the API-focused courses: where `agent-platform-101` covers the platform's breadth (Managed Agents, Skills, MCP as a hosted feature, etc.), this skill covers the mechanics of writing the request/response loop, tool-use loop, RAG pipeline, and MCP client/server yourself. Pull the relevant section rather than repeating the whole thing.

## Request basics

**Setup:** get an API key from the Anthropic Console (Get API Keys → Create Key), store it in a `.env` file (never hardcode it or commit it), install the SDK (`anthropic` for Python), then construct a client and call `client.messages.create(...)`.

**Every request needs three things:** `model`, `max_tokens` (a hard ceiling on response length — Claude doesn't try to reach it, it's a safety cap, not a target), and `messages` (a list of `{role, content}` dicts, role is `user` or `assistant`). The response's generated text lives at `message.content[0].text`.

**The five-step request lifecycle**, useful for debugging: your client → your server (never call the API directly from client-side code — that exposes your secret key) → your server → the Anthropic API → back the same way. Internally Claude processes a request in four stages: tokenization (text → tokens), embedding (token → a vector encoding possible meanings), contextualization (the embedding gets refined based on surrounding tokens to resolve which meaning applies), and generation (an output layer computes next-token probabilities and samples from them, repeating token by token). Generation stops when it hits `max_tokens`, produces a natural end-of-sequence token, or hits a configured stop sequence — check `stop_reason` in the response to know which.

## Multi-turn conversations

**Claude is stateless — it has no memory of previous requests.** To maintain a conversation, you keep your own list of messages and resend the *entire* history with every call; there's no server-side session. The standard pattern is a couple of small helpers: one to append a user message, one to append an assistant message (using the model's own prior response), and a `chat()` wrapper around `messages.create` that takes the running list. Add the model's response back into your list as an `assistant` message before appending the next user turn — skip this and follow-ups like "write another sentence" will confuse Claude, since it has nothing to refer back to.

## System prompts

A `system` string shapes role, tone, and behavioral constraints across the whole conversation — e.g., "you are a patient math tutor, guide step by step, never give the direct answer" measurably changes behavior compared to no system prompt at all. Practical gotcha: the API rejects `system=None`, so a reusable `chat()` helper needs to conditionally include the `system` key only when one is actually provided.

## Temperature

A 0–1 dial on how deterministic vs. varied token selection is. At `0`, Claude essentially always picks the highest-probability token (repeatable, safe for facts/code/extraction). At `1`, probability mass spreads out more, producing more varied, creative output. It doesn't *guarantee* different outputs each time, just changes the odds. Rough bands: **0.0–0.3** for factual/coding/extraction/moderation work, **0.4–0.7** for summarization, explanation, constrained creative writing, **0.8–1.0** for brainstorming, marketing copy, jokes, open-ended creative writing.

## Streaming

Full responses can take 10–30 seconds — streaming avoids a blank loading spinner by surfacing text as it's generated. Enable with `stream=True`; you'll receive a sequence of events (`MessageStart`, `ContentBlockStart`, `ContentBlockDelta` — this is where the actual text chunks live, `ContentBlockStop`, `MessageDelta`, `MessageStop`). Rather than hand-parsing these, the SDK's simplified interface (`client.messages.stream(...)` with a `.text_stream` you iterate over) gives you just the text; call `.get_final_message()` afterward if you also need the complete assembled message object for storage.

## Forcing structured/raw output

Claude defaults to being helpful and wrapping generated JSON/code in explanatory prose and markdown fences — a problem when a downstream system expects to copy-paste raw output. The fix is **assistant message prefilling + a stop sequence**: add an assistant turn that already starts the code fence (e.g. ` ```json `), then set that same fence as a `stop_sequence`. Claude, believing it already opened the block, just writes the content and generation halts the instant it tries to close the fence — leaving you clean, parseable output with no wrapper text. The same trick generalizes to any format Claude naturally wraps in delimiters (code blocks, lists, etc.) — identify the wrapper, prefill it, stop on it.

## Prompt evaluation

Writing a prompt is the easy part; knowing whether it actually holds up under real usage is the harder, more important part. Testing it once or tweaking it for a corner case or two is a common trap — real users generate inputs you didn't anticipate. The alternative is a real **eval pipeline**: objective, repeatable scoring you can compare across prompt versions.

**Five-step workflow:**
1. **Draft a prompt** — a template with variables to interpolate.
2. **Build an eval dataset** — representative sample inputs, generated by hand or (efficiently) by asking a fast/cheap model like Haiku to generate them for you, saved to a file for reuse.
3. **Run every dataset item through the prompt** to get Claude's outputs.
4. **Grade each output** — typically on a 1–10 scale.
5. **Change the prompt and rerun** — compare the new average score to the old one. Repeat.

**Grader types:**
- **Code graders** — deterministic, programmatic checks: does it parse as valid JSON/Python/regex, does it hit a length constraint, does it contain/avoid certain terms. Fast, free, and exact for anything mechanically checkable.
- **Model graders** — a second Claude call evaluates the first call's output against your criteria and returns a score plus reasoning. Ask explicitly for strengths/weaknesses/reasoning alongside the score — without that scaffolding, model graders drift toward a lazy, uninformative middling score (around 6) regardless of actual quality.
- **Human graders** — most flexible, most expensive; reserve for qualities that are genuinely hard to specify (depth, relevance, overall polish).

For code-generation-style tasks, combine a **code grader** (does it parse, is the format right — deterministic and cheap) with a **model grader** (does it actually solve the task — needs judgment), and merge the two scores (e.g., a simple average, or a weighted one if one dimension matters more for your use case).

**Practical build tips:** keep your eval dataset small (2-3 cases) while iterating quickly on the prompt itself, then scale it up for final validation. Don't be discouraged by a low first score (a 2-3/10 on a naive first prompt is normal) — the point of the pipeline is to make *improvement* measurable, not to nail it in one try.

## Prompt engineering technique

A companion skill to evaluation: once you can *measure* a prompt's quality, these are the levers that reliably move the score.

**Be clear and direct.** Your first line sets the frame for everything after it — lead with a plain, unambiguous statement of the task using an action verb ("Generate," "Write," "Identify"), not a roundabout question. In one measured example, simply rewriting a vague opening line into a direct instruction raised an eval score substantially — first-line clarity has an outsized effect.

**Be specific**, in two complementary ways:
- **Output guidelines** — an explicit list of qualities the response must have (length, structure, required elements, tone). Use these in nearly every prompt; they're your baseline consistency net, and adding a concrete checklist can more than double an eval score over a "be helpful" prompt with no guidance.
- **Process steps** — explicit intermediate steps for Claude to work through, especially for troubleshooting, multi-factor decisions, or anything where you'd want a human to consider several angles before concluding. Layer both together for complex professional tasks: guidelines control the shape of the output, steps control the reasoning that gets you there.

**Structure with XML tags** when a prompt mixes large or varied content (data plus instructions, code plus documentation) — wrapping each chunk in a descriptive custom tag (`<sales_records>`, `<athlete_information>`, `<my_code>`, `<docs>` — they don't need to be "real" XML, just consistent and descriptive) gives Claude unambiguous boundaries between what's instruction and what's data. Matters more as prompts grow longer and more varied; low payoff on short simple prompts.

**Provide examples (one-shot / multi-shot).** Show, don't just tell — especially for corner cases language alone struggles to pin down (e.g., sarcasm in sentiment classification). Wrap each example in tags like `<sample_input>`/`<ideal_output>`, and where useful, add a sentence explaining *why* that output is good — this teaches the underlying reasoning, not just the surface pattern. A great source of examples: your own eval results — pull the highest-scoring outputs from a prior run and feed them back in as the model of "good."

**The iteration loop that ties it together:** set a goal → write a first pass → evaluate → apply one technique → re-evaluate → repeat, changing one variable at a time so you actually know what moved the score.

## Tool use

Claude's training data has a cutoff and no access to your systems by default. Tools close that gap: you describe a function (name, description, JSON schema for its inputs), Claude decides when it's needed and asks you to run it — **Claude never executes anything itself; your code does.**

**The full loop:**
1. Send a user message plus a `tools` array of schemas.
2. Claude replies with a multi-block message — often a text block ("let me check that for you") *and* a `tool_use` block (function name, input args, and an id).
3. Your code runs the matching function.
4. You send the result back as a `tool_result` block, inside a `user` message, with `tool_use_id` matching the original call's id — the API needs that id round-tripped exactly to stitch the exchange together.
5. Claude incorporates the result and responds — with either final text or another tool call, hence a loop, not a fixed number of turns.

**Writing tool functions well:** descriptive function and parameter names, input validation with clear errors (Claude reads error text and will often retry with corrected arguments — a validation error is a signal, not just a failure), and a matching JSON schema with `name`, a 3-4 sentence `description` covering what it does / when to use it / what it returns, and an `input_schema`. You can hand your function code to Claude itself and ask it to draft the JSON schema for you rather than writing it by hand.

**Detecting a tool request:** check `response.stop_reason == "tool_use"` — that's your signal to extract every `tool_use` block from `response.content` (there can be more than one in a single turn), execute each, and wrap each result as a `tool_result`. Preserve the *entire* multi-block content list when appending Claude's message back into history — dropping the tool_use block breaks the conversation's ability to resolve the matching tool_result.

**The conversation loop for multi-turn tool use:**
```python
while True:
    response = chat(messages, tools=tools)
    add_assistant_message(messages, response)
    if response.stop_reason != "tool_use":
        break
    tool_results = run_tools(response)   # execute every tool_use block, wrap as tool_result
    add_user_message(messages, tool_results)
```
Route tool names to implementations with a simple dispatch (`if/elif` or a dict lookup); adding a new tool is just: write the function, write its schema, add it to the `tools` array, add a dispatch case. Wrap execution in try/except and return an `is_error: true` tool_result with the error text on failure — Claude can often recover from this and retry sensibly.

**Streaming tool calls.** With plain streaming, tool-argument JSON arrives in validated bursts — the API buffers a tool's arguments until a complete, schema-valid top-level key is ready before releasing it, which is why you see delay-then-burst behavior even while streaming. **Fine-grained tool calling** (`fine_grained=True`) disables that server-side validation so you get raw chunks the instant Claude generates them — more responsive, but now *your* code must tolerate genuinely invalid partial JSON (wrap parsing in try/except) since the API's safety net is gone. Default (validated) behavior is right for most apps; fine-grained is for when you specifically need to render partial tool-argument progress to a user in real time.

**Built-in tools that ship with Claude (you don't write the schema or, in some cases, the implementation):**
- **Text editor tool** — gives Claude file-manipulation verbs (view, view a line range, replace text, create a file, insert at a line, undo). You still write the implementation functions that actually touch disk; Claude just knows the calling convention. Include a small version-specific schema stub (varies by model generation) and Claude expands it into the full spec internally.
- **Web search tool** — fully server-side: you supply a schema (`type`, `name`, `max_uses` to cap how many searches a single turn can trigger) and Claude runs the searches itself; no implementation needed from you. Response includes search-result blocks and citation blocks pointing at supporting text and source URLs. Restrict to trusted domains with `allowed_domains` (e.g., `["nih.gov"]` for medical questions) when source authority matters.

## RAG (Retrieval Augmented Generation)

**The problem:** documents too large to fit in a prompt (an 800-page filing, a large knowledge base). Stuffing everything in raises cost, latency, and dilutes Claude's focus, and eventually just doesn't fit. RAG's answer: chunk documents ahead of time, then at query time retrieve only the chunks relevant to the actual question and include just those.

**Chunking strategies**, in rough order of effort vs. quality:
- **Size-based** — split into fixed-length windows, typically with overlap so words/sentences don't get severed at a boundary and so context bleeds slightly across chunk edges. Simplest, most robust fallback, works on any content including code — the default choice for production when you don't have strong structural guarantees.
- **Structure-based** — split on document structure (markdown headers, section boundaries). Produces the cleanest, most semantically whole chunks, but only works when you can rely on that structure existing.
- **Sentence-based** — split into sentences, then group N at a time with a small overlap. A reasonable middle ground for plain-text documents without reliable structure.
- **Semantic-based** — group sentences by how related consecutive ones are, using NLP similarity. Most accurate, most expensive to compute, most complex to implement.

Bad chunking has real failure modes: a chunk that mixes unrelated topics can get retrieved for the wrong reason entirely (e.g., a "bug" in a medical-research sense getting pulled in for a software-engineering question about literal bugs) purely on word overlap.

**Embeddings and semantic search.** A text embedding is a numeric vector representing a chunk's meaning — Anthropic doesn't provide an embedding endpoint itself; the course uses **VoyageAI** as the provider. You embed every chunk once (offline, ahead of time) and store the vectors in a vector index/database alongside the original chunk text (you need the text back, not just numbers, once you've found a match). At query time you embed the user's question the same way and ask the store for the closest stored vectors.

**Similarity metric: cosine similarity** — the cosine of the angle between two vectors, ranging from -1 (opposite) to 1 (identical direction); values near 1 mean high semantic similarity. **Cosine distance** (`1 - similarity`) is the same idea inverted so smaller numbers mean "closer" — the convention most vector-store APIs actually return.

**The full pipeline, end to end:** chunk the source → embed every chunk → store (embedding, original text) pairs in a vector index → [wait for a real user query] → embed the query with the same model → search the index for nearest neighbors → merge the top chunk(s) with the user's question into a final prompt → send to Claude.

**Semantic search's blind spot: exact terms.** A specific identifier (an incident ID, a product SKU) can get buried under conceptually-similar-but-wrong content, because semantic search optimizes for meaning, not literal string presence. The fix is **hybrid search**: run semantic (vector) search and **BM25** lexical search (a classic term-frequency algorithm that weights rare, specific terms — like that incident ID — far higher than common words) in parallel, then merge the two ranked result lists.

**Merging two rankings: Reciprocal Rank Fusion (RRF).** For each result, sum `1 / (k + rank)` across every ranking it appeared in (a small constant `k`, often 60, dampens the effect of rank-1 dominance), then re-sort by that combined score. A chunk that ranks reasonably well in *both* semantic and lexical search rises above one that ranks #1 in only one of them — which is usually what you actually want. Structuring both your vector index and your BM25 index behind the same `add_document()` / `search()` interface makes it easy to wrap them in a shared `Retriever` that fans a query out to all indexes and fuses the results — and just as easy to add a third index type later without touching the fusion logic.

## Features of Claude

**Extended thinking.** Lets Claude reason through a problem step by step before answering, and — unlike a black-box internal process — that reasoning is returned as a visible block alongside the final answer. Costs more (you pay for thinking tokens) and adds latency, so the right call is empirical: run your prompt evals without it first, and only turn it on if accuracy still isn't where you need it after prompt optimization. On newer Opus models thinking is adaptive — you don't set a token budget, you set an `effort` level instead and Claude decides how much to think. On other models you set a `thinking_budget` (minimum ~1024 tokens; `max_tokens` must exceed it). Thinking responses are cryptographically signed so the reasoning can't be tampered with between turns; occasionally you'll get a **redacted thinking** block (safety-system-flagged, encrypted) — pass it back unmodified in subsequent turns rather than trying to strip or read it. Not compatible with some other features (e.g., prefilling, temperature) — check current docs before combining.

**Images.** Include an `image` content block (base64 or URL) alongside a `text` block in a user message. Limits: up to 100 images per request, 5MB per image, and a resolution cap that's stricter when sending multiple images at once than a single one; token cost scales with pixel count. The single biggest accuracy lever for visual tasks is the same as for text: don't just ask a bare question — give Claude an explicit step-by-step methodology (e.g., "identify each item individually and number it, then verify by re-counting a different way") and/or a worked one-shot example, because a naive one-line prompt on a nontrivial visual task (counting, detailed structured assessment) is unreliable.

**PDFs.** Nearly identical to image handling — swap the block `type` to `"document"`, the media type to `application/pdf`. Claude can extract from and reason over text, embedded charts/images, and tables in the same pass, not just raw text.

**Citations.** Add `"title"` and `"citations": {"enabled": true}` to a document block, and Claude's response includes structured citation objects for each claim: the exact `cited_text`, which document it came from (`document_index`/`document_title` when you supply several), and a location (page range for PDFs, character offsets for plain text). This turns Claude from an opaque answer-generator into something a user can verify — worth it whenever answers need to be checked against source material or transparency matters to trust. Works for plain-text sources too, not just PDFs.

**Prompt caching.** Normally every request's tokenization/embedding/context work is thrown away after the response — wasteful if you're repeatedly sending largely-identical content (a long system prompt, a big tool schema set, a large reference document across several follow-up questions). Caching persists that preprocessing for **one hour** so a follow-up request with matching content reads from cache instead of reprocessing.

Rules that matter:
- Caching is **opt-in per block** via an explicit `cache_control: {"type": "ephemeral"}` breakpoint — nothing is cached automatically, and you need the longhand block form (not the string shorthand) to attach it.
- Everything **up to and including** a breakpoint gets cached; a follow-up only hits the cache if the content up to that point is byte-for-byte identical — even inserting one word invalidates it.
- Breakpoints can sit in system prompts, tool definitions, text blocks, image blocks, or tool result blocks — up to **4 per request**.
- Processing order is fixed: **tools → system prompt → messages** — place breakpoints with that order in mind (e.g., one after your stable tool definitions, another after a stable system prompt, so a change to messages alone doesn't blow away tool/system caching).
- Minimum cacheable size is **1024 tokens** total across what you're trying to cache — a short message won't qualify no matter how you tag it.
- Response `usage` tells you what happened: `cache_creation_input_tokens` (this request wrote to cache) vs. `cache_read_input_tokens` (this request read from cache) — watch these to confirm caching is actually engaging as expected.

Best candidates: stable system prompts and stable tool schemas (they rarely change between calls and are often the largest static chunk of a request) — that's usually where caching pays off the most.

**Code execution + the Files API.** The **Files API** lets you upload a file once (image, PDF, CSV, etc.) and get back a file ID you reference in later messages, instead of re-embedding raw base64 data in every request — useful for large files or ones you'll reference repeatedly. **Code execution** is a server-side tool: you supply a schema, no implementation, and Claude can write and run Python inside an isolated, network-disconnected Docker container, iterating (running code more than once) before it settles on a final answer. Because the sandbox has no network access, the Files API becomes the practical way to move data in (upload a CSV, reference it via a `container_upload` block) and out (Claude-generated files, like a plot, come back with their own file ID you download). The response contains a mix of text, `server_tool_use` blocks (the code Claude actually ran), and result blocks (captured stdout/output) — Claude may loop through several execute-and-inspect cycles before its final text answer. Good fits beyond data analysis: image/document processing, numeric modeling, formatted report generation — anywhere you'd want to hand Claude an actual computation to run rather than just reason about in text.

## Model Context Protocol (MCP) — building your own client and server

**The problem MCP solves:** if you want Claude to work with an external service (GitHub, Slack, a document store), the naive path is writing and maintaining a tool schema + implementation for every single capability yourself — and re-maintaining it every time that service's API changes. MCP shifts that maintenance burden to whoever publishes the MCP server: they write and keep the tool definitions current, you just connect to them. It's easy to conflate "MCP" with "tool use" — they're complementary, not the same thing: tool use is the general mechanism, MCP is a way of getting tool implementations from someone else instead of writing them yourself.

**Architecture:** an **MCP client** (your server/app) connects to one or more **MCP servers**, each acting as a wrapper around some outside capability. Communication is transport-agnostic — commonly stdio when client and server run on the same machine, or HTTP/WebSockets across a network. Core message types: `ListToolsRequest`/`Result` (what tools does this server offer) and `CallToolRequest`/`Result` (run this tool with these arguments, give me the result).

**In most real projects you build either a client or a server, not both** — you're either exposing your own service to other MCP consumers, or consuming someone else's server. Building both together (as in a learning project) is purely to see the full round trip.

**Building an MCP server with the Python SDK** (`FastMCP`) turns what would be verbose hand-written JSON schemas into simple decorators over plain Python functions:

- **Tools** (`@mcp.tool(name=..., description=...)`) — actions. Type hints plus Pydantic `Field(description=...)` on each parameter auto-generate the JSON schema; you just write the function body and raise clear errors for invalid input (e.g., an unknown document id).
- **Resources** (`@mcp.resource("your://uri", mime_type=...)`) — data to fetch, conceptually closer to a GET endpoint than an action. Direct resources have a static URI (`docs://documents` → list every doc); templated resources have a parameterized URI (`docs://documents/{doc_id}` → fetch one), and the SDK parses the URI parameter straight into your function's keyword argument. Rule of thumb: **tools do things, resources fetch things** — and resources are how you get document content directly into a prompt without the extra round trip of a tool call.
- **Prompts** (`@mcp.prompt(name=..., description=...)`) — pre-built, tested message templates the server author ships so a client doesn't have to hand-roll good instructions for common tasks (e.g., a "reformat this document as markdown" prompt tuned specifically for this server's tools). Returns a list of `base.Message` objects ready to feed into a `messages.create` call.

**Debugging without wiring up a full app:** run `mcp dev your_server.py` to launch the browser-based **MCP Inspector**, connect to your server, and directly list/run tools, list/fetch resources, and list/fill-in prompts to verify behavior in isolation before integrating a real client.

**Implementing the client side** means wrapping an SDK `ClientSession` in your own thin class with a handful of methods that mirror the message types: `list_tools()`, `call_tool(name, input)`, `read_resource(uri)` (parse the returned content by its declared MIME type — e.g. `json.loads` for `application/json`, plain text otherwise), `list_prompts()`, and `get_prompt(name, args)` (returns ready-to-send messages with your arguments already interpolated). Your application logic then treats these exactly like any other tool source: fetch the available tools once, hand them to Claude alongside the user's message, execute whichever ones Claude calls, and feed results back — the MCP layer just means you didn't have to author the tool schemas or implementations yourself.

## Anthropic's own apps as agent case studies

**Claude Code**, used here as a worked example of an agent built on generic, composable tools rather than task-specific ones (`bash`, `read`, `write`, `edit`, `glob`, `grep` — notably *not* specialized verbs like "refactor" or "install dependency"). Setup: install Node.js, `npm install -g @anthropic-ai/claude-code`, run `claude` and log in.

**`/init`** scans the current codebase and writes a `CLAUDE.md` summarizing structure, dependencies, and conventions, which then auto-loads as context in every future session in that project — with separate scopes for project (shared, checked into git), local (personal, git-ignored), and user (applies across all your projects) memory. The `#` shortcut quickly appends a new rule to the right one of these without leaving your current flow.

**Effective workflow pattern:** feed relevant existing files as context first (so Claude has real examples of your patterns to work from) → ask for a plan with an explicit instruction *not* to write code yet → review/adjust the plan → then ask for implementation. A **test-driven variant** inserts test-case brainstorming and test-writing between planning and implementation, giving Claude concrete pass/fail criteria to iterate against rather than a vaguer sense of "done."

**MCP servers extend Claude Code** past its built-in tools — register one with `claude mcp add <name> <command-to-start-server>`, after which it connects automatically on startup. Common integrations mentioned: Sentry (bug triage), Playwright (browser automation for testing), Figma (design context), Atlassian (Jira/Confluence), Firecrawl (web scraping), Slack (posting/replying). The practical payoff is composing several of these into a workflow matching your actual dev process — e.g., pull an error from Sentry, cross-reference the ticket in Jira, post a completion notice to Slack, all from one Claude Code session.

## Agent design patterns: workflows vs. agents

**Workflows** are a predetermined sequence of Claude calls (plus, optionally, ordinary code) built for a task whose steps you can already picture — you know the shape, so you hand-write the pipeline. **Agents** hand Claude a goal and a toolbox and let it work out the sequence itself — the right call when you *can't* fully predict what steps a given request will need.

**Trade-offs**, worth weighing explicitly per feature rather than picking a paradigm as a house style:
- Workflows: more predictable, far easier to test and evaluate (you know the exact steps to check), generally higher per-task accuracy since each step is narrowly focused — at the cost of flexibility and more upfront design effort, and they only handle the shapes of request you explicitly built for.
- Agents: much more flexible, can handle genuinely novel requests and ask clarifying questions on the fly — at the cost of a lower reliable-completion rate and being considerably harder to test, since you don't know in advance what sequence of tool calls a given run will take.

**Default guidance: prefer workflows, reach for agents only when you genuinely can't predict the task shape.** Users care that a product works consistently, not that it's architecturally impressive — reliability usually beats flexibility in production.

**Four workflow patterns worth having as ready-made recipes:**

- **Evaluator-optimizer** — a producer step creates output, a separate grading step checks it against explicit criteria, and failures loop feedback back to the producer for another pass, repeating until the grader accepts it. Good fit whenever "good enough" is checkable but not reliably achievable in a single shot (e.g., generating a 3D model from an image description, then rendering and comparing that render back against the original image).
- **Parallelization** — split one complex, multi-criteria decision into several independent, narrowly-scoped sub-evaluations run concurrently, then aggregate the results into a final call. Beats one giant prompt trying to juggle every criterion at once: each sub-task can be tuned and debugged independently, adding a new criterion is just adding another parallel branch, and the reduced cognitive load per call improves reliability on each piece.
- **Chaining** — break a single complex task into an ordered sequence of narrower Claude calls (optionally with ordinary code in between steps), each focused on doing one thing well rather than juggling many constraints simultaneously. Especially valuable as a fix for the "long prompt with many constraints, some of which Claude still quietly violates" problem: let the first pass produce imperfect output, then run a dedicated second pass whose entire job is enforcing the constraints that got missed — a call focused purely on revision is far more reliable at catching violations than a single call trying to write and self-police at once.
- **Routing** — classify an incoming request into one of several predefined categories with a first, small Claude call, then hand it to a category-specific pipeline (its own prompt, tools, or even its own sub-workflow) rather than one generic prompt trying to handle every category equally well. Works well whenever your request types are genuinely distinct enough to warrant separately-tuned handling (e.g., an "educational" content request needs a different structure than an "entertainment" one) and a lightweight classification step can route reliably.

**Designing tools for agents specifically:** favor **abstract, composable primitives over narrow, task-specific tools** — the Claude Code toolset (`bash`, `read`, `write`, `edit`) is the canonical example: a handful of generic verbs that Claude combines in ways its designers never explicitly anticipated, rather than a combinatorially-growing list of purpose-built ones ("refactor_function," "install_dependency"). The same handful of small tools (get current time, add a duration, set a reminder) chains into surprisingly complex requests on its own — a well-designed agent doesn't need every capability spelled out, just composable building blocks.

**Environment inspection — the pattern easiest to accidentally skip.** Claude can't see the effect of an action unless you explicitly give it a way to check: a screenshot after a UI click, reading a file's current contents before editing it, checking an API response against what was expected. Without this feedback loop Claude is effectively acting blind — it can't tell whether an action succeeded, adapt when something unexpected happened, or know when a task is genuinely finished versus merely attempted. When designing any agent, ask explicitly: *how will Claude know if this particular action worked?* — and make sure there's a tool or instruction that answers that, whether that's a verification script, a screenshot, a re-read of a modified file, or a validation check against the original requirements.

## Quick reference: what to reach for

- Need Claude to remember earlier turns → maintain the full message list yourself and resend it every call; Claude has no memory of its own.
- Need consistent role/behavior across a whole conversation → a **system prompt**.
- Need deterministic vs. varied output → tune **temperature** (low for facts/code, high for brainstorming).
- Response taking too long to start rendering → **streaming**.
- Need raw JSON/code with zero wrapper prose → **prefill the opening fence + a matching stop sequence**.
- Don't know if a prompt actually works beyond your own spot-checks → build an **eval pipeline** (dataset → run → grade → iterate) before shipping.
- Prompt output is inconsistent or missing pieces → apply, in order of typical impact: a **direct first line**, explicit **output guidelines** (+ **process steps** for multi-factor reasoning), **XML-tagged** structure for mixed/large content, and **examples** for edge cases.
- Claude needs live data or to take an action → **custom tool use**; write a specific, checkable description, validate inputs, loop on `stop_reason == "tool_use"`.
- Need web search or arbitrary Python execution without building it yourself → the **built-in web search tool** or **code execution tool**.
- Document too large for one prompt, or need search across many documents → **RAG**: chunk (size-based as the safe default), embed, store, and retrieve at query time; add **BM25 + RRF** if exact terms/IDs matter alongside meaning.
- Prompt optimization has plateaued on a genuinely hard reasoning task → **extended thinking**, decided empirically via evals, not by default.
- Need to prove where an answer came from → **citations** on your document blocks.
- Same large system prompt/tool set/document sent repeatedly within an hour → **prompt caching**, with breakpoints on the stable parts.
- Need Claude to run real computation on a file → **Files API + code execution**.
- Want to consume or expose reusable tools/resources/prompts without hand-writing every integration → **MCP** (build a server to expose your service; build a client to consume someone else's).
- Know the exact steps a task needs → build a **workflow** (chain / parallelize / route / evaluate-optimize as fits the shape). Genuinely can't predict the steps → build an **agent** with abstract, composable tools — and always give it a way to check whether its own actions worked.

