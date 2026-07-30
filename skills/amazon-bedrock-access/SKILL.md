---
name: "amazon-bedrock-access"
description: "Use when building applications that call Claude models through AWS Bedrock instead of the Anthropic API directly - covers the boto3 bedrock-runtime client, the converse/converse_stream methods, Bedrock model IDs and cross-region inference profiles, the toolConfig/toolSpec shape for tool use, and configuring Claude Code to run against Bedrock. Everything about prompting, evals, RAG, and MCP server design is identical to the plain Anthropic API and is covered by other skills - this one is scoped to what changes when Bedrock is the delivery mechanism."
---

## Purpose

Claude on Amazon Bedrock is the same models behind a different front door. Prompt engineering, prompt evaluation, RAG pipeline design, prompt caching rules, and MCP server/client design don't change based on which API delivers the model - so this skill doesn't re-cover them. It's scoped to the handful of things that are genuinely different when Bedrock is the access path: authentication, the request/response shape, model naming, and tool-use configuration.

## Setting up the client

Bedrock access goes through `boto3`, not an Anthropic SDK client:

```python
import boto3
client = boto3.client("bedrock-runtime", region_name="us-west-2")
```

Authentication is AWS-native, not an API key: either credentials in `~/.aws/credentials`, or `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` environment variables, resolved through the normal boto3 credential chain (so IAM roles on EC2/Lambda/ECS work too).

## Model IDs and regional availability

Not every model is available in every region. Requesting a model ID that isn't hosted in the region your client is configured for fails with an unhelpful "model not found"-style error rather than a clear regional message - if a request fails for a model you know exists, check region availability before anything else.

Cross-region inference profiles solve this: instead of a fixed model ID tied to one region, an inference profile ID routes the request to whichever region actually hosts that model. Profile IDs live in the Bedrock console under "Cross-region inference," not on the main model catalog page - pull the ID from there rather than guessing at the model catalog's naming.

## Making requests: converse and converse_stream

Bedrock uses the `converse` API rather than a `messages.create`-style call:

```python
user_message = {
    "role": "user",
    "content": [{"text": "What's 1+1?"}]
}
response = client.converse(modelId=model_id, messages=[user_message])
text = response["output"]["message"]["content"][0]["text"]
```

Two things differ from the plain Anthropic Messages API shape: `content` is always a list of typed blocks (even for plain text) so the same structure extends to images and other content without a schema change, and the generated text is nested several levels deep in the response rather than sitting at the top.

For streaming, `converse_stream` returns an object containing a generator that yields incremental events as text is produced - the request setup is otherwise the same. Use it whenever a chat UI needs to render tokens as they arrive instead of blocking for the full response.

## Tool use: toolConfig and toolSpec

Bedrock wraps tool definitions differently than the native Anthropic tool schema. Each tool is a `toolSpec` with `name`, `description`, and an `inputSchema` whose actual JSON Schema lives under an additional `json` key, and the full set goes into a `toolConfig` passed alongside `messages`:

```python
tool_config = {
    "tools": [
        {
            "toolSpec": {
                "name": "get_weather",
                "description": "...",
                "inputSchema": {"json": {"type": "object", "properties": {...}}}
            }
        }
    ]
}
response = client.converse(modelId=model_id, messages=messages, toolConfig=tool_config)
```

Building the schema itself (describe the function, write a sample-data dict, convert to JSON, add detailed per-property descriptions) works the same way it does for any tool-use integration - only the wrapper shape (`toolConfig` → `tools` → `toolSpec` → `inputSchema.json`) is Bedrock-specific. A tool schema written for the native Anthropic API needs its outer wrapper adjusted to this shape before it'll validate against Bedrock's `converse` call, even though the inner JSON Schema is unchanged.

## Running Claude Code against Bedrock

Claude Code can run against Bedrock instead of the Anthropic API by setting three environment variables before launching it:

```bash
export CLAUDE_CODE_USE_BEDROCK=1
export ANTHROPIC_MODEL='us.anthropic.claude-3-7-sonnet-20250219-v1:0'
export ANTHROPIC_SMALL_FAST_MODEL='us.anthropic.claude-3-5-haiku-20241022-v1:0'
```

`ANTHROPIC_MODEL` and `ANTHROPIC_SMALL_FAST_MODEL` take Bedrock model/inference-profile IDs, not the short model names used elsewhere. AWS credentials still need to be resolvable the normal way (`~/.aws/credentials` or the `AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY` env vars) for this to work - `CLAUDE_CODE_USE_BEDROCK` only changes which backend Claude Code talks to, not how it authenticates to AWS.

## What doesn't change

Prompt engineering (clarity, specificity, XML structure, examples), running prompt evaluations, RAG pipeline design (chunking, embeddings, BM25, reranking, contextual retrieval), prompt caching rules, extended thinking, image/PDF/citation support, and MCP server/client implementation are all identical regardless of whether requests go through Bedrock or the direct Anthropic API - the model-facing behavior doesn't know or care which transport delivered the request. Reach for a dedicated skill on that topic rather than expecting Bedrock-specific guidance there.

## Quick reference

- Setting up a client → `boto3.client("bedrock-runtime", region_name=...)`, not an Anthropic SDK client.
- "Model not found" error on a model you know exists → check regional availability, or switch to a cross-region inference profile ID from the Bedrock console.
- Sending a message → `client.converse(modelId=..., messages=[...])`; text comes back at `response["output"]["message"]["content"][0]["text"]`.
- Need token-by-token streaming → `converse_stream` instead of `converse`.
- Giving Claude tools → wrap each tool as `toolSpec` with `inputSchema.json`, and pass the set as `toolConfig={"tools": [...]}`.
- Running Claude Code against Bedrock → set `CLAUDE_CODE_USE_BEDROCK=1` plus `ANTHROPIC_MODEL`/`ANTHROPIC_SMALL_FAST_MODEL` to Bedrock model IDs, and make sure AWS credentials are resolvable.
- Anything about prompting, evals, RAG, caching, or MCP design → unchanged from the plain Anthropic API; use those skills directly.
