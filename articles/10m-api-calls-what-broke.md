---
title: "We Routed 10 Million API Calls Last Month — Here's What Broke"
emoji: "🔥"
type: "tech"
topics: ["ai", "api", "infrastructure", "devops"]
published: true
---

# We Routed 10 Million API Calls Last Month. Here's What Broke.

CrazyRouter sits between developers and AI providers. Every API call to Claude, GPT, Gemini, DeepSeek, or any of the 300+ models we support passes through our gateway. Last month, that was 10.2 million requests.

Most of them worked fine. This article is about the ones that didn't, and what we built to make sure your app never notices.

## Incident #1: The Thursday Night Anthropic Brownout

February 13, 2026. 11:47 PM UTC. Our monitoring dashboard lit up.

Claude Sonnet requests started returning 529 (Overloaded) at a rate of 23% — up from the usual 0.3%. Not a full outage. Anthropic's status page showed green. But nearly one in four requests was failing.

For developers hitting the Anthropic API directly, this meant one thing: errors in production. Retry loops burning through rate limits. Users staring at loading spinners.

Here's what CrazyRouter users saw: nothing. Their apps kept working.

Our gateway detected the error rate spike within 90 seconds. By the 2-minute mark, new Claude Sonnet requests were automatically routing to GPT-4o as the primary, with Gemini Pro as secondary. Existing streaming connections to Anthropic continued normally — we don't interrupt in-flight requests.

The brownout lasted 47 minutes. During that window, we rerouted 31,000 requests. Zero failures reached end users.

**What we learned:** Provider status pages lie. "Operational" can mean 77% success rate. Your monitoring needs to be more granular than theirs.

## Incident #2: The DeepSeek Rate Limit Cliff

DeepSeek V3 is popular with our users. Great quality, absurdly cheap ($0.27/M input). But their rate limiting is... aggressive.

On February 19, a user's batch processing job hit DeepSeek's rate limit. Standard stuff — we'd retry with backoff. Except DeepSeek's rate limit response didn't include a `Retry-After` header. And their 429 responses came back with a 200 status code wrapping an error message in the response body.

```json
{
  "error": {
    "message": "Rate limit reached",
    "type": "requests",
    "code": "rate_limit_exceeded"
  }
}
```

HTTP 200. Our transport layer saw success. The response parser saw an error object instead of choices. The user's app got... nothing useful.

This is the kind of failure that retry logic can't catch. The HTTP layer thinks everything is fine. The application layer knows it isn't.

We added a response validation layer that sits between transport and application:

```python
def validate_response(self, raw_response, status_code):
    # Layer 1: HTTP status (catches obvious failures)
    if status_code >= 400:
        raise TransportError(status_code)
    
    # Layer 2: Response structure (catches provider quirks)
    body = raw_response.json()
    if "error" in body:
        raise ProviderError(body["error"])
    
    if "choices" not in body and "content" not in body:
        raise MalformedResponse(body)
    
    # Layer 3: Content quality (catches silent degradation)
    if body.get("choices", [{}])[0].get("finish_reason") == "length":
        self.metrics.record("truncated_response")
    
    return body
```

Three layers of validation. Because "the API returned 200" tells you almost nothing about whether the request actually succeeded.

**What we learned:** Every provider has quirks. DeepSeek wraps errors in 200s. Anthropic sometimes returns empty content blocks. Google occasionally sends malformed JSON in streaming mode. You can't trust any single signal.

## Incident #3: The Protocol Mismatch That Took Down Tool Calling

A developer reported that function calling worked with GPT-4o but broke when their request fell back to Claude Sonnet. Same prompt, same tools, same everything. GPT-4o called the function correctly. Claude returned a text description of what it would do, without actually calling anything.

The bug wasn't in Claude. It was in how we translated the tool schema.

OpenAI's function calling format:
```json
{
  "tools": [{
    "type": "function",
    "function": {
      "name": "get_weather",
      "parameters": { "type": "object", "properties": { "city": { "type": "string" } } }
    }
  }]
}
```

Anthropic's tool use format:
```json
{
  "tools": [{
    "name": "get_weather",
    "input_schema": { "type": "object", "properties": { "city": { "type": "string" } } }
  }]
}
```

