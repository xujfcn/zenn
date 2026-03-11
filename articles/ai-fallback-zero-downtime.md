---
title: "AIモデル自動フォールバック実装ガイド：APIダウンタイムゼロを目指す"
emoji: "🛡️"
type: "tech"
topics: ["python", "ai", "api", "infrastructure", "reliability"]
published: true
---

# AIモデル自動フォールバック実装ガイド：APIダウンタイムゼロを目指す

「Claude APIが急に503を返して、本番環境が30分止まった…」

AI APIに依存するプロダクトを運用していると、こんな経験は珍しくありません。2026年現在、主要なAIプロバイダーでも月1〜2回のダウンタイムは起きています。

この記事では、**AIモデルの自動フォールバック（自動切り替え）** をPythonで実装して、ダウンタイムをゼロに近づける方法を解説します。

---

## なぜフォールバックが必要か

### 実際に起きた障害の例（2025-2026年）

| 日付 | プロバイダー | 障害内容 | ダウンタイム |
|------|------------|---------|------------|
| 2025-12 | OpenAI | API全面停止 | 約2時間 |
| 2026-01 | Anthropic | レート制限の大幅強化 | 約4時間 |
| 2026-02 | Google | Gemini APIエラー急増 | 約1時間 |

単一プロバイダーに依存していると、これらの障害がそのまま自社サービスの障害になります。

---

## 基本実装：シンプルなフォールバック

最もシンプルなフォールバック実装です：

```python
from openai import OpenAI
import time

client = OpenAI(
    api_key="your-gateway-key",
    base_url="https://crazyrouter.com/v1"
)

FALLBACK_CHAIN = [
    "claude-opus-4-6",      # メイン
    "gpt-5",                # フォールバック1
    "gemini-3-pro-preview", # フォールバック2
    "deepseek-v3.2",        # 最終手段（最安）
]

def ask_ai(prompt: str, system: str = None) -> dict:
    """フォールバック付きでAIに問い合わせ"""
    messages = []
    if system:
        messages.append({"role": "system", "content": system})
    messages.append({"role": "user", "content": prompt})
    
    for i, model in enumerate(FALLBACK_CHAIN):
        try:
            start = time.time()
            response = client.chat.completions.create(
                model=model,
                messages=messages,
                timeout=30
            )
            elapsed = time.time() - start
            
            return {
                "content": response.choices[0].message.content,
                "model": model,
                "latency": round(elapsed, 2),
                "fallback_level": i,  # 0=メイン, 1+=フォールバック
                "tokens": response.usage.total_tokens
            }
        except Exception as e:
            print(f"[{model}] 失敗: {e}")
            if i < len(FALLBACK_CHAIN) - 1:
                print(f"  → 次のモデルに切り替え: {FALLBACK_CHAIN[i+1]}")
            continue
    
    raise Exception("全てのモデルが応答不能")
```

使い方：

```python
result = ask_ai("Pythonでバブルソートを実装してください")
print(f"モデル: {result['model']}")
print(f"フォールバックレベル: {result['fallback_level']}")
print(f"応答時間: {result['latency']}秒")
print(result['content'])
```

---

## 中級実装：タスクベースのルーティング

タスクの種類に応じて最適なフォールバック順序を変える実装：

```python
TASK_ROUTES = {
    "coding": {
        "chain": ["claude-sonnet-4-6", "gpt-5.2", "deepseek-v3.2"],
        "max_tokens": 4000,
        "temperature": 0
    },
    "creative": {
        "chain": ["claude-opus-4-6", "gpt-5", "gemini-3-pro-preview"],
        "max_tokens": 2000,
        "temperature": 0.8
    },
    "analysis": {
        "chain": ["claude-opus-4-6", "gemini-3-pro-preview", "gpt-5"],
        "max_tokens": 3000,
        "temperature": 0.2
    },
    "translation": {
        "chain": ["gpt-5", "claude-sonnet-4-6", "deepseek-v3.2"],
        "max_tokens": 2000,
        "temperature": 0.3
    },
    "cheap": {
        "chain": ["deepseek-v3.2", "gpt-5-mini", "gemini-2.5-flash"],
        "max_tokens": 1000,
        "temperature": 0.5
    }
}

def ask_ai_smart(prompt: str, task: str = "analysis", system: str = None) -> dict:
    """タスクタイプに応じたスマートルーティング"""
    route = TASK_ROUTES.get(task, TASK_ROUTES["analysis"])
    
    messages = []
    if system:
        messages.append({"role": "system", "content": system})
    messages.append({"role": "user", "content": prompt})
    
    for i, model in enumerate(route["chain"]):
        try:
            response = client.chat.completions.create(
                model=model,
                messages=messages,
                max_tokens=route["max_tokens"],
                temperature=route["temperature"],
                timeout=30
            )
            return {
                "content": response.choices[0].message.content,
                "model": model,
                "task": task,
                "fallback_level": i
            }
        except Exception as e:
            print(f"[{task}][{model}] 失敗: {e}")
            continue
    
    raise Exception(f"タスク '{task}' の全モデルが応答不能")
```

使い方：

```python
# コーディングタスク → Claude Sonnet → GPT-5.2 → DeepSeek
code = ask_ai_smart("FastAPIでCRUDアプリを作って", task="coding")

# 翻訳タスク → GPT-5 → Claude → DeepSeek
translation = ask_ai_smart("この文章を英語に翻訳して：...", task="translation")

# コスト重視 → DeepSeek → GPT-5 Mini → Gemini Flash
cheap = ask_ai_smart("こんにちはと返して", task="cheap")
```

