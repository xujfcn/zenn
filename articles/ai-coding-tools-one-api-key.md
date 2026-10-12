---
title: "AIコーディングツール完全ガイド：Cursor / Cline / Aider を1つのAPIキーで使い倒す"
emoji: "💻"
type: "tech"
topics: ["AI", "Cursor", "VSCode", "Python", "LLM"]
published: true
---

> 2026-10-12 更新：モデルごとの料金・対応 API は[最新カタログ](https://crazyrouter.com/en/pricing)で確認してください。一律割引、固定モデル数、未確認の無料クレジットの記述は撤回しました。

## この記事について

2026年、AIコーディングツールは開発者の必須ツールになりました。Cursor、Cline、Aider、Continue...選択肢は豊富ですが、それぞれ別のAPIキーが必要だったり、設定方法がバラバラだったりして、意外と面倒です。

この記事では、**1つのAPIキーですべてのAIコーディングツールを統一的に使う方法**を紹介します。GPT-4o、Claude、DeepSeek、Geminiなど624以上のモデルを、ツールを問わず自由に切り替えられます。

## 前提：なぜAPIゲートウェイが必要なのか

各AIプロバイダーのAPIを直接使う場合の問題点：

| 問題 | 具体例 |
|------|--------|
| キー管理が煩雑 | OpenAI、Anthropic、Google、DeepSeekそれぞれのキーが必要 |
| 料金体系がバラバラ | プロバイダーごとにダッシュボードを確認 |
| ツールごとに設定が必要 | CursorとClineで別々にキーを設定 |
| モデル切り替えが面倒 | プロバイダーを変えるとSDKも変わる |

**APIゲートウェイ**を使えば、これらの問題がすべて解決します。この記事では [Crazyrouter](https://crazyrouter.com) を使います。OpenAI API完全互換で、624以上のモデルに1つのキーでアクセスできます。

## 共通設定

すべてのツールで使う情報：

```
API Base URL: https://crazyrouter.com/v1
API Key:      sk-あなたのキー（crazyrouter.com で取得）
```

### 環境変数で一括設定（推奨）

```bash
# ~/.bashrc or ~/.zshrc に追加
export OPENAI_API_KEY="sk-あなたのcrazyrouter-key"
export OPENAI_API_BASE="https://crazyrouter.com/v1"
export OPENAI_BASE_URL="https://crazyrouter.com/v1"
```

これを設定しておけば、多くのツールが自動的にこの設定を読み込みます。

## 1. Cursor

[Cursor](https://cursor.sh/) は現在最も人気のあるAI搭載コードエディタです。

### 設定手順

1. Cursor を開く → `Settings` → `Models`
2. `OpenAI API Key` に `sk-あなたのキー` を入力
3. `Override OpenAI Base URL` に `https://crazyrouter.com/v1` を入力
4. モデルリストに使いたいモデルを追加
5. 保存

### おすすめモデル設定

| 用途 | モデル | 理由 |
|------|--------|------|
| Tab補完 | `gpt-4o-mini` | 高速・低コスト |
| Chat | `claude-sonnet-4-6` | コード理解力最強 |
| Composer | `deepseek-chat` | コスパ最強 |

### Cursorの裏技

Cursorの月額$20のProプランを使わなくても、自分のAPIキーを設定すれば**無料版でもAI機能が使えます**。Crazyrouter経由なら、Cursorが公式サポートしていないモデル（DeepSeek、Geminiなど）も利用可能です。

## 2. Cline（VS Code拡張機能）

[Cline](https://github.com/cline/cline) はVS Code上で動作する強力なAIコーディングアシスタントです。ファイルの作成・編集、ターミナルコマンドの実行まで自律的に行えます。

### 設定手順

1. VS Codeの拡張機能マーケットプレイスで「Cline」をインストール
2. サイドバーのClineアイコンをクリック → 設定（⚙️）
3. `API Provider` → `OpenAI Compatible` を選択
4. 以下を入力：

```
Base URL: https://crazyrouter.com/v1
API Key:  sk-あなたのキー
Model ID: claude-sonnet-4-6
```

### settings.json での設定

```json
{
  "cline.apiProvider": "openai-compatible",
  "cline.openaiBaseUrl": "https://crazyrouter.com/v1",
  "cline.openaiApiKey": "sk-あなたのキー",
  "cline.openaiModelId": "claude-sonnet-4-6"
}
```

### タスク別モデル使い分け

Clineは1つのモデルしか設定できませんが、タスクに応じて切り替えると効率的です：

```
複雑なリファクタリング → claude-sonnet-4-6（最高品質）
日常的なコーディング   → deepseek-chat（コスパ最強、$0.14/100万トークン）
バグ修正・小さな変更   → gpt-4o-mini（高速）
アルゴリズム問題       → deepseek-reasoner（推論特化）
```

## 3. Continue（VS Code / JetBrains）

[Continue](https://continue.dev/) はオープンソースのAIコーディングアシスタントで、VS CodeとJetBrains IDEの両方に対応しています。

### 設定ファイル

`~/.continue/config.json` を編集：

```json
{
  "models": [
    {
      "title": "Claude Sonnet (Crazyrouter)",
      "provider": "openai",
      "model": "claude-sonnet-4-6",
      "apiBase": "https://crazyrouter.com/v1",
      "apiKey": "sk-あなたのキー"
    },
    {
      "title": "DeepSeek V3 (Crazyrouter)",
      "provider": "openai",
      "model": "deepseek-chat",
      "apiBase": "https://crazyrouter.com/v1",
      "apiKey": "sk-あなたのキー"
    },
    {
      "title": "GPT-4o (Crazyrouter)",
      "provider": "openai",
      "model": "gpt-4o",
      "apiBase": "https://crazyrouter.com/v1",
      "apiKey": "sk-あなたのキー"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Tab補完",
    "provider": "openai",
    "model": "gpt-4o-mini",
    "apiBase": "https://crazyrouter.com/v1",
    "apiKey": "sk-あなたのキー"
  }
}
```

### Continueの利点

- **複数モデルを登録**して、サイドバーのドロップダウンで瞬時に切り替え可能
- Tab補完用に別のモデル（安いモデル）を設定できる
- JetBrains IDE（IntelliJ、PyCharm、WebStorm等）でも同じ設定で動作

## 4. Aider（ターミナル）

[Aider](https://aider.chat/) はGit統合されたターミナルベースのAIコーディングツールです。

### インストールと設定

```bash
pip install aider-chat

# 環境変数を設定（前述の共通設定でOK）
export OPENAI_API_KEY="sk-あなたのキー"
export OPENAI_API_BASE="https://crazyrouter.com/v1"
```

### 使い方

```bash
# DeepSeek V3で開始（コスパ重視）
aider --model deepseek-chat

# Claude Sonnetで開始（品質重視）
aider --model claude-sonnet-4-6

# GPT-4oで開始
aider --model gpt-4o
```

### 設定ファイル

`~/.aider.conf.yml` を作成：

```yaml
openai-api-key: sk-あなたのキー
openai-api-base: https://crazyrouter.com/v1
model: deepseek-chat
```

### Aiderの強み

- **Git統合**：変更を自動でコミットしてくれる
- **マルチファイル編集**：複数ファイルにまたがるリファクタリングが得意
- **ターミナルで完結**：SSHでリモートサーバーに接続して使える

## 5. OpenClaw（AIエージェント）

[OpenClaw](https://github.com/openclaw/openclaw) はオープンソースのAIエージェントフレームワークです。Telegram、Discord、Slackなどのメッセージングプラットフォームに接続して、自律的に動作するAIアシスタントを構築できます。

### 設定

`~/.openclaw/openclaw.json`:

```json5
{
  models: {
    providers: {
      crazyrouter: {
        baseUrl: "https://crazyrouter.com/v1",
        apiKey: "sk-あなたのキー",
        api: "openai-completions",
        models: [
          {
            id: "claude-opus-4-6",
            name: "Claude Opus 4.6",
            reasoning: false,
            input: ["text", "image"],
            contextWindow: 200000,
            maxTokens: 8192,
          },
          {
            id: "deepseek-chat",
            name: "DeepSeek V3",
            reasoning: false,
            input: ["text"],
            contextWindow: 64000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "crazyrouter/claude-opus-4-6" },
    },
  },
}
```

## コスト最適化戦略

すべてのツールを1つのAPIキーで管理できるので、コストの最適化も簡単です。

### モデル選択の指針

```
┌─────────────────────────────────────────────┐
│ タスクの複雑さ                                │
│                                              │
│ 高  │ claude-sonnet-4  ($3/$15)              │
│     │ gpt-4o           ($2.5/$10)            │
│     │                                        │
│ 中  │ deepseek-chat    ($0.14/$0.28) ← 推奨  │
│     │ gpt-4o-mini      ($0.15/$0.60)         │
│     │                                        │
│ 低  │ gemini-2.0-flash ($0.10/$0.40)         │
│     │                                        │
│     └────────────────────────────────────────│
│       安い ──────────────────────────── 高い   │
└─────────────────────────────────────────────┘
```

### 実際のコスト例

1日100回のAIコーディング支援を使った場合（平均500入力 + 200出力トークン/回）：


DeepSeek V3なら**月額約$0.13**（約20円）で済みます。

### おすすめ構成

```
Tab補完:     gpt-4o-mini     （速度重視、安い）
日常コーディング: deepseek-chat   （コスパ最強）
重要なレビュー:  claude-sonnet-4  （品質最高）
```

この構成なら、ヘビーに使っても月額$5以下に収まります。

## Pythonスクリプトでの利用

ツールだけでなく、自作スクリプトからも同じキーで呼び出せます：

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://crazyrouter.com/v1",
    api_key="sk-あなたのキー"
)

def code_review(code: str, model: str = "deepseek-chat") -> str:
    """AIにコードレビューを依頼する"""
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": "あなたは経験豊富なシニアエンジニアです。コードレビューを行い、改善点を指摘してください。"},
            {"role": "user", "content": f"以下のコードをレビューしてください：\n\n```python\n{code}\n```"}
        ]
    )
    return response.choices[0].message.content

# 使用例
my_code = """
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
"""

# DeepSeekでレビュー（安い）
print(code_review(my_code, model="deepseek-chat"))

# Claudeでレビュー（高品質）
print(code_review(my_code, model="claude-sonnet-4-6"))
```

## まとめ

| ツール | 設定箇所 | 難易度 |
|--------|----------|--------|
| Cursor | Settings → Models | ⭐ 簡単 |
| Cline | サイドバー → 設定 | ⭐ 簡単 |
| Continue | ~/.continue/config.json | ⭐⭐ 普通 |
| Aider | 環境変数 or ~/.aider.conf.yml | ⭐ 簡単 |
| OpenClaw | ~/.openclaw/openclaw.json | ⭐⭐ 普通 |
| 自作スクリプト | pip install openai | ⭐ 簡単 |

**ポイント：**
- すべてのツールで同じAPIキーとBase URLを使う
- 環境変数を設定しておけば、ほとんどのツールは自動認識
- モデルはタスクの複雑さに応じて使い分ける
- DeepSeek V3（`deepseek-chat`）がコスパ最強

## 関連リンク

- 🌐 [Crazyrouter](https://crazyrouter.com) — APIキー取得
- 🤖 [オンラインデモ](https://huggingface.co/spaces/xujfcn/Crazyrouter-Demo) — ブラウザで体験
- 💰 [料金比較](https://huggingface.co/spaces/xujfcn/Crazyrouter-Pricing)
- 📖 [Getting Started](https://huggingface.co/xujfcn/Crazyrouter-Getting-Started)
- 🔗 [LangChain統合ガイド](https://huggingface.co/xujfcn/Crazyrouter-LangChain-Guide)
- 💬 [Telegramコミュニティ](https://t.me/crazyrouter)
