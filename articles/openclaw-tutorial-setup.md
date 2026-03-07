---
title: "OpenClaw チュートリアル：ゼロから始めるプライベートAIアシスタント構築ガイド"
emoji: "🚀"
type: "tech"
topics: ["openclaw", "ai", "telegram", "nodejs", "chatbot"]
published: true
---

## はじめに

自分専用のAIアシスタントを構築してみたいと思ったことはありませんか？ウェブ版のChatGPTではなく、自分のサーバーで動作し、TelegramやDiscordからいつでも呼び出せるプライベートなAIアシスタント。

**OpenClaw**は、そんな夢を実現するオープンソースのAIアシスタントフレームワークです。VPS、Raspberry Pi、ノートPCなど、お好みの環境にデプロイでき、Telegram・WhatsApp・Discordなど主要なチャットプラットフォームを通じて対話できます。627以上のAIモデルに対応し、設定も柔軟です。

この記事では、OpenClawのインストールから初回メッセージ送信まで、ステップバイステップで解説します。

---

## ステップ1：インストールと環境準備（Node.js 22+）

### システム要件

OpenClawを始める前に、以下の要件を確認してください：

- **OS**：Linux（Ubuntu 22.04+推奨）、macOS、WSL2
- **Node.js**：22.0以上（**必須要件**）
- **メモリ**：最低512MB、推奨1GB以上
- **ネットワーク**：外部APIへのアクセスが必要

Node.js 22+がまだインストールされていない場合は、nvmで簡単にセットアップできます：

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source ~/.bashrc
nvm install 22
node -v  # v22.x.x が表示されればOK
```

> **なぜNode.js 22+が必要なのか？**
> OpenClawはNode.js 22のネイティブWebSocketサポートや改良されたESMモジュールシステムなどの新機能を活用しています。低バージョンではエラーになります。

### 一行インストール

環境が整ったら、インストールはワンコマンドです：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

このスクリプトが自動的に以下を実行します：

1. システム環境とNode.jsバージョンの検出
2. npm経由でOpenClawをグローバルインストール
3. デフォルト設定ディレクトリ `~/.openclaw/` の作成
4. 初期設定ファイルの生成

インストール完了後、対話式セットアップを実行します：

```bash
openclaw onboard
```

このガイドに従って、ワークスペースの設定やデフォルトモデルの選択など、基本設定を完了できます。

---

## ステップ2：AIモデルの設定（API Key設定、627+モデル対応）

### 設定ファイルの構造

OpenClawの全設定は1つのファイルに集約されています：

```
~/.openclaw/openclaw.json
```

拡張子は`.json`ですが、実際には**JSON5**形式をサポートしており、コメントや末尾カンマ、シングルクォートが使えます。

最小限の設定ファイルは以下の通りです：

```json5
{
  // OpenClaw 最小設定
  agents: {
    defaults: {
      // ワークスペース（SOUL.mdなどの人格ファイルを格納）
      workspace: "~/.openclaw/workspace",
      
      // AIモデル設定
      model: {
        primary: "openai/gpt-4o",  // 形式：provider/model
      },
    },
  },
}
```

### ワークスペースファイルの役割

ワークスペースディレクトリにはAIアシスタントの「魂」を定義するファイルがあります：

- **`SOUL.md`** — アシスタントのコア人格と行動指針
- **`AGENTS.md`** — ワークフローとセッションルール
- **`USER.md`** — ユーザー（あなた）に関する情報
- **`TOOLS.md`** — ツール使用メモとAPIキー
- **`IDENTITY.md`** — アシスタントの身元情報（名前、性格など）

例えば、`SOUL.md`に「簡潔に話すこと。無駄な前置きは不要」と書けば、アシスタントは端的な応答スタイルになります。

### Crazyrouter経由で627+モデルに一括アクセス

OpenClawは`openai`、`anthropic`、`google`、`openrouter`、`deepseek`など複数のプロバイダーを内蔵しています。それぞれのAPI Keyを直接設定することもできますが、もっと便利な方法があります。

**Crazyrouter**を統一ゲートウェイとして使えば、1つのAPI Keyで627以上のモデルにアクセスできます（GPT-4o、Claude 4、Gemini 2.5、DeepSeek R1など）：

```json5
{
  agents: {
    defaults: {
      workspace: "~/.openclaw/workspace",
      model: {
        primary: "openai/anthropic/claude-sonnet-4-20250514",
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

#### フォールバックモデルの設定

メインモデルが利用不可の場合に自動切り替えする設定も可能です：

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "openai/anthropic/claude-sonnet-4-20250514",
        fallback: "openai/gpt-4o",        // メインがダウンした時用
        fast: "openai/gpt-4o-mini",        // 軽いタスク用（コスト削減）
      },
    },
  },
}
```

---

## ステップ3：チャットプラットフォームへの接続

モデルを設定したら、次はAIアシスタントを「オンライン」にしましょう。OpenClawはTelegram、WhatsApp、Discordなど主要プラットフォームに対応しています。

### Telegram Bot設定（最も推奨）

Telegramは設定が最も簡単で、機能も最も充実しています。

**ステップ1：Telegram Botを作成**

1. Telegramで `@BotFather` を検索
2. `/newbot` を送信し、指示に従って名前を設定
3. Bot Tokenを取得（`123456:ABC-DEF...` のような形式）

**ステップ2：設定ファイルに記述**

```json5
{
  agents: {
    defaults: {
      workspace: "~/.openclaw/workspace",
      model: {
        primary: "openai/anthropic/claude-sonnet-4-20250514",
      },
      providers: {
        openai: {
          apiKey: "sk-your-crazyrouter-key",
          baseURL: "https://crazyrouter.com/v1",
        },
      },
    },
  },
  channels: {
    telegram: {
      token: "your-bot-token",
      allowFrom: ["your-telegram-username"],  // ホワイトリスト
    },
  },
}
```

> **⚠️ セキュリティ注意**
> `allowFrom` は必ず設定してください。設定しないと、誰でもあなたのBotを利用でき、APIクレジットが消費されてしまいます。

### WhatsApp・Discord設定

**WhatsApp**はBusiness APIが必要で、設定がやや複雑です。**Discord**はDeveloper PortalでApplicationとBotを作成します。どちらもコア概念は同じで、プラットフォームのTokenを取得→設定ファイルに記載→ホワイトリストを設定、という流れです。

---

## ステップ4：初回メッセージの送信

### Gatewayの起動

すべての設定が完了したら、OpenClaw Gatewayを起動します：

```bash
openclaw gateway start
```

デフォルトポート**18789**で起動します。ステータス確認は以下のコマンドで：

```bash
openclaw gateway status
```

`running` と表示されれば準備完了です。

### Control UIでの管理

OpenClawにはWeb管理画面（Control UI）が付属しています：

```bash
openclaw dashboard
```

ブラウザで `http://localhost:18789` にアクセスすると、チャットインターフェース、設定管理、ログビューアー、モデル切り替えなどが利用できます。

