---
title: "text-embedding-3-large は使うべき？small とのコスト・品質・選び方"
emoji: "🧠"
type: "tech"
topics: [ai, rag, embeddings, api]
published: true
---

# text-embedding-3-large は使うべき？small とのコスト・品質・選び方

RAG やセマンティック検索を作るとき、多くの人が同じ疑問で止まります。

> 我到底该用 `text-embedding-3-large`，还是用更便宜的 `text-embedding-3-small`？

答えは「large が常に良い」でも「small で十分」でもありません。

更准确的说法是：**如果检索质量直接影响业务结果，large 值得测试；如果项目还在早期或数据量特别大，small 往往是更稳的起点。**

![embedding model cost and quality comparison dashboard](https://raw.githubusercontent.com/xujfcn/images/main/blog/posts/embedding-cost-quality-dashboard.webp)

実プロジェクトの観点から、embedding モデル選定で見るべきポイントを整理します。

## 先说结论：怎么选？

可以直接按这个表判断：

| 场景 | 推荐起点 | 原因 |
|---|---|---|
| 企业知识库问答 | text-embedding-3-large | 召回质量更重要 |
| 多语言 RAG | text-embedding-3-large | 非英文和跨语言检索更值得测试 |
| 客服机器人 | large 或 small A/B 测试 | 看错误回答成本 |
| 内部工具搜索 | text-embedding-3-small | 成本优先，容错较高 |
| MVP / demo | text-embedding-3-small | 先跑通链路 |
| 海量文档索引 | small 或 large 降维 | 控制存储和检索成本 |
| 代码/技术文档搜索 | large + rerank | 语义和精确匹配都重要 |

如果你只能记一句话：

**先用 small 建基线，再用 large 在真实 query 上评测提升。**

## text-embedding-3-large 强在哪里？

`text-embedding-3-large` 是更高能力的 embedding 模型。

根据 OpenAI 官方文档，它默认输出 3072 维向量，最大输入 8192 tokens，定位是英文和非英文任务上的高能力 embedding 模型。

它的优势主要体现在：

1. 更强的语义表达能力
2. 更适合复杂查询
3. 更适合多语言或跨语言检索
4. 对长文档知识库更友好
5. 更适合作为生产级 RAG 的候选模型

但它也有代价：

- 单位 token 成本更高
- 默认向量维度更大
- 向量库存储成本更高
- 检索时内存和索引压力更大

所以，选 large 的前提应该是：它带来的召回提升能覆盖额外成本。

## text-embedding-3-small 适合什么？

`text-embedding-3-small` 的定位更偏成本效率。

它适合：

- FAQ 搜索
- 小型知识库
- 早期 MVP
- 内部检索工具
- 单语言内容库
- 对错误召回容忍度较高的场景

很多项目没必要一开始就上 large。

尤其当你还没有真实用户问题、没有评测集、没有线上反馈时，上大模型向量可能只是“感觉更稳”，但你无法证明它真的更好。

## 成本不只看 API 价格

Embedding 成本不只是调用模型那一项。

你还要算：

| 成本项 | 受什么影响 |
|---|---|
| API 调用成本 | 输入 tokens 数量、重复索引次数 |
| 向量库存储 | 文档数量、chunk 数、向量维度 |
| 检索延迟 | 索引规模、维度、top_k |
| 内存/磁盘 | 向量数量和精度 |
| 维护成本 | 重建索引、版本迁移、评测 |

举个简单例子：

如果你有 100 万个 chunk，每个向量 3072 维，使用 float32 存储，单纯向量数据大约是：

```text
1,000,000 × 3072 × 4 bytes ≈ 12.3 GB
```

如果降到 1536 维，大约就是一半。

这还不包括索引结构、metadata、数据库额外开销。

所以大规模项目一定要关注 `dimensions` 参数和向量数据库成本。

## dimensions 参数怎么用？

OpenAI 第三代 embedding 模型支持通过 `dimensions` 控制输出向量大小。

示例：

```python
from openai import OpenAI

client = OpenAI(
    api_key="your-api-key",
    base_url="https://crazyrouter.com/v1"
)

response = client.embeddings.create(
    model="text-embedding-3-large",
    input="How to reduce AI API cost with model routing?",
    dimensions=1536,
    encoding_format="float"
)

vector = response.data[0].embedding
print(len(vector))
```

这对生产环境很有用。

你可以测试：

- large 3072 维
- large 1536 维
- large 1024 维
- small 默认维度

然后比较 top 5 召回率、延迟和成本。

## 选型不要只看 MTEB 分数

公开 benchmark 有参考价值，但不要把它当最终答案。

你的业务数据可能和 benchmark 完全不同：

- 用户问题很短
- 文档是中英混合
- 内容里有很多产品名和错误码
- 有大量表格和参数说明
- 用户经常用口语表达

所以最好建立自己的小评测集。

最简单的格式：

| query | 应召回文档 | 类型 |
|---|---|---|
| 怎么查看余额？ | billing.md | FAQ |
| base_url 应该填什么？ | quickstart.md | technical |
| Claude Code 怎么配置？ | claude-code.md | integration |
| API 调用失败 401 怎么办？ | auth-errors.md | troubleshooting |

每个模型跑一遍，看 top 3 / top 5 是否命中。

## 一个可执行的 A/B 测试流程

推荐你这样测：

### 1. 准备 50-200 个真实 query

不要自己拍脑袋写。优先用：

- 站内搜索日志
- 客服问题
- 用户群问题
- 工单标题
- 文档评论

### 2. 标注正确答案文档

每个 query 标 1-3 个正确 chunk 或文档。

### 3. 建立多个索引

例如：

- index_small_default
- index_large_3072
- index_large_1536

### 4. 跑召回评测

核心指标：

| 指标 | 含义 |
|---|---|
| Recall@3 | 前 3 个结果是否包含正确文档 |
| Recall@5 | 前 5 个结果是否包含正确文档 |
| MRR | 正确文档排得越靠前越好 |
| latency | 查询耗时 |
| cost | 索引和查询成本 |

### 5. 决定是否升级 large

如果 large 只提升 1%，但成本高很多，未必值得。

如果 large 把 Recall@5 从 78% 提到 90%，而业务又很依赖准确检索，那就很值得。

## RAG 质量差，未必是 embedding 模型的问题

很多时候，换成 large 也救不了系统。

因为问题可能在别处：

| 问题 | 表现 | 优先修复 |
|---|---|---|
| chunk 切坏了 | 找到的内容上下文不完整 | 重新切块 |
| metadata 缺失 | 无法按语言/权限/产品过滤 | 补 metadata |
| query 太短 | “扣费问题”召回不稳定 | query rewrite |
| 文档过期 | 找到了旧答案 | 加更新时间和版本过滤 |
| 没有 rerank | top 结果相似但不精确 | 加 rerank |
| prompt 太松 | 模型根据常识乱答 | 强制只根据资料回答 |

所以选模型前，先保证 RAG 管线别太粗糙。

## 在 OpenAI 兼容接口中切换 embedding 模型

如果你通过 OpenAI SDK 接入，切换模型很简单。

```python
from openai import OpenAI

client = OpenAI(
    api_key="your-api-key",
    base_url="https://crazyrouter.com/v1"
)

# 成本效率优先
small = client.embeddings.create(
    model="text-embedding-3-small",
    input="semantic search for AI documentation"
)

# 质量优先
large = client.embeddings.create(
    model="text-embedding-3-large",
    input="semantic search for AI documentation"
)
```

如果你用 Crazyrouter 这类 OpenAI 兼容网关，通常不用改 SDK，只改模型名和 base URL。

你可以先在 [Crazyrouter Playground](https://crazyrouter.com/playground?utm_source=blog&utm_medium=article&utm_campaign=dev_community) 验证调用，再把同样参数放进服务端。

## 推荐的项目路线图

### 阶段 1：MVP

- 用 text-embedding-3-small
- 本地或 pgvector 存储
- top 5 检索
- 不加复杂 rerank
- 目标：跑通链路

### 阶段 2：真实评测

- 收集用户 query
- 标注正确文档
- 测 small vs large
- 调整 chunk 策略
- 目标：找到召回瓶颈

### 阶段 3：生产优化

- 使用 large 或 large 降维版本
- 加 hybrid search
- 加 rerank
- 加权限过滤
- 加引用来源
- 目标：稳定、可解释、可控成本

## 结论：large 不是默认答案，但值得认真测试

`text-embedding-3-large` 的价值在于更强的语义表达能力，尤其适合高质量 RAG、多语言知识库和复杂语义搜索。

但工程上不要盲目上最强模型。

更好的策略是：

1. 用 small 建立成本和效果基线
2. 用真实 query 测 large 的提升
3. 用 dimensions 控制向量大小
4. 加 rerank 和 hybrid search，而不是只靠换模型
5. 根据业务错误成本决定是否升级

如果 RAG 结果直接影响用户决策、客服体验或付费转化，`text-embedding-3-large` 很可能值得。

如果只是内部工具或早期验证，先从 small 开始更务实。

## FAQ

### text-embedding-3-large 一定比 text-embedding-3-small 好吗？

通常能力更强，但不代表每个项目都值得用。要看真实 query 的召回提升是否覆盖成本。

### dimensions 降低后会不会影响效果？

可能会。降维能减少存储和检索成本，但可能降低召回质量。建议用自己的评测集测试。

### RAG 项目应该先优化模型还是切块？

先优化切块和评测。chunk 策略很差时，换更强 embedding 模型也可能效果有限。

### 多语言知识库更适合 text-embedding-3-large 吗？

值得优先测试。官方定位里 large 适合英文和非英文任务，多语言检索通常更依赖语义表达能力。

### 可以用 Crazyrouter 调用 text-embedding-3-large 吗？

可以通过 OpenAI 兼容的 `/v1/embeddings` 端点调用。代码里使用 `https://crazyrouter.com/v1` 作为 base URL 即可。


## Crazyrouter で試す

Embeddings、RAG、複数 AI モデルを 1 つの OpenAI-compatible API キーで扱いたい場合は、[Crazyrouter](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=embedding_i18n) を試せます。SDK の base URL は `https://crazyrouter.com/v1` です。
