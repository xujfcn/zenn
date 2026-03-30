---
title: "トークンとバイトの違いとは？AIが実際にテキストを処理する仕組みを徹底解説"
emoji: "🔤"
type: "tech"
topics: ["ai", "llm", "tokenization", "bpe"]
published: true
---

GPT-5に「你好 Hello」と入力すると、モデルが実際に処理するのは**2トークン**です。文字数ではなくトークン数で課金されます。

一方、コンピュータ上では同じテキストが**12バイト**として保存されています。

バイト・文字・トークンの違いは何でしょうか？なぜAIは生のバイトではなくトークンを使うのでしょうか？

## バイトとは何か？

**バイト**はコンピュータがデータを保存する最小単位です。1バイト = 8ビット = 0〜255の数値。

**UTF-8**エンコーディングでの各文字のバイト数：

| 文字 | UTF-8バイト | バイト数 | Hex |
|------|-------------|----------|-----|
| `H` | 72 | 1 | 48 |
| `你` | 228, 189, 160 | 3 | e4 bd a0 |
| `🚀` | 240, 159, 154, 128 | 4 | f0 9f 9a 80 |

- **英字**：各1バイト
- **中国語・日本語・韓国語**：各3バイト
- **絵文字**：各4バイト

## 4つの階層：バイト → 文字 → 単語 → トークン

| レベル | "Hello, World" | 個数 | 説明 |
|--------|---------------|------|------|
| **バイト** | `48 65 6c 6c 6f 2c 20 57 6f 72 6c 64` | 12 | 生のストレージ |
| **文字** | `H e l l o , ␣ W o r l d` | 12 | 人間が読める文字 |
| **単語** | `Hello, World` | 2 | スペース区切り |
| **トークン** | `Hello` `, ` `World` | 3 | AIが処理する単位 |

**トークンはバイトでも文字でも単語でもありません。** 語彙サイズとシーケンス長のバランスを取るサブワード単位です。

## なぜバイトや単語を使わないのか？

### バイトの問題：シーケンスが長すぎる

Transformerのアテンション計算コストはO(n²)。バイトではシーケンスが3〜4倍長くなり、処理コストが爆発します。

### 単語の問題：語彙が爆発する

英語だけで17万語以上。技術用語やURLを加えると数百万語に。OOV（未知語）問題も発生します。

### トークン：最適なバランス

```
"unbelievable" → ["un", "bel", "ievable"]    (3トークン)
"tokenization" → ["Token", "ization"]         (2トークン)
"Hello"        → ["Hello"]                    (1トークン)
```

## BPEトークナイゼーションの仕組み

**Byte Pair Encoding（BPE）**のステップ：

1. テキストを1文字ずつに分割
2. 最も頻出する隣接ペアをカウント
3. そのペアを新トークンとしてマージ
4. 目標語彙サイズまで繰り返し

### tiktokenで確認

```python
import tiktoken

enc = tiktoken.get_encoding("o200k_base")
text = "你好 Hello"
tokens = enc.encode(text)
token_strings = [enc.decode([t]) for t in tokens]

print(f"Text: {text}")
print(f"UTF-8 bytes: {len(text.encode('utf-8'))}")
print(f"Tokens ({len(tokens)}): {token_strings}")
```

結果：12バイトがたった2トークンに！

## モデルごとに異なるトークナイザー

| テキスト | cl100k_base (GPT-4) | o200k_base (GPT-4o/5) | UTF-8バイト |
|----------|---------------------|----------------------|-------------|
| Hello, how are you today? | 7 | 7 | 25 |
| 你好，请用中文解释一下什么是token | 15 | 9 | 47 |
| こんにちは、トークンとは何ですか？ | 12 | 10 | 51 |

GPT-5は中国語・日本語で**40%効率的**です。

## 言語別トークン効率

| 言語 | テキスト | トークン | バイト | バイト/トークン |
|------|---------|---------|--------|---------------|
| 英語 | "Hello, how are you today?" | 7 | 25 | 3.6 |
| 中国語 | "你好，今天怎么样？" | 5 | 27 | 5.4 |
| 日本語 | "こんにちは" | 1 | 15 | 15.0 |
| 韓国語 | "안녕하세요" | 2 | 15 | 7.5 |

日本語は最もトークン効率が高い言語です！

### モデル比較のすすめ

```python
from openai import OpenAI

client = OpenAI(
    api_key="your-crazyrouter-key",
    base_url="https://crazyrouter.com/v1"
)

for model in ["gpt-5", "gpt-5-mini", "deepseek-v3.2", "claude-sonnet-4"]:
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": "Explain tokenization in 2 sentences"}],
        max_tokens=100
    )
    usage = response.usage
    print(f"{model}: {usage.prompt_tokens} in / {usage.completion_tokens} out")
```

[Crazyrouter](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=tokens_vs_bytes)なら、1つのAPIキーで627以上のモデルにアクセスできます。

## クイックリファレンス

| 属性 | バイト | 文字 | 単語 | トークン |
|------|--------|------|------|---------| 
| **定義** | 生のストレージ | 人間が読める | スペース区切り | AI用サブワード |
| **"Hello"** | 5 | 5 | 1 | 1 |
| **"你好"** | 6 | 2 | 1 | 1 |
| **AI課金基準？** | いいえ | いいえ | いいえ | **はい** |

### 換算目安：
- 1英語トークン ≈ 4文字 ≈ 0.75語
- 1,000トークン ≈ 英語750語

---

*完全版はこちら：[Crazyrouter Blog](https://crazyrouter.com/ja/blog/ja-tokens-vs-bytes-ai-tokenization-explained?utm_source=zenn&utm_medium=article&utm_campaign=tokens_vs_bytes)*
