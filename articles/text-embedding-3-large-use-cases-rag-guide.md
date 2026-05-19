---
title: "text-embedding-3-large の用途：Embeddings 入門と RAG での使い方"
emoji: "🧠"
type: "tech"
topics: [ai, rag, embeddings, api]
published: true
---

# text-embedding-3-large の用途：Embeddings 入門と RAG での使い方

普段 GPT、Claude、Gemini などを使うとき、多くは「答えを生成する」用途ですよね。しかし `text-embedding-3-large` のようなモデルは、チャットや文章生成はしません。

このモデルの主な役割は、**テキストを数値ベクトルに変換すること** です。

少し抽象的に聞こえるかもしれませんが、RAG（知識拡張生成）、セマンティック検索、類似記事推薦、FAQボット、ドキュメント検索などの基盤となる技術です。

![text-embedding-3-large semantic search pipeline](https://raw.githubusercontent.com/xujfcn/images/main/blog/posts/semantic-search-pipeline.webp)

AI に「自分のドキュメントから答えを探してほしい」と思ったら、embedding は避けて通れません。

## text-embedding-3-large の主な用途

`text-embedding-3-large` は OpenAI の高性能テキストベクトル化モデルです。テキストを読み込み、高次元ベクトルを出力します。

このベクトルは、テキストの「意味的な座標」と考えるとイメージしやすいです。

例えば、以下のような文があります。

- 「AI API コストを下げる方法は？」
- 「GPT の利用料金を節約するには？」
- 「AI モデルの利用料が高すぎる場合は？」

キーワードは完全一致しませんが、意味は近いですよね。Embedding モデルは、これらを近い位置にマッピングします。

これにより、従来のキーワード検索では難しかった「意味で探す」が可能になります。

主な用途例：

| シーン | embedding の役割 |
|---|---|
| セマンティック検索 | ユーザーの質問とドキュメントをベクトル化し、最も近い内容を探す |
| RAG 知識ベース | 関連ドキュメントを検索し、大規模言語モデルに渡して回答生成 |
| レコメンドシステム | 記事・商品・ユーザー説明の意味的な類似度で推薦 |
| ドキュメントクラスタリング | 類似ドキュメントを自動でグループ化 |
| テキスト分類 | 類似度でどのラベルに属するか判定 |
| 異常検知 | 他と大きく意味が離れたデータを検出 |

OpenAI の公式ドキュメントでも、embeddings は search、clustering、recommendations、anomaly detection、diversity measurement、classification などに使われています。

## なぜ RAG で text-embedding-3-large が重要なのか？

RAG（Retrieval-Augmented Generation、検索拡張生成）は、以下のような流れで動きます。

1. ドキュメントを小さなチャンクに分割
2. embedding モデルで各チャンクをベクトル化
3. ベクトルをベクトルデータベースに保存
4. ユーザーの質問もベクトル化
5. 最も関連性の高いチャンクを検索
6. その内容を大規模言語モデルに渡して回答生成

embedding がなければ、単なるキーワードマッチしかできません。

例えば、ユーザーが

> 「なぜ残高がすぐ減るの？」

と質問したとします。ドキュメントには

> 「高コンテキストモデルはより多くのトークンを消費し、料金は入力・出力トークン数で計算されます。」

と書かれていた場合、キーワードは一致しませんが、意味は関連しています。Embedding はこうした「意味の橋渡し」をしてくれます。

![RAG workflow with embeddings and vector database](https://raw.githubusercontent.com/xujfcn/images/main/blog/posts/rag-workflow-vector-db.webp)

## text-embedding-3-large とチャットモデルの違い

初めて embedding を触ると「これも Q&A モデル？」と思いがちですが、実際は違います。

| 機能 | チャットモデル | embedding モデル |
|---|---|---|
| 入力 | ユーザーの質問や文脈 | テキスト断片 |
| 出力 | 自然言語の答え | 数値ベクトル |
| 主な用途 | 生成・要約・推論・対話 | 検索・類似度計算・クラスタリング・分類 |
| 直接質問に答えるか | はい | いいえ |
| RAG での役割 | 最終回答を生成 | 情報検索を担当 |

役割分担で例えると、

- embedding モデル：図書館の司書（資料を探す）
- チャットモデル：ライター（答えをまとめる）

知識ベースQAシステムでは、両方を組み合わせて使うのが一般的です。

## text-embedding-3-large の主なパラメータ

OpenAI 公式ドキュメントによると、`text-embedding-3-large` はデフォルトで 3072 次元のベクトルを出力し、最大入力は 8192 トークンです。

また `dimensions` パラメータで、主要な意味情報を保ったまま次元数を減らすことも可能です。

これは非常に重要で、ベクトルの次元数は

- ベクトルDBのストレージコスト
- 検索速度
- インデックスサイズ
- メモリ消費

に大きく影響します。

中小規模のFAQやカスタマーサポート用途なら、必ずしも 3072 次元フルで使う必要はありません。まずは 1024 や 1536 次元で試して、検索品質を確認しましょう。

## OpenAI 互換APIで text-embedding-3-large を使う方法

以下は Python の例です。API の base_url には UTM パラメータを付けないでください。

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

既に OpenAI SDK を使っている場合は、`base_url` を互換ゲートウェイのURLに変更するだけでOKです。

OpenAI 互換の接続方法は [Crazyrouter ドキュメント](https://docs.crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=embedding_i18n) を参照してください。コスト比較は [価格ページ](https://crazyrouter.com/pricing?utm_source=zenn&utm_medium=article&utm_campaign=embedding_i18n) も便利です。

## 最小構成のセマンティック検索サンプル

ここでは cosine similarity を使ったシンプルな例を紹介します。実運用では Qdrant、Milvus、Pinecone、pgvector などのベクトルDB利用を推奨します。

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

この例はシンプルですが、テキストをベクトル化し、類似度でソートするというセマンティック検索の基本ロジックが詰まっています。

## text-embedding-3-large を使うべきタイミング

判断の目安は以下の通りです。

| ニーズ | 推奨 |
|---|---|
| 高品質なRAG、クロス言語検索、長文知識ベース | まず text-embedding-3-large を試す |
| FAQや小規模検索、コスト重視 | まず text-embedding-3-small を検討 |
| 多言語ドキュメント検索 | large のテスト価値が高い |
| キーワードフィルタのみ | embedding は不要な場合も |
| データ量が膨大・予算厳しい | 次元削減や階層検索を検討 |

要するに、「検索品質がユーザー体験に直結する」なら `text-embedding-3-large` を優先的に試す価値があります。

社内ツールやMVP段階なら、より安価な embedding モデルから始めても良いでしょう。

## RAG プロジェクトのベストプラクティス

### 1. ドキュメントのチャンクは細かく

1文書を丸ごとembeddingせず、意味段落ごとに分割しましょう。

目安：

- 1チャンク 300～800トークン
- 50～100トークン程度のオーバーラップを持たせる
- タイトル・パス・日時などのメタデータは別管理

### 2. クエリも前処理する

ユーザーの質問は短く口語的なことが多いです。チャットモデルで検索向きクエリに書き換えてから embedding するのも有効です。

### 3. top 1 だけでなく top 3～10 を見る

RAG では通常、関連度上位3～10件のチャンクを抽出し、大規模言語モデルに渡します。

### 4. rerank（再ランキング）を組み合わせる

embedding で粗く絞り込み、rerank で精密に順位付け。カスタマーサポートや法務・財務・技術文書では rerank で誤答を大幅に減らせます。

### 5. 実際のクエリログを記録・評価

デモ用データだけでなく、実際のユーザー質問（匿名化済み）で評価しましょう。本当に使えるかどうかは現場データでしか分かりません。

## よくある勘違い

### 勘違い1：embedding の次元数は高いほど良い

必ずしもそうではありません。高次元ほど表現力は上がりますが、ストレージや計算コストも増えます。実際のプロジェクトでは検索精度・遅延・コストのバランスが重要です。

### 勘違い2：RAG で embedding を使えば絶対に誤答しない

これも誤りです。embedding は資料検索の役割だけ。最終的な回答の信頼性は、プロンプト設計・文脈品質・モデル能力・引用制御などにも依存します。

### 勘違い3：全てのドキュメントはそのままベクトル化できる

表・コード・ログ・PDFスキャンなどは特別な処理が必要です。特に表は構造化情報を残すのがベストです。

## 結論：text-embedding-3-large はAIアプリの「検索基盤」

`text-embedding-3-large` はチャット用ではなく、テキストの意味的類似度を理解するためのモデルです。

特に以下の用途に最適です。

- RAG 知識ベース
- セマンティック検索
- 多言語検索
- レコメンドシステム
- ドキュメントクラスタリング
- テキスト分類

AIカスタマーサポート、企業ナレッジベース、ドキュメントQA、コード検索、コンテンツ推薦などを作るなら、embedding モデルはシステムの基盤となります。

OpenAI 互換APIで簡単に導入でき、既存の OpenAI SDK プロジェクトなら `base_url` と API key を変えるだけで移行可能です。

## FAQ

### text-embedding-3-large は直接質問に答えられますか？

いいえ。出力はベクトルであり、自然言語の答えではありません。質問への回答には GPT、Claude、Gemini などのチャットモデルと組み合わせる必要があります。

### text-embedding-3-large は RAG に向いていますか？

はい。RAG の検索フェーズでよく使われます。ユーザーの質問や知識ベースの文書をベクトル化し、関連性の高い内容を抽出します。

### text-embedding-3-large のデフォルト次元数は？

公式ドキュメントによるとデフォルトは 3072 次元です。`dimensions` パラメータで次元数を下げることも可能です。

### text-embedding-3-large と text-embedding-3-small の選び方は？

検索品質や多言語対応、プロダクション品質のRAGを重視するなら large を優先的にテストしましょう。コスト重視や大規模データの場合は small でベースラインを作るのもおすすめです。

### ベクトルデータベースは必須ですか？

小規模なデモなら配列＋cosine similarity でも動きますが、実運用では pgvector、Qdrant、Milvus、Pinecone などのベクトルDB利用を推奨します。

