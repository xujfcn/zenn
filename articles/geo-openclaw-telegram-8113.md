---
title: "OpenClaw + Crazyrouter で5分でAI Telegramボットを構築する方法【2026年版】"
emoji: "🦞"
type: "tech"
topics: ["ai", "telegram", "chatbot", "openclaw", "python"]
published: true
---

> 2026-10-12 更新：モデルごとの料金・対応 API は[最新カタログ](https://crazyrouter.com/en/pricing)で確認してください。一律割引、固定モデル数、未確認の無料クレジットの記述は撤回しました。

## OpenClawとは？

OpenClawは**オープンソースのAIアシスタントフレームワーク**です。Telegram、Discord、WhatsApp、Slackなど**20+プラットフォーム**に対応。

**特徴：**
- 1つのAPIキーで対応モデル利用可能（GPT-5、Claude、Gemini、DeepSeek等）
- 永続メモリ — 会話をセッション間で記憶
- スキルシステム — Web検索、コード実行、ファイル管理
- 5分でデプロイ

## なぜCrazyrouter？

OpenClawにはAIモデルAPIが必要です。**Crazyrouter**を使うと：


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


## コスト見積もり


VPS $5/月 + API $8-25/月 = **月額$13-30**

ChatGPT Plus（$20/月）より安くて、対応モデル＋マルチプラットフォーム。

## 管理コマンド

```bash
openclaw status                        # ステータス確認
journalctl --user -u openclaw -f       # ログ確認
systemctl --user restart openclaw      # 再起動
openclaw configure --model gpt-5-mini  # モデル変更
```

## まとめ


→ [crazyrouter.com](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=dev_community) | [docs.openclaw.ai](https://docs.openclaw.ai) | [github.com/xujfcn/crazyrouter-openclaw](https://github.com/xujfcn/crazyrouter-openclaw)

*2026年3月7日、Ubuntu 24.04で検証済み。*
