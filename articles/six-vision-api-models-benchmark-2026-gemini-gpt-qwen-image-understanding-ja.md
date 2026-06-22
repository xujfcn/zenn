---
title: "6つのVision APIモデルを実測：Gemini 2.5、GPT-4.1、Qwen3 VLの画像理解はどう選ぶべきか"
emoji: "👁️"
type: "tech"
topics: [ai, api, llm, vision]
published: true
---

# 6つのVision APIモデルを実測：Gemini 2.5、GPT-4.1、Qwen3 VLの画像理解はどう選ぶべきか

画像理解APIをプロダクトに入れるとき、「画像対応モデルです」という説明だけでは足りません。

モデルページには vision input 対応と書かれていても、実運用では次のような点を確認する必要があります。

- OpenAI互換の `image_url` payload が本当にモデルまで届いているか
- HTTP 200 が返っても、モデルが画像を見たとは限らない
- ユーザーが待つ画面で十分に速いか
- バッチ処理で十分に安いか
- 失敗時に fallback しやすいか
- usage metadata から壊れた media path を検知できるか

今回は、同じOpenAI互換API形式で6つのvision-capableモデルを比較しました。

- `gemini-2.5-flash`
- `gemini-2.5-flash-lite`
- `gpt-4.1-mini`
- `gpt-4.1-nano`
- `qwen3-vl-flash`
- `qwen3-vl-plus`

目的は「絶対的な最強モデル」を決めることではありません。

より実用的な問いは、**どのユーザーworkflowにどのモデルをroutingすべきか**です。

---

## テスト方法

すべてのリクエストは Crazyrouter の OpenAI互換 Base URL を使いました。

```text
https://cn.crazyrouter.com/v1
```

API形式は `chat/completions` で、画像は `messages[].content[]` の中に `image_url` として渡しています。

テスト画像は2つです。

- Python logo
- GitHub logo

各モデルで各画像を3回ずつ処理しました。つまり、1モデルあたり合計6リクエストです。

テスト時刻：`2026-06-21T13:36:32Z`

これは **Vision API smoke test** です。`image_url` 経路が正常に動くか、モデルが簡単な画像認識をできるかを見るためのテストです。OCR、図表理解、文書抽出、手書き文字、医療画像などの完全な評価ではありません。

---

## 先に結論

今回の結果から見ると、実用上のおすすめは次の通りです。

- **リアルタイムのユーザー画像アップロード**：`gpt-4.1-mini`
- **大量のlogo / icon / 簡単な画像分類**：`qwen3-vl-flash`
- **低コストなGeminiルート**：`gemini-2.5-flash-lite`
- **低コストなOpenAI系ルート**：`gpt-4.1-nano`
- **Qwen VLの品質重視アップグレード**：`qwen3-vl-plus`
- **今回のimage_url経路ではデフォルト非推奨**：`gemini-2.5-flash`

一番重要な発見はこれです。

> **HTTP 200 は、画像理解が成功した証明ではありません。**

今回 `gemini-2.5-flash` は6回すべてHTTP上は成功しましたが、画像認識は **0/6** でした。「画像が提供されていない」という応答や、GitHub logoを別のロゴとして認識する出力もありました。

これはproductionでかなり危険な失敗パターンです。API呼び出しは成功しているように見えるのに、モデルは画像を正しく処理していないからです。

---

## 全体結果

| モデル | HTTP成功 | 正しく認識 | no-image応答 | 平均レイテンシ | 最遅リクエスト | 10k回相当の概算コスト | 向いている用途 |
|---|---:|---:|---:|---:|---:|---:|---|
| `qwen3-vl-flash` | 6/6 | 6/6 | 0 | 3.819s | 5.975s | $0.0915 | 低コストな大量認識 |
| `gpt-4.1-mini` | 6/6 | 6/6 | 0 | 1.491s | 2.189s | $0.5226 | ユーザー向けリアルタイム機能 |
| `gpt-4.1-nano` | 6/6 | 6/6 | 0 | 2.863s | 4.213s | $0.1666 | 低コストなOpenAI系fallback |
| `qwen3-vl-plus` | 6/6 | 6/6 | 0 | 3.859s | 4.821s | $0.3848 | Qwen系の品質重視ルート |
| `gemini-2.5-flash` | 6/6 | 0/6 | 1 | 4.965s | 9.507s | $0.6168 | 今回のimage_url経路では失敗 |
| `gemini-2.5-flash-lite` | 6/6 | 6/6 | 0 | 2.618s | 4.195s | $0.5466 | 低コストなGeminiルート |

