---
title: "/v1/chat/completions vs /v1/responses vs /v1/messages: Which AI API Endpoint Sho"
emoji: "🔌"
type: "tech"
topics: [api, openai, claude, ai]
published: true
---

# /v1/chat/completions vs /v1/responses vs /v1/messages: Which AI API Endpoint Should You Use?

A common support issue in AI API gateways is not the API key, not the model, and not the SDK. It is the endpoint.

A user picks a model that exists, but sends the request to the wrong endpoint. The result looks like:

- model unavailable;
- unsupported endpoint;
- invalid request body;
- tool calling not working;
- streaming format mismatch;
- Claude model works in one tool but fails in another.

This guide explains the difference between three endpoint families:

```text
/v1/chat/completions
/v1/responses
/v1/messages
```

Short version:

| Endpoint | Native ecosystem | Best for | Request style |
|---|---|---|---|
| `/v1/chat/completions` | OpenAI-compatible legacy/current chat | Most apps, SDKs, LiteLLM, Cursor-style tools, simple chat | `messages: [{role, content}]` |
| `/v1/responses` | Newer OpenAI Responses API | Tool use, multimodal, reasoning items, newer OpenAI-style agents | `input`, `tools`, structured response items |
| `/v1/messages` | Anthropic Claude API | Claude-native SDKs and Claude-style apps | `messages` plus top-level `system`, Anthropic schema |

If your client says “OpenAI-compatible”, start with:

```text
https://crazyrouter.com/v1/chat/completions
```

or set base URL to:

```text
https://crazyrouter.com/v1
```

and let the SDK append `/chat/completions`.

If your tool specifically requires the OpenAI Responses API, use `/v1/responses`.

If your tool is Anthropic-native and expects Claude’s Messages API, use `/v1/messages`.

## The most common mistake

The most common mistake is mixing the model family and the endpoint family.

For example, a Claude model can be exposed through an OpenAI-compatible gateway, but that does not automatically mean your request body should use Anthropic’s native `/v1/messages` schema.

Likewise, a tool that sends Anthropic-native requests to `/v1/messages` cannot be fixed by only changing the model name. The endpoint and request body must match.

Think of it this way:

```text
model name + endpoint + request schema must match
```

If one of the three is wrong, the model may look unavailable even when the model itself is healthy.

## What `/v1/chat/completions` is

`/v1/chat/completions` is the classic OpenAI-compatible chat endpoint.

A typical request looks like this:

```bash
curl https://crazyrouter.com/v1/chat/completions \
  -H "Authorization: Bearer $CRAZYROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-5.5",
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "Explain API endpoints in one paragraph."}
    ]
  }'
```

Use `/v1/chat/completions` when:

- the app says it supports an OpenAI-compatible API;
- the config asks for `OPENAI_BASE_URL` or `base_url`;
- the request body uses `messages` with `role` and `content`;
- you are using common OpenAI SDK compatibility mode;
- you are configuring tools like Cursor-style clients, LiteLLM-style routers, FastGPT-style apps, or many chat UIs.

For many users, this is the safest default endpoint.

## What `/v1/responses` is

`/v1/responses` is the newer OpenAI Responses API endpoint.

It is designed around a more general response object, and it can represent things like:

- text output;
- tool calls;
- multimodal input;
- reasoning items;
- structured output;
- agent-like workflows.

A simplified request looks like this:

```bash
curl https://crazyrouter.com/v1/responses \
  -H "Authorization: Bearer $CRAZYROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-5.5",
    "input": "Explain the difference between chat completions and responses."
  }'
```

Use `/v1/responses` when:

- the tool explicitly says it uses the OpenAI Responses API;
- the config mentions `responses` rather than `chat.completions`;
- the request body uses `input` instead of `messages`;
- the model/provider route supports Responses API behavior;
- you need newer OpenAI-style tool or reasoning output formats.

Do not send a Chat Completions body to `/v1/responses` unless the gateway explicitly documents that conversion.

## What `/v1/messages` is

`/v1/messages` is the Anthropic Messages API style endpoint.

A typical Claude-native request looks like this:

```bash
curl https://crazyrouter.com/v1/messages \
  -H "Authorization: Bearer $CRAZYROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-sonnet-4-6",
    "max_tokens": 1024,
    "system": "You are a helpful assistant.",
    "messages": [
      {"role": "user", "content": "Explain Claude Messages API."}
    ]
  }'
```

