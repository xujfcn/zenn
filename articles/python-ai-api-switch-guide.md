---
title: "PythonでAI APIを3行で切り替える方法：GPT-5・Claude・Gemini実測比較ガイド"
emoji: "🔄"
type: "tech"
topics: ["python", "ai", "openai", "claude", "gemini"]
published: true
---

## はじめに

「GPT-5を試したいけど、Claudeも気になる。Geminiも使ってみたい…」

こんな悩みを持つ開発者は多いはずです。でも、各社のSDKを個別にインストールして、APIキーを3つ管理して、コードを書き換えて…となると面倒ですよね。

この記事では、**たった3行のコード変更で複数のAIモデルを切り替える方法**を実際のコードで解説します。

## なぜモデル切り替えが必要なのか

2026年現在、主要なAIモデルにはそれぞれ得意分野があります：

| モデル | 得意分野 | 苦手分野 |
|--------|---------|---------|
| GPT-5 | 汎用タスク、関数呼び出し | 長文の一貫性 |
| Claude Opus 4.6 | 複雑な分析、コーディング | 画像生成 |
| Gemini 3 Pro | 長文処理（200K+）、マルチモーダル | 日本語の微妙なニュアンス |
| DeepSeek V3.2 | コスパ最強、推論 | 英語以外の精度 |

プロジェクトによって最適なモデルは異なります。だからこそ、**簡単に切り替えられる仕組み**が重要です。

## 方法：OpenAI SDK + APIゲートウェイ

ポイントは、**OpenAI互換のAPIゲートウェイ**を使うことです。ゲートウェイ経由なら、OpenAI SDKのコードをそのまま使って、モデル名を変えるだけで別のAIを呼び出せます。

### ステップ1：セットアップ（1分）

```bash
pip install openai
```

### ステップ2：クライアント初期化（変更するのはここだけ）

```python
from openai import OpenAI

client = OpenAI(
    api_key="your-gateway-key",          # ← ゲートウェイのAPIキー
    base_url="https://crazyrouter.com/v1" # ← ゲートウェイのURL
)
```

この2行だけで、627以上のモデルにアクセスできるようになります。

### ステップ3：モデルを切り替える（1行変更）

```python
# GPT-5を使う場合
response = client.chat.completions.create(
    model="gpt-5",
    messages=[{"role": "user", "content": "Pythonでフィボナッチ数列を実装してください"}]
)

# Claude Opus 4.6に切り替え → model名を変えるだけ！
response = client.chat.completions.create(
    model="claude-opus-4-6",
    messages=[{"role": "user", "content": "Pythonでフィボナッチ数列を実装してください"}]
)

# Gemini 3 Proに切り替え → 同じく1行だけ
response = client.chat.completions.create(
    model="gemini-3-pro-preview",
    messages=[{"role": "user", "content": "Pythonでフィボナッチ数列を実装してください"}]
)
```

**変更するのは `model=` の1行だけ。** SDKもAPIキーもエンドポイントもそのままです。

## 実践：モデル比較スクリプト

複数モデルを一括で比較するスクリプトを作ってみましょう：

```python
from openai import OpenAI
import time

client = OpenAI(
    api_key="your-gateway-key",
    base_url="https://crazyrouter.com/v1"
)

models = [
    "gpt-5",
    "claude-opus-4-6",
    "gemini-3-pro-preview",
    "deepseek-v3.2",
    "grok-4",
]

prompt = "Pythonで効率的なフィボナッチ数列の実装を3つ書いてください。それぞれの計算量も説明してください。"

for model_name in models:
    print(f"\n{'='*60}")
    print(f"モデル: {model_name}")
    print(f"{'='*60}")
    
    start = time.time()
    try:
        response = client.chat.completions.create(
            model=model_name,
            messages=[{"role": "user", "content": prompt}],
            max_tokens=1000
        )
        elapsed = time.time() - start
        
        print(f"応答時間: {elapsed:.2f}秒")
        print(f"トークン数: {response.usage.total_tokens}")
        print(f"\n{response.choices[0].message.content[:500]}...")
    except Exception as e:
        print(f"エラー: {e}")
```

このスクリプトを実行すると、5つのモデルの回答を一度に比較できます。

## 応用：環境変数でモデルを切り替え

