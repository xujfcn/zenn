---
title: "text-embedding-3-large は使うべき？small とのコスト・品質・選び方"
emoji: "🧠"
type: "tech"
topics: [ai, rag, embeddings, api]
published: true
---

# text-embedding-3-large は使うべき？small とのコスト・品質・選び方

RAG やセマンティック検索を導入する際、多くの開発者が悩むのがこの問題です。

> 結局、`text-embedding-3-large` を使うべきか、それとも安価な `text-embedding-3-small` で十分なのか？

答えは「large が常に最良」でも「small で十分」でもありません。

より正確に言うと、**検索品質がビジネス成果に直結するなら large をテストする価値あり。プロジェクト初期やデータ量が膨大な場合は small から始めるのが堅実です。**

![embedding model cost and quality comparison dashboard](https://raw.githubusercontent.com/xujfcn/images/main/blog/posts/embedding-cost-quality-dashboard.webp)

この記事では、実際のプロジェクト視点で embedding モデル選定のポイントを解説します。

## まず結論：どう選ぶ？

この表を参考にしてください。

| シーン | 推奨スタート | 理由 |
|---|---|---|
| 企業ナレッジベースQA | text-embedding-3-large | 検索品質が最重要 |
| 多言語RAG | text-embedding-3-large | 非英語・クロス言語検索の精度向上 |
| カスタマーサポートBot | large/smallでA/Bテスト | 誤回答のコスト次第 |
| 社内ツール検索 | text-embedding-3-small | コスト優先・許容度高め |
| MVP / デモ | text-embedding-3-small | まずは動作確認 |
| 大規模ドキュメントインデックス | small または large の次元削減 | ストレージ・検索コスト抑制 |
| コード/技術ドキュメント検索 | large + rerank | セマンティック＆精度両立 |

一言でまとめるなら、

**まず small でベースラインを作り、large で実際のクエリを評価してみましょう。**

## text-embedding-3-large の強み

`text-embedding-3-large` は高性能な embedding モデルです。

OpenAI公式ドキュメントによると、デフォルトで3072次元ベクトルを出力し、最大入力は8192トークン。英語・非英語問わず高精度な埋め込みを狙ったモデルです。

主な強みは以下の通りです。

1. より強力なセマンティック表現
2. 複雑なクエリへの対応力
3. 多言語・クロス言語検索に強い
4. 長文ドキュメントにも対応しやすい
5. 本番RAG用途の有力候補

ただし、コスト面では

- 1トークンあたりのAPIコストが高い
- デフォルトのベクトル次元数が大きい
- ベクトルDBのストレージコスト増
- 検索時のメモリ・インデックス負荷増

といったデメリットもあります。

つまり、large を選ぶなら「品質向上が追加コストを上回る」ことが前提です。

## text-embedding-3-small が向いているケース

`text-embedding-3-small` はコスト効率重視のモデルです。

向いている用途は

- FAQ検索
- 小規模ナレッジベース
- 初期MVP
- 社内検索ツール
- 単一言語のコンテンツ
- 検索ミスにある程度寛容な場面

多くのプロジェクトは、最初から large を使う必要はありません。

特に、実際のユーザーの質問や評価データ、フィードバックがまだ無い段階では、「大きい embedding の方が安心」と感じても、その効果を証明できません。

## コストはAPI料金だけじゃない

Embedding のコストは API の利用料だけではありません。

他にも以下のコストが発生します。

| コスト項目 | 影響要因 |
|---|---|
| API利用料 | 入力トークン数・再インデックス頻度 |
| ベクトルDBストレージ | ドキュメント数・チャンク数・ベクトル次元 |
| 検索レイテンシ | インデックス規模・次元数・top_k |
| メモリ/ディスク | ベクトル数・精度 |
| 運用コスト | インデックス再構築・バージョン移行・評価 |

例えば、

100万チャンク、1ベクトル3072次元、float32で保存した場合、ベクトルデータだけで

```text
1,000,000 × 3072 × 4 bytes ≈ 12.3 GB
```

となります。

1536次元にすれば約半分です。

これにインデックス構造やメタデータ、DBのオーバーヘッドも加わります。

大規模運用では `dimensions` パラメータとベクトルDBコストに要注意です。

## dimensions パラメータの使い方

OpenAI 第三世代 embedding モデルは `dimensions` で出力ベクトルの次元数を指定できます。

例：

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

これは本番環境で非常に役立ちます。

例えば

- large 3072次元
- large 1536次元
- large 1024次元
- small デフォルト次元

などでテストし、top 5 のリコール率・レイテンシ・コストを比較しましょう。

## MTEBスコアだけで選ばない

公開ベンチマークは参考になりますが、それが最終判断基準ではありません。

実際のビジネスデータはベンチマークと大きく異なることが多いです。

- ユーザーの質問が短い
- ドキュメントが日英混在
- 製品名やエラーコードが多い
- 表やパラメータ説明が多い
- 口語表現が多用される

自分のデータで小さな評価セットを作るのがベストです。

シンプルなフォーマット例：

| クエリ | 正解ドキュメント | 種別 |
|---|---|---|
| 残高の確認方法は？ | billing.md | FAQ |
| base_url には何を入れる？ | quickstart.md | technical |
| Claude Code の設定方法は？ | claude-code.md | integration |
| API呼び出しで401エラーが出た場合？ | auth-errors.md | troubleshooting |

各モデルで top 3 / top 5 に正解が入るか確認しましょう。

## 実践的なA/Bテスト手順

おすすめの評価フローは以下です。

### 1. 50〜200件の実クエリを用意

自作ではなく、できるだけ

- サイト内検索ログ
- サポート問い合わせ
- ユーザーコミュニティの質問
- チケットタイトル
- ドキュメントコメント

などから収集しましょう。

### 2. 正解ドキュメントをアノテーション

各クエリに対し、1〜3件の正解チャンクやドキュメントを紐付けます。

### 3. 複数のインデックスを作成

例：

- index_small_default
- index_large_3072
- index_large_1536

### 4. リコール評価を実施

主な指標：

| 指標 | 意味 |
|---|---|
| Recall@3 | 上位3件に正解が含まれるか |
| Recall@5 | 上位5件に正解が含まれるか |
| MRR | 正解が上位ほど高評価 |
| latency | クエリ応答速度 |
| cost | インデックス・検索コスト |

### 5. large へのアップグレード判断

large で1%しか改善しないのにコストが大幅増なら、無理に使う必要はありません。

逆に、Recall@5 が78%→90%に上がり、ビジネス的に精度が重要なら、large を選ぶ価値は十分あります。

## RAG の品質問題＝embedding モデルのせいとは限らない

large に変えても改善しない場合、他に原因があることが多いです。

| 問題 | 症状 | 優先対応 |
|---|---|---|
| chunk 切り方が悪い | コンテキストが不完全 | 再チャンク |
| metadata 不足 | 言語・権限・製品で絞れない | metadata追加 |
| クエリが短すぎる | 「課金問題」などで不安定 | クエリリライト |
| ドキュメントが古い | 古い情報がヒットする | 更新日・バージョンで絞る |
| rerank 未導入 | 上位が似てるだけで不正確 | rerank追加 |
| プロンプトが緩い | モデルが常識で回答 | 情報ソース限定プロンプト |

まずは RAG パイプライン全体の品質を見直しましょう。

## OpenAI互換APIでembeddingモデルを切り替える

OpenAI SDK を使っている場合、モデル切り替えは簡単です。

```python
from openai import OpenAI

client = OpenAI(
    api_key="your-api-key",
    base_url="https://crazyrouter.com/v1"
)

# コスト重視
small = client.embeddings.create(
    model="text-embedding-3-small",
    input="semantic search for AI documentation"
)

# 品質重視
large = client.embeddings.create(
    model="text-embedding-3-large",
    input="semantic search for AI documentation"
)
```

Crazyrouter のような OpenAI 互換ゲートウェイを使えば、SDKはそのままでモデル名と base URL を変えるだけでOKです。

まずは [Crazyrouter Playground](https://crazyrouter.com/playground?utm_source=zenn&utm_medium=article&utm_campaign=embedding_i18n) で動作確認し、同じパラメータをサーバー側に適用しましょう。

## 推奨プロジェクトロードマップ

### フェーズ1：MVP

- text-embedding-3-small を利用
- ローカル or pgvector で保存
- top 5 検索
- rerank など複雑な処理は省略
- まずは動作することが目標

### フェーズ2：実データ評価

- ユーザークエリを収集
- 正解ドキュメントをアノテーション
- small vs large で比較
- chunk戦略を調整
- ボトルネック特定が目標

### フェーズ3：本番最適化

- large または large の次元削減版を利用
- ハイブリッド検索導入
- rerank追加
- 権限フィルタ追加
- 情報ソースの引用表示
- 安定性・説明性・コスト制御が目標

## 結論：large はデフォルトではないが、真剣にテストする価値あり

`text-embedding-3-large` の強みは高いセマンティック表現力。高品質RAG、多言語ナレッジベース、複雑な検索には特に有効です。

ただし、最強モデルを盲目的に選ぶのは避けましょう。

おすすめの進め方は

1. small でコスト・品質のベースラインを作る
2. 実クエリで large の効果を評価
3. dimensions でベクトル次元を調整
4. rerank やハイブリッド検索も活用（モデル変更だけに頼らない）
5. ビジネス上の誤答コストでアップグレード判断

RAG の結果がユーザー体験や売上に直結するなら、`text-embedding-3-large` は十分検討に値します。

一方、社内ツールや初期検証なら small から始めるのが現実的です。

## FAQ

### text-embedding-3-large は text-embedding-3-small より必ず優れている？

基本的に性能は上ですが、すべてのプロジェクトでコストに見合うとは限りません。実クエリでのリコール向上がコストを上回るかで判断しましょう。

### dimensions を下げると品質は落ちる？

影響する可能性があります。次元削減でストレージ・検索コストは下がりますが、リコール精度が下がる場合も。自分の評価セットで検証をおすすめします。

### RAG プロジェクトはモデル最適化とチャンク戦略、どちらを優先すべき？

まずはチャンク戦略と評価セットの整備を。チャンクが悪いと、どんなに強い embedding でも効果が限定的です。

### 多言語ナレッジベースは text-embedding-3-large が向いている？

優先的にテストする価値があります。公式にも large は英語・非英語両対応とされ、多言語検索ではセマンティック表現力が重要です。

### Crazyrouter で text-embedding-3-large を使える？

OpenAI互換の `/v1/embeddings` エンドポイント経由で利用可能です。コードの base URL を `https://crazyrouter.com/v1` に設定してください。

