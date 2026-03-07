---
title: "OpenClaw チュートリアル：ゼロから始めるプライベート AI アシスタント構築ガイド"
emoji: "🦞"
type: "tech"
topics: ["openclaw", "ai", "selfhosted", "telegram"]
published: true
---

## はじめに

自分専用の AI アシスタントを持ちたいと思ったことはありませんか？Web 版の ChatGPT ではなく、自分のサーバーで動作し、Telegram や Discord からいつでも話しかけられるアシスタントです。

この OpenClaw チュートリアルでは、まさにそれを実現します。OpenClaw はオープンソースの AI アシスタントフレームワークで、VPS やラズベリーパイ、ノートPC にデプロイし、Telegram・WhatsApp・Discord などのプラットフォームから対話できます。627 以上の AI モデルに対応し、設定も柔軟です。

それでは、ステップバイステップで進めていきましょう。

---

## OpenClaw チュートリアル：インストールと環境準備（Node.js 22+、ワンライナーインストール）

### OpenClaw チュートリアル：システム要件と前提条件

まず、以下の環境要件を確認してください。

- **OS**：Linux（Ubuntu 22.04+ 推奨）、macOS、WSL2
- **Node.js**：22.0 以上（必須）
- **メモリ**：最低 512MB、推奨 1GB 以上
- **ネットワーク**：外部接続可能（AI モデル API へのアクセスが必要）

Node.js 22+ がまだインストールされていない場合は、nvm で簡単にセットアップできます。

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source ~/.bashrc
nvm install 22
node -v  # v22.x.x と表示されれば OK
```

#### OpenClaw チュートリアル補足：なぜ Node.js 22+ が必須なのか？

OpenClaw はネイティブ WebSocket サポートや改良された ESM モジュールシステムなど、Node.js 22 の新機能を活用しています。古いバージョンではエラーが発生しますのでご注意ください。

### OpenClaw チュートリアル：ワンライナーインストールスクリプト

環境が整ったら、OpenClaw のインストールはたった 1 行です。

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

このスクリプトは以下を自動的に実行します。

1. システム環境と Node.js バージョンの検出
2. npm によるグローバルインストール
3. デフォルト設定ディレクトリ `~/.openclaw/` の作成
4. 初期設定ファイルの生成

インストール完了後、オンボーディングコマンドで初期設定を行います。

```bash
openclaw onboard
```

この対話式ガイドで、作業ディレクトリの設定やデフォルトモデルの選択など、基本設定を完了できます。

---

## OpenClaw チュートリアル：最初の AI モデルを設定する（API Key 設定、Crazyrouter 経由で 627+ モデルに接続）

### OpenClaw チュートリアル：設定ファイルの構造を理解する

OpenClaw の設定はすべて 1 つのファイルに集約されています。

```
~/.openclaw/openclaw.json
```

拡張子は `.json` ですが、実際には **JSON5** 形式をサポートしています。コメントや末尾カンマ、シングルクォートが使えるので、開発者にとっては非常に便利です。

最小構成の設定ファイルは以下のとおりです。

```json5
{
  // OpenClaw 最小構成
  agents: {
    defaults: {
      // ワークスペースディレクトリ（SOUL.md などのペルソナファイルを配置）
      workspace: "~/.openclaw/workspace",
      
      // AI モデル設定
      model: {
        primary: "openai/gpt-4o",  // 形式：provider/model
      },
    },
  },
}
```

#### OpenClaw チュートリアル Tips：ワークスペースファイルの説明

ワークスペースディレクトリには、AI アシスタントの「魂」を定義する重要なファイルがあります。

| ファイル | 役割 |
|------|------|
| `SOUL.md` | アシスタントのコアとなるペルソナと行動指針 |
| `AGENTS.md` | ワークフローとセッションルール |
| `USER.md` | ユーザー（あなた）に関する情報 |
| `TOOLS.md` | ツール使用メモと API キー |
| `IDENTITY.md` | アシスタントの ID 情報（名前、性格など） |

これらのファイルを自由に編集して、アシスタントをカスタマイズできます。例えば `SOUL.md` に「簡潔に話して、無駄な言葉は省いて」と書けば、アシスタントは端的な応答をするようになります。

### OpenClaw チュートリアル：Crazyrouter で 627+ モデルにワンストップ接続

OpenClaw には `openai`、`anthropic`、`google`、`openrouter`、`deepseek` など複数の AI プロバイダーが組み込まれています。各プロバイダーの API Key を直接設定して利用できます。

しかし、複数の API Key を管理したくない場合は、**Crazyrouter** を統一ゲートウェイとして使うのがおすすめです。

Crazyrouter は AI モデルルーティングサービスで、1 つの API Key で GPT-4o、Claude 4、Gemini 2.5、DeepSeek R1 など 627 以上のモデルにアクセスできます。

```json5
{
  agents: {
    defaults: {
      workspace: "~/.openclaw/workspace",
      model: {
        // Crazyrouter 経由で Claude を使用
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

#### OpenClaw チュートリアル応用編：マルチモデル切り替え戦略

フォールバックモデルを設定しておけば、メインモデルが利用不可の場合に自動で切り替わります。

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "openai/anthropic/claude-sonnet-4-20250514",
        fallback: "openai/gpt-4o",        // メインが落ちたらこちら
        fast: "openai/gpt-4o-mini",        // 簡単なタスクはコスト重視
      },
    },
  },
}
```

タスクの複雑さやモデルの可用性に応じて、最適なモデルが自動選択されます。

---

## OpenClaw チュートリアル：チャットプラットフォームに接続する（Telegram・WhatsApp・Discord 設定）

モデルの設定が完了したら、次は AI アシスタントを「オンライン」にしましょう。OpenClaw は Telegram、WhatsApp、Discord などの主要プラットフォームに対応しています。

### OpenClaw チュートリアル：Telegram Bot 設定（最もおすすめ）

Telegram は OpenClaw で最も推奨されるプラットフォームです。設定が最もシンプルで、機能も充実しています。

**ステップ 1：Telegram Bot を作成する**

1. Telegram で `@BotFather` を検索
2. `/newbot` を送信し、指示に従って名前を設定
3. Bot Token を取得（`123456:ABC-DEF...` のような形式）

**ステップ 2：設定ファイルに記述する**

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

#### OpenClaw チュートリアル セキュリティ注意：allowFrom は必ず設定すること

`allowFrom` はホワイトリスト機能で、リストに含まれるユーザーのみが Bot と対話できます。設定しないと、誰でもあなたの Bot（と API クレジット）を使えてしまいます。

### OpenClaw チュートリアル：WhatsApp と Discord の設定

**WhatsApp** は WhatsApp Business API を使用するため、設定がやや複雑です。

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+81901234xxxx"],  // 許可する電話番号
    },
  },
}
```

