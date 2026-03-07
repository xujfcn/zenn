---
title: "OpenClaw + Crazyrouter で5分でAI Telegramボットを構築する方法【2026年版】"
emoji: "🦞"
type: "tech"
topics: ["ai", "telegram", "chatbot", "openclaw", "python"]
published: true
---

## OpenClawとは？

OpenClawは**オープンソースのAIアシスタントフレームワーク**です。Telegram、Discord、WhatsApp、Slackなど**20+プラットフォーム**に対応。

**特徴：**
- 1つのAPIキーで627+モデル利用可能（GPT-5、Claude、Gemini、DeepSeek等）
- 永続メモリ — 会話をセッション間で記憶
- スキルシステム — Web検索、コード実行、ファイル管理
- 5分でデプロイ

## なぜCrazyrouter？

OpenClawにはAIモデルAPIが必要です。**Crazyrouter**を使うと：

| 項目 | 直接OpenAI | Crazyrouter |
|------|----------|-------------|
| モデル数 | OpenAIのみ | **627+** |
| GPT-5.2 価格 | $10/M tokens | **$4.50** (55%オフ) |
| Claude Opus 4.6 | 別アカウント必要 | **同じキーで利用可** |
| クレジット期限 | あり | **なし** |

→ [crazyrouter.com](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=dev_community)

## デプロイ手順（5分）

### 準備

1. **VPS** — 2コア4GB以上（$5/月〜）
2. **Crazyrouter APIキー** — [crazyrouter.com](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=dev_community) で無料登録
3. **Telegram Botトークン** — @BotFather で作成

### ワンラインインストール

```bash
curl -fsSL https://raw.githubusercontent.com/xujfcn/crazyrouter-openclaw/main/install.sh | bash
```

### 対話式セットアップ

```
🔑 Crazyrouter APIキーを入力: sk-xxxxx
🤖 Telegram Botトークンを入力: 123456:ABC...
⚙️ 15+モデルを設定中...
🚀 OpenClawを起動中...
✅ 完了！
```

### Telegramでペアリング

1. Telegramでボットに `/start` を送信
2. ペアリングコードが返信される
3. ターミナルにコードを入力

## おすすめモデル

| 用途 | モデル | Crazyrouter価格 |
|------|--------|----------------|
| 万能アシスタント | Claude Opus 4.6 | $6.75/M input |
| コスパ最強 | GPT-5 Mini | $0.14/M input |
| コーディング | GPT-5.3 Codex | $3.38/M input |
| 高速応答 | Gemini 3 Flash | $0.04/M input |
| 推論・数学 | DeepSeek R1 | $0.055/M input |

## コスト見積もり

| 使用量 | 月間トークン | 月額（Crazyrouter） |
|--------|------------|-------------------|
| ライト（10通/日） | ~300K | **$2** |
| 通常（50通/日） | ~1.5M | **$8** |
| ヘビー（200通/日） | ~6M | **$30** |

VPS $5/月 + API $8-25/月 = **月額$13-30**

ChatGPT Plus（$20/月）より安くて、627+モデル＋マルチプラットフォーム。

## 管理コマンド

```bash
openclaw status                        # ステータス確認
journalctl --user -u openclaw -f       # ログ確認
systemctl --user restart openclaw      # 再起動
openclaw configure --model gpt-5-mini  # モデル変更
```

## まとめ

| 項目 | 値 |
|------|---|
| セットアップ時間 | 5分 |
| モデル数 | 627+ |
| 対応プラットフォーム | 20+ |
| 月額コスト | $13-30 |
| 必要なスキル | コマンドライン基礎のみ |

→ [crazyrouter.com](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=dev_community) | [docs.openclaw.ai](https://docs.openclaw.ai) | [github.com/xujfcn/crazyrouter-openclaw](https://github.com/xujfcn/crazyrouter-openclaw)

*2026年3月7日、Ubuntu 24.04で検証済み。*