### 実践：最初のメッセージを送ってみよう

Telegramで先ほど作成したBotを開き、メッセージを送信してみましょう：

```
こんにちは！あなたは誰ですか？
```

設定が正しければ、数秒以内にAIアシスタントが応答します。おめでとうございます、あなたのプライベートAIアシスタントが稼働開始です！

応答がない場合のトラブルシューティング：

1. `openclaw gateway status` でサービスの稼働を確認
2. Bot Tokenが正しいか確認
3. `allowFrom` にあなたのユーザー名が含まれているか確認
4. ターミナルのログ出力を確認

### よく使うCLIコマンド一覧

```bash
# サービス管理
openclaw gateway start      # 起動
openclaw gateway stop       # 停止
openclaw gateway restart    # 再起動
openclaw gateway status     # ステータス確認

# 初期設定・管理
openclaw onboard            # 対話式セットアップ
openclaw dashboard          # Web管理画面を開く
```

---

## まとめ

ここまでで、OpenClawのインストール→モデル設定→プラットフォーム接続→初回メッセージ送信まで、すべてのステップが完了しました。要約すると、たった3ステップです：**OpenClawをインストール → 設定を記述 → サービスを起動**。

AIモデルの選択で迷っている方には、[Crazyrouter](https://crazyrouter.com?utm_source=zenn&utm_medium=tutorial&utm_campaign=openclaw_tutorial)がおすすめです。1つのAPI Keyで627以上のモデルに対応し、OpenAI・Anthropic・Google・DeepSeekなどすべての主要プロバイダーをサポート。従量課金で無料枠もあるため、個人開発者にとってOpenClawとの最適な組み合わせです。

質問があれば、[OpenClaw GitHub](https://github.com/nicepkg/openclaw)でIssueを立てるか、コミュニティに参加してください。

Happy hacking! 🚀