本番環境では、環境変数でモデルを管理するのがベストプラクティスです：

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.getenv("AI_API_KEY"),
    base_url=os.getenv("AI_BASE_URL", "https://crazyrouter.com/v1")
)

# 環境変数でモデルを指定
MODEL = os.getenv("AI_MODEL", "gpt-5")

def ask_ai(prompt: str) -> str:
    response = client.chat.completions.create(
        model=MODEL,
        messages=[{"role": "user", "content": prompt}]
    )
    return response.choices[0].message.content
```

```bash
# 開発環境（安いモデル）
export AI_MODEL=deepseek-v3.2

# 本番環境（高精度モデル）
export AI_MODEL=claude-opus-4-6
```

コードを一切変更せずに、環境変数だけでモデルを切り替えられます。

## 応用：自動フォールバック

メインモデルがダウンした時に自動で別モデルに切り替える実装：

```python
def ask_with_fallback(prompt: str, models: list[str] = None) -> str:
    if models is None:
        models = ["claude-opus-4-6", "gpt-5", "deepseek-v3.2"]
    
    for model_name in models:
        try:
            response = client.chat.completions.create(
                model=model_name,
                messages=[{"role": "user", "content": prompt}],
                timeout=30
            )
            return response.choices[0].message.content
        except Exception as e:
            print(f"{model_name} failed: {e}, trying next...")
    
    raise Exception("All models failed")
```

## ストリーミング対応

リアルタイムで回答を表示したい場合：

```python
def stream_response(prompt: str, model: str = "gpt-5"):
    stream = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}],
        stream=True
    )
    
    for chunk in stream:
        if chunk.choices[0].delta.content:
            print(chunk.choices[0].delta.content, end="", flush=True)
    print()

# 使い方
stream_response("東京の観光スポットを5つ教えてください", model="gemini-3-pro-preview")
```

ストリーミングもモデル名を変えるだけで、どのモデルでも同じコードで動きます。

## 2026年モデル選択ガイド

| ユースケース | おすすめモデル | 理由 |
|-------------|-------------|------|
| 日常的なチャット | GPT-5 Mini | 安くて速い |
| コーディング | Claude Sonnet 4.6 / GPT-5.2 | コード品質が高い |
| 長文ドキュメント処理 | Gemini 2.5 Pro | 2Mトークンのコンテキスト |
| コスト最優先 | DeepSeek V3.2 | 圧倒的に安い |
| 複雑な推論 | Claude Opus 4.6 | 分析力が最強 |
| 画像生成 | DALL-E 3 / Flux Pro | 高品質な画像 |

## まとめ

1. **OpenAI互換のAPIゲートウェイ**を使えば、SDKは1つでOK
2. モデル切り替えは **`model=` の1行変更だけ**
3. 環境変数管理で**コード変更なしにモデル切り替え**
4. フォールバック実装で**可用性を向上**

APIキーを何個も管理する時代は終わりました。1つのキーで627以上のモデルを使い分けましょう。

## よくある質問

### Q1: APIゲートウェイを使うとレイテンシは増えますか？

追加レイテンシは通常10〜50ms程度です。LLMの応答自体が数秒かかるため、体感差はほぼありません。日本にノードがあるサービスを選べばさらに高速です。

### Q2: 既存のOpenAI SDKコードをそのまま使えますか？

はい。`base_url`と`api_key`を変更するだけで、既存コードがそのまま動きます。Function Calling、ストリーミング、Vision（画像認識）もすべて互換です。

### Q3: Anthropic SDKやGemini SDKも使えますか？

一部のゲートウェイは、OpenAI形式だけでなくAnthropic Messages APIやGemini Native APIにも対応しています。既存のAnthropic SDKコードもそのまま使えます。

### Q4: 無料で試せますか？

多くのゲートウェイが無料枠を提供しています。新規登録で体験クレジットがもらえるサービスもあります。

### Q5: データのセキュリティは大丈夫ですか？

ゲートウェイはリクエストを中継するだけで、データを保存しません。セキュリティが特に重要な場合は、LiteLLMなどのオープンソースゲートウェイを自前でホスティングすることも可能です。

---

*この記事が参考になったら「いいね」をお願いします。AI API関連の最新情報は [Crazyrouter Blog](https://crazyrouter.com/blog?utm_source=zenn&utm_medium=article&utm_campaign=ja_python_tutorial) で発信しています。*
