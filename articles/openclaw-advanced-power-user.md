---
title: "OpenClaw + Crazyrouter で24時間AIアシスタントを構築：デプロイから上級テクニックまで"
emoji: "🤖"
type: "tech"
topics: ["ai", "chatbot", "telegram", "openai", "selfhosted"]
published: true
---

ChatGPT Plus は月額$20。Claude Pro も$20。Gemini Advanced も$20。

3つのAIサブスクで月$60。しかも各アプリでしか使えない。

**OpenClaw + Crazyrouter** なら、1つのゲートウェイで **300以上のAIモデル** にアクセスでき、Telegram/Discord/Slack から24時間対話できます。

## 30秒でデプロイ

Linuxサーバー（1-2GB RAM）で実行：

```bash
curl -fsSL https://raw.githubusercontent.com/xujfcn/crazyrouter-openclaw/main/install.sh | bash
```

インストーラーが自動で処理：
- Node.js インストール
- 15以上のAIモデルを事前設定（Claude, GPT, Gemini, DeepSeek）
- Telegram Bot セットアップ（対話式）
- 自動起動設定（systemd/launchd）

APIキーは [crazyrouter.com](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=dev_community) で無料取得。

## 上級テク①：スマートモデルルーティング

会話中にモデルを切り替え：

```
/model claude-opus-4-6    → 複雑な推論
/model gpt-5-mini         → 日常会話（安い）
/model deepseek-r1        → コード生成
/model gemini-3-flash     → 高速マルチリンガル
```

**おすすめルーティング戦略：**
- 雑談 → GPT-5 Mini（$0.15/Mトークン）
- コーディング → DeepSeek R1 or Claude Opus
- 翻訳 → Gemini Flash
- 深い分析 → Claude Opus

1つのBotで全シーン対応。4つのアプリを切り替える必要なし。

## 上級テク②：コーディングツールのバックエンドに

OpenClaw は OpenAI API互換。開発ツールのバックエンドとして使えます：

### Cursor

```
Settings → Models → OpenAI API Key
Base URL: http://your-server:18789/v1
API Key: your-crazyrouter-key
```

### Cline (VS Code)

```
API Provider: OpenAI Compatible
Base URL: http://your-server:18789/v1
Model: claude-opus-4-6
```

### Aider

```bash
export OPENAI_API_BASE=http://your-server:18789/v1
export OPENAI_API_KEY=your-key
aider --model claude-opus-4-6
```

## 上級テク③：マルチプラットフォーム同時接続

複数のIMプラットフォームに同時接続：

```yaml
plugins:
  telegram:
    token: "your-tg-token"
  discord:
    token: "your-discord-token"
  slack:
    token: "your-slack-token"
```

## 上級テク④：定時タスク

AIを能動的に動かす：

```bash
openclaw cron add --name "morning-brief" \
  --schedule "0 8 * * *" \
  --task "今日のテックニュースをまとめてTelegramに送信"
```

活用例：
- 毎朝8時にニュースダイジェスト
- 4時間ごとのメール要約
- 週次プロジェクトレポート

## 上級テク⑤：記憶システム

AIが会話を跨いでコンテキストを記憶：

```
あなた：覚えて、私のプロジェクトは Python + FastAPI で AWS にデプロイしてる
AI：了解しました。今後の回答でこの技術スタックを考慮します。

（3日後）
あなた：APIエンドポイントを書いて
AI：FastAPIプロジェクトに基づいて、こちらがコードです...
```

## コスト比較

| ソリューション | 月額 | モデル数 | プラットフォーム |
|--------------|------|---------|---------------|
| ChatGPT Plus | $20 | 1 | Web |
| Claude Pro | $20 | 1 | Web |
| 両方 | $40 | 2 | Web |
| **OpenClaw + Crazyrouter** | **$5-15** | **300+** | **TG/Discord/Slack/API** |

## 始めよう

```bash
curl -fsSL https://raw.githubusercontent.com/xujfcn/crazyrouter-openclaw/main/install.sh | bash
```

- GitHub: [xujfcn/crazyrouter-openclaw](https://github.com/xujfcn/crazyrouter-openclaw)
- APIキー: [crazyrouter.com](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=dev_community)
- コミュニティ: [Telegram グループ](https://t.me/crazyrouter)