10k回相当の概算コストは、今回のlogo認識テストで観測されたusageに基づくものです。大きな画像、OCR、長い説明、multi-image promptでは実際のコストは変わります。

重要なのは、モデルの単価だけではなく **cost per successful image task** を見ることです。

安いモデルでも、retryやfallbackが頻発するならproductionでは安くありません。

---

## Accuracy：5モデルは成功、1モデルは失敗

今回のsmoke testでの正答率は以下です。

1. `qwen3-vl-flash`: 6/6
2. `gpt-4.1-mini`: 6/6
3. `gpt-4.1-nano`: 6/6
4. `qwen3-vl-plus`: 6/6
5. `gemini-2.5-flash-lite`: 6/6
6. `gemini-2.5-flash`: 0/6

簡単なlogo / icon recognitionでは、6つ中5つのルートが正しく動きました。つまり、基本的な画像分類であれば、軽量モデルでも十分なケースがあります。

ただし `gemini-2.5-flash` の結果は重要です。HTTP success は image path が正常であることを意味しません。

---

## Latency：最速はGPT-4.1 Mini

平均レイテンシの低い順です。

1. `gpt-4.1-mini`: avg 1.491s, median 1.292s, slowest 2.189s
2. `gemini-2.5-flash-lite`: avg 2.618s, median 2.627s, slowest 4.195s
3. `gpt-4.1-nano`: avg 2.863s, median 2.562s, slowest 4.213s
4. `qwen3-vl-flash`: avg 3.819s, median 3.493s, slowest 5.975s
5. `qwen3-vl-plus`: avg 3.859s, median 3.729s, slowest 4.821s
6. `gemini-2.5-flash`: avg 4.965s, median 4.333s, slowest 9.507s

ユーザーが画面上で待つ機能では、レイテンシはプロダクト品質の一部です。画像をアップロードして返答を待つ場合、1〜2秒の差は体感に影響します。

そのようなworkflowでは、今回 `gpt-4.1-mini` が最も強いdefault routeでした。

---

## Cost：成功した中で最安はQwen3 VL Flash

10,000回のtest-style callsでの概算コストは以下です。

1. `qwen3-vl-flash`: 約 $0.0915
2. `gpt-4.1-nano`: 約 $0.1666
3. `qwen3-vl-plus`: 約 $0.3848
4. `gpt-4.1-mini`: 約 $0.5226
5. `gemini-2.5-flash-lite`: 約 $0.5466
6. `gemini-2.5-flash`: 約 $0.6168

大量のlogo detection、icon classification、screenshot pre-filtering、dataset taggingでは、`qwen3-vl-flash` が有力です。

安いだけではなく、visual smoke testも6/6で通っている点が重要です。

---

## モデル別メモ

### GPT-4.1 Mini

リアルタイム用途向きです。

平均レイテンシが最も低く、認識も6/6で成功しました。

ユーザーが結果を待つ画像アップロード、サポート用スクリーンショット分析、画像入力つきチャット、低レイテンシが必要なagent workflowに向いています。

### Qwen3 VL Flash

大量・低コストの画像認識に向いています。

6/6で認識に成功し、概算コストも最も低い結果でした。

logo recognition、icon detection、simple image classification、screenshot pre-classificationなどに向いています。

### GPT-4.1 Nano

低コストなOpenAI系fallbackとして使いやすい候補です。

今回の簡単な認識テストは6/6で成功しました。ただし、複雑なOCRやdocument understandingまで任せる前提にはしない方が安全です。

### Qwen3 VL Plus

Qwen系のquality-oriented upgrade routeです。

flashより高価で、最速でもありませんが、`qwen3-vl-flash` で足りない場合の昇格先として使えます。

### Gemini 2.5 Flash Lite

低コストなGemini系ルートとしては使えます。

今回6/6で認識に成功しました。ただしusage metadataはQwenほど分かりやすくないため、productionではvisual smoke testを残すべきです。

### Gemini 2.5 Flash

今回の構成ではdefaultにしない方がよいルートです。

HTTPは成功しましたが、画像認識smoke testは失敗しました。

