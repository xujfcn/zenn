---
title: "text-embedding-3-large の用途：Embeddings 入門と RAG での使い方"
emoji: "🧠"
type: "tech"
topics: [ai, rag, embeddings, api]
published: true
---

# text-embedding-3-large の用途：Embeddings 入門と RAG での使い方

GPT、Claude、Gemini を呼び出すとき、多くの場合は「回答を生成する」ことを期待します。しかし `text-embedding-3-large` のようなモデルはチャットしたり記事を書いたりするためのものではありません。

中心的な役割は、**テキストを数値ベクトルに変換すること**です。

少し抽象的に聞こえますが、RAG ナレッジベース、セマンティック検索、関連記事推薦、FAQ ボット、ドキュメント検索の土台になる技術です。

![text-embedding-3-large semantic search pipeline](https://raw.githubusercontent.com/xujfcn/images/main/blog/posts/semantic-search-pipeline.webp)

AI に自社ドキュメントから答えを探させたいなら、モデルの記憶に頼るだけでは足りません。embeddings はほぼ必須です。

## text-embedding-3-large 的核心用途是什么？

`text-embedding-3-large` 是 OpenAI 的高能力文本向量模型。它会读取一段文本，然后输出一个高维向量。

向量可以理解成文本的“语义坐标”。

比如下面几句话：

- “如何降低 AI API 成本？”
- “怎么节省 GPT 接口费用？”
- “AI 模型调用太贵怎么办？”

关键词不完全一样，但意思接近。Embedding 模型会把它们映射到相近的位置。

这让系统可以做传统关键词搜索做不好的事情：**按语义找内容**。

常见用途包括：

| 场景 | embedding 在里面做什么 |
|---|---|
| 语义搜索 | 把用户问题和文档都转成向量，找最相似内容 |
| RAG 知识库 | 先检索相关文档，再交给大模型生成答案 |
| 推荐系统 | 根据文章、商品、用户描述的语义相似度推荐内容 |
| 文档聚类 | 自动把相似文档分组 |
| 文本分类 | 用相似度判断文本属于哪个标签 |
| 异常检测 | 找出和大多数内容语义距离很远的数据 |

OpenAI 官方文档也把 embeddings 用于 search、clustering、recommendations、anomaly detection、diversity measurement 和 classification。

## 为什么 RAG 特别需要 text-embedding-3-large？

RAG，全称 Retrieval-Augmented Generation，中文可以理解为“检索增强生成”。

它的流程通常是：

1. 把文档切成小块
2. 用 embedding 模型把每个文档块转成向量
3. 存进向量数据库
4. 用户提问时，把问题也转成向量
5. 找出最相关的几个文档块
6. 把这些内容交给大模型回答

如果没有 embedding，系统只能做关键词匹配。

比如用户问：

> “余额为什么扣得这么快？”

文档里可能写的是：

> “高上下文模型会消耗更多 tokens，费用按输入和输出 tokens 计费。”

关键词几乎对不上，但语义相关。Embedding 就是解决这种问题的。

![RAG workflow with embeddings and vector database](https://raw.githubusercontent.com/xujfcn/images/main/blog/posts/rag-workflow-vector-db.webp)

## text-embedding-3-large 和聊天模型有什么区别？

很多开发者第一次接触 embedding 会误以为它也是“问答模型”。其实不是。

| 能力 | 聊天模型 | embedding 模型 |
|---|---|---|
| 输入 | 用户问题、上下文 | 文本片段 |
| 输出 | 自然语言答案 | 数字向量 |
| 主要用途 | 生成、总结、推理、对话 | 检索、相似度、聚类、分类 |
| 是否直接回答问题 | 是 | 否 |
| 是否适合 RAG | 负责最终回答 | 负责找资料 |

可以把它们分工理解为：

- embedding 模型：图书管理员，负责找资料
- 聊天模型：写作者，负责组织答案

一个完整的知识库问答系统，通常两者都要用。

## text-embedding-3-large 的关键参数

根据 OpenAI 官方文档，`text-embedding-3-large` 默认输出 3072 维向量，最大输入上下文为 8192 tokens。

它也支持 `dimensions` 参数。你可以在保留主要语义信息的前提下，把向量维度降下来。

这很重要，因为向量维度会影响：

- 向量数据库存储成本
- 检索速度
- 索引大小
- 内存占用

如果你只是做中小型 FAQ、客服知识库，未必一定要 3072 维全量向量。可以先测试 1024 或 1536 维，观察召回质量。

## 如何用 OpenAI 兼容接口调用 text-embedding-3-large？

下面是 Python 示例。代码里的 API 地址不要加 UTM 参数。

```python
from openai import OpenAI

client = OpenAI(
    api_key="your-api-key",
    base_url="https://crazyrouter.com/v1"
)

response = client.embeddings.create(
    model="text-embedding-3-large",
    input="AI API gateway helps developers connect to many models with one key.",
    encoding_format="float"
)

vector = response.data[0].embedding
print(len(vector))
print(vector[:5])
```

如果你已经在项目里使用 OpenAI SDK，只需要把 `base_url` 换成兼容网关地址即可。

你可以通过 [Crazyrouter 文档](https://docs.crazyrouter.com?utm_source=blog&utm_medium=article&utm_campaign=dev_community) 查看 OpenAI 兼容接入方式，也可以在 [价格页面](https://crazyrouter.com/pricing?utm_source=blog&utm_medium=article&utm_campaign=dev_community) 对比不同模型成本。

## 一个最小可用的语义搜索示例

下面用最简单的 cosine similarity 演示 embedding 的工作方式。生产环境建议换成向量数据库，例如 Qdrant、Milvus、Pinecone、pgvector。

```python
from openai import OpenAI
import numpy as np

client = OpenAI(
    api_key="your-api-key",
    base_url="https://crazyrouter.com/v1"
)

def embed(text: str):
    res = client.embeddings.create(
        model="text-embedding-3-large",
        input=text,
        encoding_format="float"
    )
    return np.array(res.data[0].embedding)

def cosine(a, b):
    return float(np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b)))

docs = [
    "AI API costs are calculated by input and output tokens.",
    "RAG systems retrieve relevant documents before generating answers.",
    "Vector databases store embeddings for semantic search."
]

query = "How does retrieval augmented generation find context?"

query_vec = embed(query)
doc_vecs = [embed(doc) for doc in docs]

scores = [(doc, cosine(query_vec, vec)) for doc, vec in zip(docs, doc_vecs)]
scores.sort(key=lambda x: x[1], reverse=True)

for doc, score in scores:
    print(round(score, 4), doc)
```

这个例子虽然简单，但已经包含了语义搜索的核心逻辑：文本转向量，然后按相似度排序。

## 什么时候该用 text-embedding-3-large？

我建议按下面的方式判断：

| 需求 | 建议 |
|---|---|
| 高质量 RAG、跨语言检索、长文档知识库 | 优先测试 text-embedding-3-large |
| FAQ、小规模搜索、成本敏感项目 | 可以先用 text-embedding-3-small |
| 多语言文档检索 | 更值得测试 large |
| 只做关键词过滤 | 不一定需要 embedding |
| 数据量极大、预算紧 | 先评估降维和分层检索 |

一句话：如果你的搜索质量直接影响用户体验，`text-embedding-3-large` 值得优先测试。

如果只是内部工具或早期 MVP，可以先从更便宜的 embedding 模型开始。

## RAG 项目里的最佳实践

### 1. 文档切块不要太粗

不要把整篇文档直接塞进去。建议按语义段落切块。

常见范围：

- 300-800 tokens 一个 chunk
- 保留 50-100 tokens overlap
- 标题、路径、时间等 metadata 单独存储

### 2. 查询也要预处理

用户问题可能很短、很口语。可以先让聊天模型把问题改写成更适合检索的查询，再做 embedding。

### 3. 不要只看 top 1

RAG 通常取 top 3 到 top 10 个文档块，再交给大模型判断。

### 4. 加 rerank 会更稳

Embedding 负责粗召回，rerank 负责精排。对于客服、法律、财务、技术文档，rerank 能明显减少答非所问。

### 5. 记录真实查询日志

不要只用 demo 数据测试。把真实用户问题脱敏后做评测，才知道召回是否靠谱。

## 常见误区

### 误区 1：embedding 维度越高一定越好

不一定。高维向量通常表达能力更强，但也更占存储和计算。实际项目要看召回效果、延迟和成本。

### 误区 2：RAG 有了 embedding 就不会胡说

也不对。Embedding 只负责找资料。最终回答是否可靠，还取决于 prompt、上下文质量、模型能力和引用约束。

### 误区 3：所有文档都适合直接向量化

表格、代码、日志、PDF 扫描件都需要特殊处理。尤其是表格，最好保留结构化字段。

## 结论：text-embedding-3-large 是 AI 应用的“检索底座”

`text-embedding-3-large` 不是用来聊天的模型，而是用来理解文本相似度的模型。

它最适合这些场景：

- RAG 知识库
- 语义搜索
- 多语言检索
- 推荐系统
- 文档聚类
- 文本分类

如果你正在做 AI 客服、企业知识库、文档问答、代码搜索或内容推荐，embedding 模型就是系统的基础层。

你可以通过 OpenAI 兼容接口接入 embeddings API。对于已经使用 OpenAI SDK 的项目，迁移成本很低：改 `base_url`，换 API key，然后继续使用同样的调用方式。

## FAQ

### text-embedding-3-large 可以直接回答问题吗？

不可以。它输出的是向量，不是自然语言答案。回答问题通常需要搭配 GPT、Claude、Gemini 等聊天模型。

### text-embedding-3-large 适合做 RAG 吗？

适合。它常用于 RAG 的检索阶段，把用户问题和知识库文档转成向量，再找出最相关内容。

### text-embedding-3-large 默认多少维？

官方文档显示默认是 3072 维，也可以通过 `dimensions` 参数降低输出维度。

### text-embedding-3-large 和 text-embedding-3-small 怎么选？

如果追求检索质量、多语言效果或生产级 RAG，可以优先测试 large。如果成本敏感或数据量很大，可以先用 small 做基线。

### 向量数据库是必须的吗？

小规模 demo 可以直接用数组和 cosine similarity。生产环境建议使用向量数据库，例如 pgvector、Qdrant、Milvus 或 Pinecone。
