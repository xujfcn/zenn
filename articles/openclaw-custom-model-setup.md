---
title: "OpenClaw カスタムモデル設定ガイド：任意のAIモデルを接続する完全マニュアル"
emoji: "🔌"
type: "tech"
topics: ["openclaw", "ai", "llm", "selfhosted"]
published: true
---

OpenClawはオープンソースのAIアシスタントフレームワークとして、特定のモデルベンダーにロックインされない柔軟な**Provider体系**を持っています。OpenAI、Anthropic、Google、DeepSeekなどのクラウドモデルはもちろん、OllamaやLlamaなどのローカルモデルも接続可能。本記事ではOpenClaw カスタムモデルの設定方法を体系的に解説します。

## OpenClaw カスタムモデルの基本：Provider体系を理解する

### OpenClaw カスタムモデルの組み込みProvider一覧

OpenClawには以下のProviderが組み込まれています：

| Provider | 対応モデル | 認証方式 |
|----------|-----------|----------|
| `openai` | GPT-5, GPT-4o, o3 | API Key |
| `anthropic` | Claude Opus 4, Sonnet 4 | API Key |
| `google` | Gemini 3, 2.5 Pro | API Key |
| `deepseek` | DeepSeek V3, R1 | API Key |
| `openrouter` | 300+ モデル | API Key |
| `opencode` | OpenCode Zen | API Key |

### OpenClaw カスタムモデルのAPI Key設定と自動ローテーション

OpenClawのAPI Keyは環境変数で管理します。優先順位：

```bash
# 優先順位（高→低）
OPENCLAW_LIVE_OPENAI_KEY      # 最優先：ライブキー
OPENAI_API_KEYS=key1,key2,key3  # 複数キーローテーション
OPENAI_API_KEY                  # 標準的な単一キー
OPENAI_API_KEY_1, _2, _3       # 番号付き複数キー
```

重要：ローテーションは`429 Rate Limit`エラー時のみ発動します。

#### OpenClaw カスタムモデルの認証方式比較

各Providerの認証方式にはそれぞれ特徴があります：
- **API Key方式**：最も一般的。環境変数またはconfig直接指定
- **OAuth方式**：Google Vertex AIなどで使用
- **カスタムヘッダー**：一部のプロキシサービスで必要

## OpenClaw カスタムモデル：主要クラウドモデルの接続

### OpenClaw カスタムモデル × OpenAI（GPT-5シリーズ）

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "openai/gpt-5.1-codex",
        fast: "openai/gpt-4o-mini",
      },
    },
  },
}
```

環境変数：`export OPENAI_API_KEY="sk-xxx"`

### OpenClaw カスタムモデル × Anthropic（Claude Opus 4）

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-20250514",
      },
    },
  },
}
```

### OpenClaw カスタムモデル × Google（Gemini 3）

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "google/gemini-3.0-pro",
      },
    },
  },
}
```

#### OpenClaw カスタムモデル × DeepSeek V3

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "deepseek/deepseek-chat",
      },
    },
  },
}
```

## OpenClaw カスタムモデル：ローカルモデルの接続

### OpenClaw カスタムモデル × Ollama

Ollamaをインストール済みであれば、すぐに接続可能：

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/llama3.3:70b",
      },
      providers: {
        ollama: {
          baseURL: "http://localhost:11434/v1",
        },
      },
    },
  },
}
```

### OpenClaw カスタムモデル × llama.cpp

llama.cppのサーバーモードを起動：

```bash
./llama-server -m model.gguf -c 4096 --port 8080
```

OpenClaw設定：

```json5
{
  agents: {
    defaults: {
      providers: {
        llamacpp: {
          baseURL: "http://localhost:8080/v1",
        },
      },
      model: {
        primary: "llamacpp/local-model",
      },
    },
  },
}
```

#### OpenClaw カスタムモデルのローカル vs クラウド比較

| 項目 | ローカル | クラウド |
|------|---------|--------|
| レイテンシ | 低（ネットワーク不要） | 中〜高 |
| コスト | 電気代のみ | 従量課金 |
| モデル品質 | 70Bまで実用的 | 最先端モデル利用可 |
| プライバシー | 完全ローカル | データ送信あり |

## OpenClaw カスタムモデル：APIゲートウェイで一括管理

### OpenClaw カスタムモデル × Crazyrouter（627+モデル一括接続）

複数のAPI Keyを管理するのが面倒なら、**Crazyrouter**がおすすめです。1つのAPI Keyで627以上のモデルにアクセスでき、料金は公式の約55%オフ：

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "openai/anthropic/claude-sonnet-4-20250514",
        fallback: "openai/gpt-4o",
      },
      providers: {
        openai: {
          apiKey: "sk-your-crazyrouter-key",
          baseURL: "https://crazyrouter.com/v1",
        },
      },
    },
  },
}
```

### OpenClaw カスタムモデルのFailoverと負荷分散

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "openai/claude-sonnet-4-20250514",
        fallback: "openai/gpt-4o",
        fast: "openai/gpt-4o-mini",
      },
    },
  },
}
```

primaryが失敗するとfallbackに自動切替。simpleタスクにはfastモデルを使用してコスト削減。

#### OpenClaw カスタムモデルのコスト最適化戦略

1. **タスク別モデル選択**：簡単な質問はmini、複雑なコーディングはopus
2. **APIゲートウェイ活用**：Crazyrouter経由で約45%コスト削減
3. **ローカル+クラウドハイブリッド**：プライバシー重視タスクはローカル、品質重視はクラウド

## まとめ

OpenClaw カスタムモデル設定の柔軟性は、他のAIアシスタントツールと一線を画しています。クラウド、ローカル、ゲートウェイ——あらゆる接続パターンに対応し、failoverやコスト最適化も設定一つで実現できます。

まだモデル選びに悩んでいるなら、まずは[Crazyrouter](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=openclaw_models)で気軽に試してみてください。627+モデルが1つのキーで使えます。