これはモデル能力そのものではなく、adapter、media-fetch、payload-conversion、upstream routingなどの問題かもしれません。

ただしproduction routingでは結論は同じです。自分の環境で画像経路が正常と確認できるまで、default routeにはしない方が安全です。

---

## シナリオ別routing

| シナリオ | Default route | Fallback | 理由 |
|---|---|---|---|
| リアルタイムの画像アップロード | `gpt-4.1-mini` | `qwen3-vl-flash` / `gemini-2.5-flash-lite` | レイテンシと信頼性を優先 |
| 大量のlogo / icon認識 | `qwen3-vl-flash` | `gpt-4.1-nano` | 成功したルートの中で低コスト |
| 簡単なスクリーンショット分類 | `qwen3-vl-flash` / `gpt-4.1-nano` | `gpt-4.1-mini` | まず安く、難しいケースだけ昇格 |
| サポート用スクリーンショット分析 | `gpt-4.1-mini` | `qwen3-vl-plus` | ユーザーが待つため低レイテンシが重要 |
| OCR / 文書前処理 | 別途benchmarkが必要 | OCR/Document向けモデル | logo testではOCR品質は分からない |
| Agentの視覚入力 | `gpt-4.1-mini` / `qwen3-vl-flash` | smoke test + fallback | Agentは誤った視覚入力を後続処理に伝播させる |
| Gemini backup route | `gemini-2.5-flash-lite` | `gpt-4.1-nano` | Flash Liteは成功、Flashは今回失敗 |

重要なのは、1つの万能モデルを探すことではありません。

タスクごとにrouteを分けることです。

---

## Visual smoke testを入れる

Vision APIでは、テキストだけのhealth checkでは不十分です。

実用的な監視では、以下を確認します。

1. 固定の小さな画像を送る
2. 答えが決まっている質問をする
3. 期待するキーワードを確認する
4. “no image provided”系の応答を失敗扱いにする
5. 不自然なusage metadataを確認する
6. レイテンシの急上昇を監視する

簡単な例です。

```python
def is_vision_route_healthy(response):
    text = response.choices[0].message.content.lower()

    if "no image" in text or "image provided" in text:
        return False

    if "python" not in text:
        return False

    usage = getattr(response, "usage", None)
    if usage is None:
        return False

    return True
```

実際のシステムでは、モデルごとの表現揺れ、usageの違い、retry、fallback policyを考慮する必要があります。

しかし原則は同じです。

> HTTP 200を画像理解成功の証拠にしない。

---

## API example

今回使ったOpenAI互換のリクエスト例です。

```python
from openai import OpenAI

client = OpenAI(
    api_key="YOUR_API_KEY",
    base_url="https://cn.crazyrouter.com/v1"
)

response = client.chat.completions.create(
    model="qwen3-vl-flash",
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "Identify the main logo or object in this image."},
            {
                "type": "image_url",
                "image_url": {
                    "url": "https://raw.githubusercontent.com/github/explore/main/topics/python/python.png",
                    "detail": "low"
                }
            }
        ]
    }],
    max_tokens=40,
    temperature=0,
)

print(response.choices[0].message.content)
```

API endpointにはUTMパラメータを付けないでください。UTMは人がクリックするリンクに付けるもので、SDKの `base_url` には不要です。

---

## まとめ

Vision APIの選定は、モデル名ではなくworkflowで決めるべきです。

- リアルタイム用途では、正しい認識と低レイテンシを優先
- バッチ分類では、cost per successful imageを優先
- Agent用途では、監視とfallbackを優先
- OCRや文書理解では、実データで別benchmarkを実施

今回の実用的な順位は次の通りです。

1. リアルタイム用途のdefault: `gpt-4.1-mini`
2. バッチ低コスト認識のdefault: `qwen3-vl-flash`
3. 低コストGemini backup: `gemini-2.5-flash-lite`
4. 低コストOpenAI backup: `gpt-4.1-nano`
5. Qwen quality upgrade route: `qwen3-vl-plus`
6. 今回はdefault非推奨: `gemini-2.5-flash`

大事なのは「このモデルは画像対応か？」ではありません。

より重要なのは次の問いです。

> このproduction routeは、画像を安定してモデルまで届けられるか？

本番投入前に、ここを必ず検証するべきです。