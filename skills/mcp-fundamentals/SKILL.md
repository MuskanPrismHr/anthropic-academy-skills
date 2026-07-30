---
name: "mcp-fundamentals"
description: "Reference guide to the Model Context Protocol (MCP): the problem it solves (letting someone else's server own tool/resource/prompt implementations instead of you hand-writing every integration), client/server architecture and transports, the ListTools/CallTool/ReadResource/GetPrompt message flow, building an MCP server with the Python FastMCP SDK (tools via decorators, direct vs. templated resources, prompts), testing with the MCP Inspector, implementing the client side (list_tools/call_tool/read_resource/list_prompts/get_prompt), and the three server primitives explained by who controls them (tools = model-controlled, resources = app-controlled, prompts = user-controlled) with a decision guide for which to use. Use whenever the user asks about MCP, building or debugging an MCP server or client, tool/resource/prompt design in MCP, or how MCP relates to tool use."
---

## Purpose

Condensed reference to Anthropic's "Introduction to Model Context Protocol" course — a hands-on build of both sides of an MCP integration (a document-management server and a CLI client) using the Python SDK. Complements `building-agents-with-the-api`, which covers the same SDK mechanics as one section among many; this skill goes deeper on MCP specifically. Pull the relevant section rather than repeating the whole thing.

## The problem MCP solves

Claude can be given tools, but authoring, testing, and maintaining tool schemas and implementations for a large external service (GitHub, Slack, a database) yourself is a lot of ongoing work. If a user asks "what open pull requests are there across all my repositories," answering that well means covering a large surface area of GitHub's API — every endpoint you want Claude to reach needs its own hand-written tool.

MCP shifts that burden to whoever publishes the server. An **MCP server** wraps up a service's functionality and exposes it as a standardized set of tools (and resources and prompts); your application becomes an **MCP client** that connects to it instead of implementing everything from scratch. Anyone can author an MCP server — often the service provider itself ships an official one.

**MCP vs. tool use — not the same thing.** Tool use is the general mechanism by which Claude calls a function and gets a result back. MCP is a way of getting tools (and resources and prompts) *already implemented for you* by someone else, rather than writing the schema and the function yourself. The two are complementary: an MCP server's tools still reach Claude through ordinary tool use.

## Client/server architecture

An **MCP client** is the communication bridge in your application — your access point to whatever an MCP server provides. It handles message exchange and protocol details so your application code doesn't have to.

**Transport agnostic.** Client and server can communicate over different transports depending on setup: most commonly stdio (both processes on the same machine), but also HTTP, WebSockets, or other network protocols.

**Core message types:**
- `ListToolsRequest` / `ListToolsResult` — "what tools do you provide?"
- `CallToolRequest` / `CallToolResult` — "run this tool with these arguments, give me the result"
- `ReadResourceRequest` / `ReadResourceResult` — "give me this resource's data"
- Prompt equivalents for listing and fetching prompts (see below)

**The full request flow**, using "what repositories do I have?" as an example: user query reaches your server → your server asks the MCP client for available tools → the client sends `ListToolsRequest` to the MCP server and gets back `ListToolsResult` → your server sends the query plus those tools to Claude → Claude decides to call a tool → your server asks the client to run it → the client sends `CallToolRequest` to the MCP server, which makes the real external API call → the result flows back as `CallToolResult` → your server passes it to Claude → Claude's final answer goes back to the user. Many steps, but each component has one clear job — the client abstracts server communication, letting your application logic stay focused on orchestration.

**In most real projects you build either a client or a server, not both** — you're either exposing your own service to other MCP consumers, or consuming someone else's server. Building both together (as in a learning project) is purely to see the full round trip.

## Building an MCP server (Python SDK)

The official Python SDK (`FastMCP`) turns server creation into decorators over plain functions instead of hand-written JSON schemas.

```python
from mcp.server.fastmcp import FastMCP
mcp = FastMCP("DocumentMCP", log_level="ERROR")
```

### Tools — model-controlled

Decorate a function with `@mcp.tool(name=..., description=...)`. Python type hints plus Pydantic `Field(description=...)` on each parameter let the SDK auto-generate the schema Claude sees — no manual JSON schema writing.

```python
@mcp.tool(
    name="read_doc_contents",
    description="Read the contents of a document and return it as a string."
)
def read_document(
    doc_id: str = Field(description="Id of the document to read")
):
    if doc_id not in docs:
        raise ValueError(f"Doc with id {doc_id} not found")
    return docs[doc_id]
```

Raise ordinary Python exceptions for invalid input (a missing document id, say) — error handling integrates naturally, and Claude can read the error text and often retry with corrected arguments. A second example tool (`edit_document`, doing a find-and-replace) shows the same pattern extends cleanly to multi-parameter tools.

**Benefits of the decorator approach:** no manual schema writing, type hints give automatic validation, Field descriptions communicate parameter intent to Claude, and tool registration happens automatically — add a function, get a tool.

### Resources — app-controlled

Resources expose data for your application to fetch, conceptually closer to a GET handler than an action — used when you need to *fetch* information rather than perform one. A common use case: an "@mention a document" feature, where selecting a resource injects its content straight into the prompt instead of making Claude spend a tool call to go get it.

Resources follow a request/response pattern: your client sends a `ReadResourceRequest` with a URI identifying what it wants, the server runs the matching function, and returns a `ReadResourceResult`.

Two shapes:
- **Direct resources** — a static URI, for parameter-free lookups: `@mcp.resource("docs://documents", mime_type="application/json")` returning, say, a list of every document id.
- **Templated resources** — a URI with a `{parameter}` placeholder; the SDK parses it straight into your function's keyword argument: `@mcp.resource("docs://documents/{doc_id}", mime_type="text/plain")` taking `doc_id: str`.