Spot the difference? `function.parameters` vs `input_schema`. `type: "function"` wrapper vs flat structure. Our translator was handling the basic conversion, but edge cases in nested schemas were getting mangled.

We rewrote the protocol translation layer from scratch. Not just field mapping — full semantic translation that preserves behavior across providers:

```python
class ToolSchemaTranslator:
    def openai_to_anthropic(self, tools):
        return [{
            "name": t["function"]["name"],
            "description": t["function"].get("description", ""),
            "input_schema": self._deep_translate_schema(
                t["function"]["parameters"]
            )
        } for t in tools]
    
    def _deep_translate_schema(self, schema):
        # Handle nested $ref, allOf, oneOf, anyOf
        # Convert OpenAI's strict mode to Anthropic's equivalent
        # Preserve default values and enum constraints
        # ... 200 lines of edge case handling
```

200 lines of schema translation. Because "OpenAI-compatible" is a spectrum, not a binary.

**What we learned:** Protocol translation is the iceberg problem. The 80% case is trivial. The remaining 20% — nested schemas, streaming deltas, vision input formats, multi-turn tool use — is where everything breaks.

## The Numbers Nobody Talks About

After 6 months of running CrazyRouter in production, here are the reliability numbers across providers that no one publishes:

| Provider | Advertised Uptime | Actual Success Rate* | Common Failure Mode |
|----------|------------------|---------------------|-------------------|
| OpenAI | 99.9% | 97.1% | Rate limits during peak (429) |
| Anthropic | 99.5% | 96.8% | Overloaded errors (529) |
| Google | 99.9% | 98.2% | Regional routing failures |
| DeepSeek | Not published | 91.3% | Rate limits wrapped in 200s |
| Groq | 99.9% | 94.7% | Capacity limits on large models |
| Mistral | 99.9% | 97.5% | Occasional empty responses |

*Measured as "request returned usable content" not just "HTTP 200"

The gap between "HTTP 200" and "actually worked" is 1-8% depending on the provider. That's the gap CrazyRouter fills.

## What We Built (And Why It's Hard to DIY)

After these incidents, our routing architecture evolved into something we couldn't have designed upfront:

**1. Real-time health scoring.** Every provider gets a health score updated every 30 seconds based on error rate, latency p95, and response quality. Routing decisions use live scores, not static config.

**2. Capability-aware fallback.** When GPT-4o falls back to Claude, we verify the fallback supports every capability the request needs: vision, tool calling, JSON mode, streaming. A fallback that drops features is worse than an error.

**3. Cost-aware routing.** For requests where the user hasn't specified a model, we route to the cheapest option that meets the quality threshold. A classification task doesn't need Claude Opus.

**4. Protocol normalization.** Your request comes in as OpenAI format. We translate to whatever the target provider speaks. The response comes back normalized to OpenAI format. You never know which provider actually served it.

**5. Request hedging for critical paths.** For users who opt in, we send the same request to two providers simultaneously and return whichever responds first. Doubles cost, halves tail latency, eliminates single-provider risk.

## The Cost Equation

Here's the part that surprises people: multi-provider routing is usually cheaper than single-provider, not more expensive.

Why? Because the cheapest model that can handle your task varies by provider and by hour. Right now, DeepSeek V3 is the best value for code generation. Tomorrow, Groq might offer a faster Llama variant at half the price. Next week, Google might drop Gemini Flash pricing again.

CrazyRouter tracks pricing across all providers in real-time. When you send a request for "a model equivalent to GPT-4o," we route to whichever provider offers that capability tier at the lowest current price.

Average savings across our user base: **62% compared to using a single provider directly.**

## Try It

One API key. 300+ models. Automatic fallback. Smart routing.

```bash
# Replace your OpenAI base URL. That's it.
curl https://crazyrouter.com/v1/chat/completions \
  -H "Authorization: Bearer YOUR_KEY" \
  -d '{"model": "gpt-4o", "messages": [{"role": "user", "content": "Hello"}]}'
```

Works with every OpenAI-compatible tool: Cursor, Cline, Aider, LangChain, LlamaIndex, and more.

→ [crazyrouter.com](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=routing_incidents)
→ [GitHub](https://github.com/xujfcn/crazyrouter-openclaw)
→ [Telegram Community](https://t.me/crzrouter)