---

## 上級実装：サーキットブレーカー

特定のモデルが連続で失敗した場合、一定時間スキップする「サーキットブレーカー」パターン：

```python
from collections import defaultdict
import time

class CircuitBreaker:
    def __init__(self, failure_threshold=3, recovery_time=300):
        self.failures = defaultdict(int)
        self.last_failure = defaultdict(float)
        self.failure_threshold = failure_threshold
        self.recovery_time = recovery_time  # 秒
    
    def is_open(self, model: str) -> bool:
        """サーキットが開いている（=使用不可）かチェック"""
        if self.failures[model] >= self.failure_threshold:
            elapsed = time.time() - self.last_failure[model]
            if elapsed < self.recovery_time:
                return True  # まだ回復待ち
            else:
                # リカバリー時間経過 → リセット
                self.failures[model] = 0
                return False
        return False
    
    def record_failure(self, model: str):
        self.failures[model] += 1
        self.last_failure[model] = time.time()
    
    def record_success(self, model: str):
        self.failures[model] = 0

# グローバルインスタンス
breaker = CircuitBreaker(failure_threshold=3, recovery_time=300)

def ask_ai_resilient(prompt: str, system: str = None) -> dict:
    """サーキットブレーカー付きフォールバック"""
    messages = []
    if system:
        messages.append({"role": "system", "content": system})
    messages.append({"role": "user", "content": prompt})
    
    for model in FALLBACK_CHAIN:
        if breaker.is_open(model):
            print(f"[{model}] サーキット開放中、スキップ")
            continue
        
        try:
            response = client.chat.completions.create(
                model=model,
                messages=messages,
                timeout=30
            )
            breaker.record_success(model)
            return {
                "content": response.choices[0].message.content,
                "model": model
            }
        except Exception as e:
            breaker.record_failure(model)
            print(f"[{model}] 失敗 ({breaker.failures[model]}/{breaker.failure_threshold}): {e}")
            continue
    
    raise Exception("利用可能なモデルがありません")
```

---

## 本番運用のベストプラクティス

### 1. ログを残す

フォールバックが発生したら必ずログを記録しましょう：

```python
import logging

logger = logging.getLogger("ai_fallback")

def log_fallback(primary: str, fallback: str, error: str):
    logger.warning(f"Fallback: {primary} → {fallback} | Error: {error}")
```

### 2. メトリクスを計測

どのモデルがどれくらい失敗しているか追跡：

```python
from collections import Counter

metrics = {
    "requests": Counter(),
    "failures": Counter(),
    "fallbacks": 0
}
```

### 3. アラートを設定

フォールバック率が一定以上になったら通知：

```python
def check_health():
    total = sum(metrics["requests"].values())
    if total > 0 and metrics["fallbacks"] / total > 0.1:
        # フォールバック率10%超え → アラート
        send_alert("AI APIフォールバック率が10%を超えています")
```

---

## APIゲートウェイを使うメリット

上記のコードでは、すべてのモデルを **1つのAPIエンドポイント** で呼び出しています。これはAPIゲートウェイを使っているからです。

ゲートウェイなしの場合：
- OpenAI用、Anthropic用、Google用にそれぞれSDKとAPIキーが必要
- フォールバック実装が複雑になる

ゲートウェイありの場合：
- 1つのSDK（OpenAI SDK）で全モデル対応
- `model=` を変えるだけでプロバイダーをまたぐフォールバックが可能

---

## まとめ

| 実装レベル | 機能 | おすすめシーン |
|-----------|------|-------------|
| 基本 | 順番にフォールバック | 個人プロジェクト |
| 中級 | タスクベースルーティング | チーム開発 |
| 上級 | サーキットブレーカー | 本番プロダクト |

AI APIに100%の可用性は期待できません。でも、フォールバックを実装すれば **体感99.9%の可用性** は実現できます。

---

## FAQ

### Q1: フォールバック時にレスポンスの品質は落ちますか？

モデルによって得意不得意はありますが、主要モデル間ならほとんどのタスクで品質差は許容範囲です。重要なのは「回答なし」を避けることです。

### Q2: フォールバック先のモデルの料金が高い場合は？

FALLBACK_CHAINを安い順に並べることで回避できます。コスト重視なら `deepseek-v3.2` → `gpt-5-mini` → `gemini-2.5-flash` の順がおすすめです。

### Q3: ストリーミングレスポンスでもフォールバックは使えますか？

はい。ストリーミング開始前にエラーが出れば次のモデルに切り替わります。ストリーミング中の切断にはリトライロジックを追加してください。

### Q4: どのAPIゲートウェイがフォールバックに適していますか？

OpenAI SDK互換であれば何でも使えます。Crazyrouter は627以上のモデルに対応しているので、フォールバックチェーンの選択肢が最も多いです。[価格表はこちら](https://crazyrouter.com/pricing?utm_source=zenn&utm_medium=article&utm_campaign=ja_fallback)。

### Q5: テスト方法は？

存在しないモデル名を指定すればエラーが発生するので、フォールバック動作をテストできます：

```python
FALLBACK_CHAIN = ["nonexistent-model", "gpt-5"]
result = ask_ai("テスト")  # → gpt-5にフォールバック
```

---

*この記事が参考になったら「いいね」をお願いします！*