Set `mime_type` to tell clients how to interpret the payload (`application/json`, `text/plain`, `application/pdf`, etc.) — the SDK serializes your return value automatically, so return the actual data structure rather than manually converting to a JSON string.

### Prompts — user-controlled

Prompts are pre-built, tested instruction templates the server author ships, so users get an expert-crafted prompt instead of writing their own from scratch. Users *can* already ask Claude to do most things directly ("reformat report.pdf in markdown"), but a prompt you've deliberately written, tested, and refined for edge cases will reliably outperform an ad hoc request — and you only have to get it right once.

```python
@mcp.prompt(
    name="format",
    description="Rewrites the contents of the document in Markdown format."
)
def format_document(
    doc_id: str = Field(description="Id of the document to format")
) -> list[base.Message]:
    prompt = f"""Your goal is to reformat a document to be written with markdown syntax.
    The id of the document you need to reformat is: {doc_id}
    Add in headers, bullet points, tables, etc as necessary...
    Use the 'edit_document' tool to edit the document."""
    return [base.UserMessage(prompt)]
```

The function returns a list of `base.Message` objects (user and/or assistant messages) — you can build multi-turn seed conversations, not just a single instruction. Benefits: consistency (every user gets the same well-tested result), encoded expertise (your domain knowledge lives in the prompt, not in each user's head), reusability across client applications, and centralized maintenance (fix it once, every client benefits).

### Testing with the MCP Inspector

Run `mcp dev mcp_server.py` (or `uv run mcp dev mcp_server.py`) to start a local dev server with a browser-based Inspector (typically at `http://127.0.0.1:6274`). Click **Connect** first, then use the Tools / Resources / Resource Templates / Prompts tabs to list and directly invoke anything your server exposes — see the exact request/response, including how prompt variables get interpolated, before any real client touches it. State persists between calls within a session, so you can, e.g., edit a document with one tool call and immediately read it back with another to confirm the change stuck. This tight feedback loop is the standard way to iterate on a server without wiring up a full client first.

## Building the MCP client (Python SDK)

The client has two parts: a `ClientSession` (the actual connection, provided by the SDK) and your own thin wrapper class around it that handles setup/cleanup and exposes simple methods to the rest of your app.

```python
async def list_tools(self) -> list[types.Tool]:
    result = await self.session().list_tools()
    return result.tools

async def call_tool(self, tool_name: str, tool_input: dict) -> types.CallToolResult | None:
    return await self.session().call_tool(tool_name, tool_input)
```

**Reading resources** requires checking the MIME type to decide how to parse the payload:

```python
async def read_resource(self, uri: str) -> Any:
    result = await self.session().read_resource(AnyUrl(uri))
    resource = result.contents[0]
    if isinstance(resource, types.TextResourceContents):
        if resource.mimeType == "application/json":
            return json.loads(resource.text)
    return resource.text
```

**Prompts** mirror the tools pattern — list them, then fetch one with arguments that get interpolated server-side:

```python
async def list_prompts(self) -> list[types.Prompt]:
    result = await self.session().list_prompts()
    return result.prompts

async def get_prompt(self, prompt_name, args: dict[str, str]):
    result = await self.session().get_prompt(prompt_name, args)
    return result.messages
```

Once these methods exist, the rest of your application treats them like any other tool/data source: fetch the tool list once, hand it to Claude alongside the user's message, execute whichever tools Claude calls, feed results back — the MCP layer just means you didn't have to author the schemas or implementations yourself. A typical CLI wires this up so `@` triggers resource autocomplete and `/` triggers prompt commands, with the underlying document content or prompt template fetched through the client transparently.

## The three primitives, and who controls each

The single most useful mental model for deciding which primitive to reach for is: **each one is controlled by a different layer of your stack.**

| Primitive | Controlled by | Used for |
|---|---|---|
| **Tools** | The model (Claude) | Giving Claude new capabilities it decides to invoke autonomously — e.g. Claude deciding to run a calculation or fetch data to answer a question |
| **Resources** | Your application code | Getting data into your app for UI (autocomplete lists) or for augmenting a prompt with context — your code decides when to fetch and inject, not Claude |
| **Prompts** | The user | Predefined, optimized workflows a user triggers on demand — a slash command, a menu click, a button |

Anthropic's own products illustrate all three: the workflow buttons under a chat input are prompts, an "Add from Google Drive" picker is resources in action, and Claude autonomously running code or a calculation is tools.

**Decision guide:**
- Need to give Claude a new autonomous capability? → **tool**
- Need to get data into your app for UI or to enrich context without Claude spending a tool call? → **resource**
- Want to hand users a one-click, pre-tested workflow? → **prompt**

## Quick reference

- Facing a large external service with too much surface area to hand-write every tool → look for an existing **MCP server** for that service before building your own tool integrations.
- Building your own integration for others to consume → build an **MCP server**; consuming someone else's → build an **MCP client**. Rarely both in a real project.
- Need Claude to call a function autonomously → **tool**, defined with `@mcp.tool`, tested via the **Inspector** (`mcp dev`).
- Need to fetch data for your app's UI or to inject context without a tool round-trip → **resource** — static URI for parameter-free lookups, templated URI (`{param}`) when you need arguments.
- Need to ship a pre-tested, reusable instruction workflow to users → **prompt**, returning a list of seed messages, triggered by a client-side action like a slash command.
- Building the client side → wrap `ClientSession` in your own class exposing `list_tools`/`call_tool`/`read_resource`/`list_prompts`/`get_prompt`; check MIME type when parsing resource content.
- Confused about MCP vs. tool use → MCP is *who wrote the tool*, tool use is *how Claude calls it* — MCP servers deliver already-implemented tools (and resources and prompts) so you don't have to write them yourself.

