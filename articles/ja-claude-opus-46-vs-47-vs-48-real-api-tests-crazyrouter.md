---
title: "Claude Opus 4.6 vs 4.7 vs 4.8：Crazyrouter 経由の実 API テスト"
emoji: "🧪"
type: "tech"
topics: [claude, ai, api, llm]
published: true
---

# Claude Opus 4.6 vs 4.7 vs 4.8：Crazyrouter 経由の実 API テスト

日本語の結論：Opus 4.7 は合格率、Opus 4.8 は平均レイテンシ、Opus 4.6 は安定した既存用途に向いています。

本篇/この記事/이 글/Bài viết này 基于 `https://cn.crazyrouter.com/v1` 的真实调用结果，不是厂商宣传复述。测试模型：`claude-opus-4-6`、`claude-opus-4-7`、`claude-opus-4-8`。

![Claude Opus benchmark score and latency](https://raw.githubusercontent.com/xujfcn/images/main/blog/posts/claude-opus-46-47-48-score-latency.webp)

## 核心结果 / Key results

| Model | Score | Avg latency | Total tokens | Best fit |
|---|---:|---:|---:|---|
| `claude-opus-4-6` | 4/6 | 5.2s | 2847 | stable SQL, JSON, API review, Chinese support replies |
| `claude-opus-4-7` | 5/6 | 7.46s | 3297 | best overall pass rate, long-context extraction, structured output |
| `claude-opus-4-8` | 4/6 | 4.59s | 2838 | fastest average latency, concise JSON/API review, low token use |


## 测试矩阵 / Test matrix

![Claude Opus API test matrix](https://raw.githubusercontent.com/xujfcn/images/main/blog/posts/claude-opus-46-47-48-test-matrix.webp)

| Test | Opus 4.6 | Opus 4.7 | Opus 4.8 | What it checked |
|---|---|---|---|---|
| arithmetic revenue | ⚠️ | ⚠️ | ⚠️ | business arithmetic and step-by-step numeric reasoning |
| postgres sql | ✅ | ✅ | ✅ | Postgres query construction for paid users and token usage |
| long context extraction | ⚠️ | ✅ | ⚠️ | finding exact operational facts in a long noisy log |
| strict json no fence | ✅ | ✅ | ✅ | JSON-only schema following without markdown fences |
| api client review | ✅ | ✅ | ✅ | developer code review quality for an API client |
| chinese support reply | ✅ | ✅ | ✅ | Chinese customer-support answer with correct cn.crazyrouter.com/v1 guidance |


## 可复现实测代码

```python
from openai import OpenAI
client = OpenAI(api_key="YOUR_CRAZYROUTER_API_KEY", base_url="https://cn.crazyrouter.com/v1")
resp = client.chat.completions.create(
    model="claude-opus-4-8",
    messages=[{"role":"user","content":"Return valid JSON only."}],
    temperature=0,
)
print(resp.choices[0].message.content)
```

## 结论 / Conclusion

- `claude-opus-4-7`: best default for agent/log extraction workflows in this run.
- `claude-opus-4-8`: best latency profile in this run.
- `claude-opus-4-6`: still useful when prompts are already validated.

人看的链接加 UTM，代码里的 API base URL 不加 UTM。访问：[Crazyrouter](https://crazyrouter.com?utm_source=ja&utm_medium=article&utm_campaign=claude_opus_46_47_48)