Use `/v1/messages` when:

- the client is Anthropic-native;
- the SDK expects Claude Messages API format;
- the request has top-level `system` instead of a system message inside the array;
- the request requires Anthropic-specific content blocks or tool schema;
- the documentation explicitly says to call `/v1/messages`.

Do not assume every Claude model should be called with `/v1/messages`. If you are using an OpenAI-compatible gateway or SDK, Claude models may be called through `/v1/chat/completions` instead.

## Why endpoint mistakes make models look unavailable

When a model fails, users often assume the model is down. But in many cases, the model is fine and the request format is wrong.

Common mismatch examples:

| Mistake | What happens | Fix |
|---|---|---|
| Sending `messages` chat body to `/v1/responses` | Invalid request body or ignored fields | Use `/v1/chat/completions` or change body to `input` |
| Sending Anthropic `system` + `messages` body to `/v1/chat/completions` | Schema mismatch | Use `/v1/messages` or convert to OpenAI message format |
| Using `/v1/messages` in an OpenAI SDK | SDK may append the wrong path or parse the wrong response | Use OpenAI-compatible base URL with chat completions |
| Missing `/v1` in base URL | 404 or unknown route | Set base URL to `https://crazyrouter.com/v1` |
| Adding `/chat/completions` to base URL and SDK appends it again | Double path like `/chat/completions/chat/completions` | Base URL should usually end at `/v1` |
| Adding UTM parameters to API endpoint | Auth/routing errors | UTM only belongs on website links |

## Base URL vs full endpoint

Many tools ask for a Base URL, not a full endpoint.

Correct Base URL:

```text
https://crazyrouter.com/v1
```

Then the SDK calls:

```text
https://crazyrouter.com/v1/chat/completions
```

Wrong Base URL for most SDKs:

```text
https://crazyrouter.com/v1/chat/completions
```

Why? Because the SDK may append `/chat/completions` again.

Rule:

- If the field is called `base_url`, use `/v1`.
- If the field is called `endpoint` and asks for a full URL, use the full path the tool requires.

## Which endpoint should I choose?

Use this decision tree:

1. Does your tool say “OpenAI-compatible API”?  
   Use `/v1/chat/completions` or Base URL `https://crazyrouter.com/v1`.

2. Does your tool specifically say “Responses API”?  
   Use `/v1/responses`.

3. Does your tool use the Anthropic SDK or Claude Messages API?  
   Use `/v1/messages`.

4. Are you unsure?  
   Start with `/v1/chat/completions`, because most third-party clients support it.

## Recommended Crazyrouter configuration

For most OpenAI-compatible tools:

```text
API Key: your Crazyrouter API key
Base URL: https://crazyrouter.com/v1
Model: choose a model from the Crazyrouter model list
```

For users in regions where the global route is unstable:

```text
Base URL: https://cn.crazyrouter.com/v1
```

Human-facing links can use UTM tracking:

```text
https://crazyrouter.com/models?utm_source=blog&utm_medium=article&utm_campaign=api_endpoints
```

API endpoints should not:

```text
https://api-endpoint.example.invalid/v1?utm_source=blog
```

## Quick debugging checklist

Before reporting “model unavailable,” check these five things:

1. Is the model name spelled exactly as shown in the model list?
2. Is the endpoint family correct: chat completions, responses, or messages?
3. Does the request body match that endpoint’s schema?
4. Is the Base URL exactly `/v1`, not missing it and not duplicating endpoint paths?
5. Are you using the right SDK mode: OpenAI-compatible or Anthropic-native?

## Final recommendation

If you are building general app integrations, start with `/v1/chat/completions`. It is the broadest compatibility path.

Use `/v1/responses` when your client or workflow explicitly requires the Responses API.

Use `/v1/messages` when you are using Anthropic-native tooling.

Most endpoint problems disappear when you remember this rule:

```text
OpenAI-compatible client → /v1/chat/completions
OpenAI Responses client → /v1/responses
Anthropic-native client → /v1/messages
```

If the model exists but appears unavailable, do not only change the model name. Check the endpoint and request schema first.

---

Canonical: https://crazyrouter.com/blog/chat-completions-vs-responses-vs-messages-api?utm_source=zenn&utm_medium=article&utm_campaign=api_endpoints
