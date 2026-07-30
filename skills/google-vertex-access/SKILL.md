---
name: "google-vertex-access"
description: "Use when building applications that call Claude models through Google Cloud Vertex AI instead of the Anthropic API directly - covers enabling Anthropic models in the Vertex Model Garden, gcloud CLI authentication, the AnthropicVertex Python client, Vertex's name@version model ID format, and why request/response shape and tool schemas are unchanged from the plain Anthropic API. Everything about prompting, evals, RAG, prompt caching, and MCP server design is identical regardless of access path and is covered by other skills - this one is scoped to what's different about Vertex specifically."
---

## Purpose

Vertex AI is one of several ways to reach the same Claude models - the model behavior, prompting technique, and tool-use design don't change based on the access path. This skill is scoped to the parts that are genuinely Vertex-specific: enabling model access in Google Cloud, authenticating, and the small client-level differences. For everything else (prompt engineering, evals, RAG, prompt caching, MCP), use a dedicated skill on that topic rather than expecting Vertex-specific guidance here.

Compared to Bedrock, Vertex's delta from the plain Anthropic API is smaller: Vertex uses the same Anthropic Python SDK message format and the same flat tool-schema shape, just through a different client class and a different auth/model-ID convention.

## Enabling model access in Google Cloud

Before writing any code, the target model has to be enabled in the Vertex Model Garden:

1. Go to the Vertex AI dashboard in the Cloud console and open Model Garden.
2. Search for "Anthropic" and select the model you want.
3. If an "Enable" button is present, click it - if it's absent, access is already granted.

Skipping this step is the most common cause of an otherwise-correct request failing outright.

## Authenticating with gcloud

Vertex access is Google Cloud IAM-based, not an API key. Install the gcloud CLI, then:

```bash
gcloud init
gcloud auth login
gcloud config set project YOUR_PROJECT_ID
gcloud auth application-default login
```

The last command sets up Application Default Credentials (ADC). The Anthropic SDK's Vertex client picks these up automatically - there's no separate credential object to construct or pass in.

## The AnthropicVertex client

Install the SDK with the Vertex extra, then construct a Vertex-specific client instead of the default Anthropic client:

```python
%pip install "anthropic[vertex]"

from anthropic import AnthropicVertex

client = AnthropicVertex(region="global", project_id="your-project-id")
model = "claude-sonnet-4@20250514"
```

Two things to get right here: the client class is `AnthropicVertex`, not `Anthropic`, and it needs a `region` and `project_id` rather than an API key. Model IDs on Vertex use an `@version` suffix (`claude-sonnet-4@20250514`) rather than the dated-string format used by the direct API - copy the exact ID from the Model Garden listing rather than guessing at the naming convention.

## Requests and responses are otherwise unchanged

Once the client is set up, everything downstream matches the plain Anthropic Messages API:

```python
message = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=[{"role": "user", "content": "What is quantum computing? Answer in one sentence"}]
)
text = message.content[0].text
```

`messages.create`, the role/content message structure, and pulling text out at `message.content[0].text` are identical to using `Anthropic()` directly - nothing about the request or response shape needs to be relearned. This is the key practical difference from Bedrock, which wraps requests in its own `converse` API and response structure: on Vertex, code written against the standard Anthropic SDK docs works with only the client construction changed.

Tool schemas follow the same pattern: a flat `{"name": ..., "description": ..., "input_schema": {...}}` dict, no extra wrapper layer - a tool schema written for the direct API can be reused on Vertex without reshaping it.

## What doesn't change

Prompt engineering, prompt evaluation workflow, RAG pipeline design, prompt caching rules, extended thinking, multi-turn conversation structure, and MCP server/client implementation are all identical to the direct Anthropic API - the model doesn't know or behave differently based on which cloud delivered the request. Reach for a dedicated skill on that topic instead of expecting Vertex-specific guidance there.

## Quick reference

- Request fails with a model-not-found-style error → check the model is enabled in Vertex's Model Garden before debugging code.
- Setting up auth → `gcloud auth application-default login` after `gcloud init`/`gcloud auth login`; no manual credential object needed.
- Creating a client → `AnthropicVertex(region=..., project_id=...)`, not `Anthropic(api_key=...)`.
- Model ID format → `name@version` (e.g. `claude-sonnet-4@20250514`), pulled from the Model Garden listing.
- Sending a message → `client.messages.create(...)`, identical to the direct API; text is at `message.content[0].text`.
- Defining tools → the standard flat `name`/`description`/`input_schema` shape, no Vertex-specific wrapper.
- Anything about prompting, evals, RAG, caching, or MCP design → unchanged from the plain Anthropic API; use those skills directly.