**Discord** は Discord Developer Portal で Application と Bot を作成する必要があります。

```json5
{
  channels: {
    discord: {
      token: "your-discord-bot-token",
      allowFrom: ["your-discord-user-id"],
    },
  },
}
```

どのプラットフォームも基本的な流れは同じです。Token を取得し、設定ファイルに記述し、ホワイトリストを設定するだけです。

---

## OpenClaw チュートリアル：最初のメッセージを送信する（Control UI の使い方）

### OpenClaw チュートリアル：Gateway サービスを起動する

すべての設定が完了したら、OpenClaw Gateway を起動します。

```bash
openclaw gateway start
```

Gateway はデフォルトポート **18789** で起動します。以下のコマンドでステータスを確認できます。

```bash
openclaw gateway status
```

`running` と表示されれば正常です。

### OpenClaw チュートリアル：Control UI でアシスタントを管理する

OpenClaw には Web 管理画面（Control UI）が付属しています。

```bash
openclaw dashboard
```

ブラウザで `http://localhost:18789` にアクセスすると、以下の機能を備えた管理パネルが表示されます。

- **チャット画面**：Web 上で直接 AI アシスタントと対話
- **設定管理**：設定ファイルのビジュアル編集
- **ログ閲覧**：対話ログとエラー情報のリアルタイム表示
- **モデル切り替え**：ワンクリックで AI モデルを変更

#### OpenClaw チュートリアル実践：最初のメッセージを送ってみよう

Telegram を開いて、作成した Bot にメッセージを送ってみましょう。

```
こんにちは！あなたは誰ですか？
```

設定が正しければ、数秒以内に AI アシスタントが返信します。おめでとうございます、あなた専用の AI アシスタントが稼働開始です！

返信がない場合は、以下の順序でトラブルシューティングしてください。

1. `openclaw gateway status` でサービスの稼働を確認
2. 設定ファイルの Bot Token が正しいか確認
3. `allowFrom` にあなたのユーザー名が含まれているか確認
4. ターミナルのログ出力を確認

### OpenClaw チュートリアル：よく使う CLI コマンド一覧

```bash
# サービス管理
openclaw gateway start      # サービス起動
openclaw gateway stop       # サービス停止
openclaw gateway restart    # サービス再起動
openclaw gateway status     # ステータス確認

# 初期化と管理
openclaw onboard            # 対話式ガイド設定
openclaw dashboard          # Web 管理画面を開く
```

---

## まとめとおすすめ

ここまでで、OpenClaw チュートリアルの全内容が完了しました。環境構築からモデル設定、プラットフォーム接続、最初のメッセージ送信まで、一通りの流れを解説しました。手順はシンプルです：OpenClaw をインストール → 設定を記述 → サービスを起動。

AI モデルの選択について、複数の API Key を管理する手間を省きたい方には [Crazyrouter](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=openclaw_tutorial) を強くおすすめします。1 つの Key で 627 以上のモデルにアクセスでき、OpenAI・Anthropic・Google・DeepSeek など主要プロバイダーをすべてカバーしています。従量課金で無料枠もあるため、個人開発者や小規模チームに最適です。

> 🔗 Crazyrouter 公式サイト：[https://crazyrouter.com](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=openclaw_tutorial)
> 登録するだけで無料クレジットが付与されます。OpenAI、Claude、Gemini など主要モデルに対応。

---

*本記事は OpenClaw 最新版に基づいて執筆しています。ご質問があれば、[OpenClaw GitHub](https://github.com/nicepkg/openclaw) で Issue を作成するか、コミュニティにご参加ください。*
