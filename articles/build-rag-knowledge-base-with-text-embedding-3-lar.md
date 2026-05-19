---
title: "text-embedding-3-large で RAG ナレッジベースを作る：チャンク分割から検索順位付けまで"
emoji: "🧠"
type: "tech"
topics: [ai, rag, embeddings, api]
published: true
---

# text-embedding-3-large で RAG ナレッジベースを作る：チャンク分割から検索順位付けまで

多くのRAGデモは一見シンプルです。PDFをアップロードして、質問するとAIが答えてくれる。

でも実際に本番運用しようとすると、すぐに課題が出てきます。

- ドキュメントが長すぎて、そのままではコンテキストに入りきらない
- キーワード検索では意味が近い内容を見つけられない
- ユーザーの質問は口語的、ドキュメントは形式的でギャップが大きい
- 検索結果が的外れだと、モデルが「それっぽく」作り話を始める
- データ量が増えると、検索速度やコストが一気に上がる

`text-embedding-3-large` が解決するのは、まさにこの中核部分、**質問とドキュメントを比較可能な意味ベクトルに変換すること**です。

![RAG knowledge base architecture with embeddings](https://raw.githubusercontent.com/xujfcn/images/main/blog/posts/rag-architecture-embeddings.webp)

この記事では抽象的な話は抜きにして、エンジニア視点で実践的なRAG構築フローを解説します。

## RAGシステムの基本構成

一般的なRAGナレッジベースは、オフラインとオンラインの2つの処理チェーンに分かれます。

まずはオフラインのインデックス作成フロー：

1. ドキュメント収集
2. テキストのクレンジング
3. チャンク分割（chunking）
4. embeddingモデルでベクトル化
5. ベクトルDBに保存

次にオンラインのQAフロー：

1. ユーザーが質問
2. 質問をembedding化
3. ベクトルDBで類似チャンクを検索
4. （任意）rerankで再順位付け
5. コンテキストを組み立て
6. チャットモデルで回答生成

この流れの中で、`text-embedding-3-large` は主に4番目と、オンラインの2番目で使われます。

最終的な回答は生成しませんが、モデルが正しい情報にアクセスできるかどうかを左右します。

## ステップ1：ドキュメント準備とチャンク分割

RAGの品質はチャンク設計に大きく左右されます。

チャンクが大きすぎると、ノイズが増えて無関係な情報まで拾ってしまう。

逆に小さすぎると、文脈が切れて必要な情報が足りなくなる。

よく使われる目安は以下の通りです：

| ドキュメント種別 | 推奨チャンクサイズ | オーバーラップ |
|---|---:|---:|
| FAQ・ヘルプセンター | 200-500トークン | 30-80トークン |
| 技術ドキュメント | 400-800トークン | 80-120トークン |
| レポート・論文 | 600-1000トークン | 100-150トークン |
| コードドキュメント | 関数/クラス/見出し単位 | ケースバイケース |

チャンク分割のサンプルコード：

```python
def chunk_text(text, chunk_size=600, overlap=100):
    words = text.split()
    chunks = []
    start = 0

    while start < len(words):
        end = start + chunk_size
        chunk = " ".join(words[start:end])
        chunks.append(chunk)
        start = end - overlap

    return chunks
```

本番環境では単純なスペース区切りではなく、見出し・段落・リスト・コードブロックの境界で分割するのが理想です。

## ステップ2：text-embedding-3-large でベクトル化

以下はOpenAI互換APIを使った例です。同じSDKで `/v1/embeddings` エンドポイントを呼び出せます。

```python
from openai import OpenAI

client = OpenAI(
    api_key="your-api-key",
    base_url="https://crazyrouter.com/v1"
)

def get_embedding(text: str):
    text = text.replace("\n", " ")
    response = client.embeddings.create(
        model="text-embedding-3-large",
        input=text,
        encoding_format="float"
    )
    return response.data[0].embedding
```

ベクトルの次元数を指定したい場合は、`dimensions` パラメータを追加します：

```python
response = client.embeddings.create(
    model="text-embedding-3-large",
    input=text,
    dimensions=1536,
    encoding_format="float"
)
```

次元数を下げるとストレージや検索コストは下がりますが、検索精度も落ちる可能性があります。実際のクエリでA/Bテストするのがおすすめです。

## ステップ3：ベクトルDBに保存

小規模な検証ならローカルファイルやSQLiteでも動きますが、本番運用では専用のベクトルDBを使いましょう。

代表的な選択肢：

| ツール | 適した用途 |
|---|---|
| pgvector | 既存のPostgreSQLを活用したい場合 |
| Qdrant | 独立型ベクトルDB、導入が簡単でフィルタ機能が強力 |
| Milvus | 大規模ベクトル検索 |
| Pinecone | フルマネージド型、運用不要 |
| Weaviate | スキーマやハイブリッド検索対応 |

どのDBでも、各チャンクには最低限以下の情報を持たせるのが推奨です：

```json
{
  "id": "doc_001_chunk_003",
  "text": "chunk content here",
  "embedding": [0.0123, -0.0456],
  "metadata": {
    "source": "billing-guide.md",
    "title": "Billing Guide",
    "section": "Token pricing",
    "updated_at": "2026-05-18"
  }
}
```

メタデータは非常に重要です。製品・言語・日付・権限などで柔軟に検索結果を絞り込めます。

## ステップ4：検索時のセマンティック検索

ユーザーが質問したら、まずembedding化してからベクトルDBで類似チャンクを検索します。

```python
def retrieve(query: str, vector_db, top_k=5):
    query_vector = get_embedding(query)
    results = vector_db.search(
        vector=query_vector,
        top_k=top_k,
        filter={"language": "zh"}
    )
    return results
```

ベクトルDBを使わずに、numpyでコサイン類似度を計算するサンプルも：

```python
import numpy as np

def cosine_similarity(a, b):
    a = np.array(a)
    b = np.array(b)
    return float(np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b)))
```

ただし、数万件を超えるとベクトルDBを使うべきです。

## ステップ5：rerankで「的外れ回答」を減らす

Embedding検索は「粗い」検索です。高速ですが、必ずしも最適な順序とは限りません。

Rerankは「精密」な再順位付け。クエリと候補ドキュメントの関連度を再評価します。

おすすめの流れ：

1. embeddingでtop20を取得
2. rerankで順位付け
3. 最終的にtop5をチャットモデルに渡す

embeddingのtop5をそのまま使うより、安定した精度が出ます。特に以下のようなケースで有効です：

- 技術ドキュメントQA
- カスタマーサポートナレッジ
- 法務・財務資料
- 多言語コンテンツ
- ドキュメント数が多く、タイトルが似ている場合

Crazyrouterは `/v1/rerank` のような再順位付けエンドポイントを提供しているので、RAGの検索パイプラインに組み込むのが簡単です。

## ステップ6：検索結果をチャットモデルに渡す

検索したチャンクをプロンプトとして組み立てます：

```python
def build_prompt(question, chunks):
    context = "\n\n".join(
        f"Source: {c['metadata']['source']}\n{c['text']}"
        for c in chunks
    )

    return f"""
あなたは厳密なナレッジベースアシスタントです。
以下の資料だけを根拠に質問に答えてください。
資料に答えがなければ「資料中に見つかりませんでした」と返答してください。

資料：
{context}

質問：{question}
"""
```

そしてチャットモデルで回答を生成します。

```python
answer = client.chat.completions.create(
    model="gpt-5-mini",
    messages=[{"role": "user", "content": build_prompt(question, chunks)}]
)

print(answer.choices[0].message.content)
```

ここでembeddingモデルとチャットモデルは役割が分かれています：

- `text-embedding-3-large`：関連資料の検索
- `gpt-5-mini` / Claude / Gemini：資料をもとに回答を生成

## text-embedding-3-large の多言語RAGでの強み

多くのチームでは、ドキュメントが単一言語とは限りません。

例えば：

- 英語APIドキュメント
- 中国語ヘルプセンター
- 日本語ユーザーマニュアル
- 韓国語コミュニティ記事
- ベトナム語チュートリアル

多言語RAGの難しさは、ユーザーの質問と言語が異なる資料に答えがある場合です。

`text-embedding-3-large` は公式にも英語・非英語の両方に強いembeddingモデルとされています。クロスリンガル検索用途では、まず候補としてテストする価値があります。

ただし、公式ベンチマークだけでなく、自分のデータで評価しましょう。

| クエリ | 正解ドキュメント | 言語 | 召喚できたか |
|---|---|---|---|
| 余额为什么扣费快？ | billing-token-cost.md | zh | yes/no |
| how to set API base URL | quickstart.md | en | yes/no |
| Claude Code 怎么配置？ | integrations/claude-code.md | zh | yes/no |

最終的にはtop3/top5の召喚率で評価します。

## 本番運用のためのTips

### 1. 増分インデックスで効率化

ドキュメント更新時は、変更があったチャンクだけ再ベクトル化しましょう。

各チャンクにハッシュ値を保存しておくと便利です：

```python
import hashlib

def content_hash(text):
    return hashlib.sha256(text.encode("utf-8")).hexdigest()
```

ハッシュが変わらなければ再処理不要です。

### 2. バッチembeddingを活用

1件ずつAPIを呼ぶのは非効率です。Embeddings APIは通常バッチ入力に対応しています。

```python
response = client.embeddings.create(
    model="text-embedding-3-large",
    input=[chunk1, chunk2, chunk3],
    encoding_format="float"
)
```

これで高速化＆リクエスト数のコントロールがしやすくなります。

### 3. ハイブリッド検索を組み合わせる

ベクトル検索だけだと、エラーコード・注文番号・関数名などの完全一致ワードを見逃すことがあります。

より堅牢な方法は：

- BM25やキーワード検索
- ベクトル検索
- 結果をマージ
- rerankで最終順位付け

### 4. 回答に出典を明記

答えだけでなく、参照元のタイトルやリンクも一緒に出すのがベストです。

ユーザーの信頼性が上がり、誤った召喚の検証もしやすくなります。

### 5. 権限付き検索

企業ナレッジベースでは必須です。

「検索後にフィルタ」ではなく、「ベクトルDB検索時に権限条件を付与」しましょう。

## よくあるトラブルと対策

| 問題 | 主な原因 | 解決策 |
|---|---|---|
| 正しいドキュメントが見つからない | チャンクサイズ不適切、クエリが短すぎ | チャンク調整、クエリリライト |
| 回答が誤った資料を参照 | top_kが小さい、rerank未導入 | top20＋rerank |
| レイテンシが高い | 毎回ドキュメントをembedding化 | ドキュメントはオフラインで、クエリのみリアルタイムembedding |
| コストが急増 | 重複インデックス、次元数が高すぎ | ハッシュで重複排除、次元数テスト |
| 多言語検索が弱い | モデルが多言語非対応 | largeモデルをテスト、多言語評価セット作成 |

## text-embedding-3-large を使わなくてもいいケース

全てのプロジェクトで最高性能embeddingが必要なわけではありません。

例えば：

- データ量が少なく、キーワード検索で十分な場合
- 管理画面など、検索精度にこだわらない用途
- 予算が限られていて、FAQなど単一言語が中心
- まだMVP段階で、実際のクエリデータがない

現実的には、まずsmall/large両方で実際のクエリを使って召喚率を比較し、必要ならアップグレードするのが良いでしょう。

## まとめ：RAGの成否は「検索」が半分

どんなに強力なチャットモデルでも、間違ったコンテキストを渡せば、真面目に間違った答えを返します。

`text-embedding-3-large` の価値は、意味ベースで資料を探せること。キーワードだけに頼る運任せから脱却できます。

本番RAGを作るなら、以下の順序で進めるのがおすすめです：

1. 実際のドキュメントを整理
2. 適切なチャンク分割
3. text-embedding-3-largeでベクトルインデックス作成
4. 実ユーザーの質問でtop5召喚率を評価
5. rerankを追加
6. プロンプトやチャットモデルを最適化

OpenAI互換SDKで [Crazyrouter embeddings API](https://docs.crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=embedding_i18n) を使えば、同じ`base_url`でembedding・rerank・チャットモデルをまとめて呼び出せるので、RAGパイプラインの構築がスムーズです。

## FAQ

### RAGナレッジベースには必ずtext-embedding-3-largeが必要ですか？

必須ではありません。高品質・多言語・本番用途ならおすすめですが、小規模ならコスト重視で他のembeddingモデルから始めてもOKです。

### チャンクサイズはどれくらいが最適？

決まった正解はありません。技術ドキュメントなら400-800トークン、FAQならもっと短めから始めて、実際のクエリで召喚率を見て調整しましょう。

### text-embedding-3-largeはpgvectorと組み合わせられますか？

はい。生成したベクトルをPostgreSQLのpgvector型に格納し、ベクトル類似度検索が可能です。

### embeddingで資料を見つけたのに、モデルが誤答するのはなぜ？

召喚内容にノイズが多い、プロンプトが制限されていない、モデルが出典を無視する、複数資料の統合推論が必要などが考えられます。rerankや出典制約を追加しましょう。

### RAGにrerankは必要？

本番運用なら推奨です。embeddingで高速召喚、rerankで精密順位付け。この組み合わせが最も安定します。

