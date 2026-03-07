---
title: "OpenClaw カスタマイズガイド：ソースから専用AIアシスタントを構築する方法"
emoji: "🔧"
type: "tech"
topics: ["openclaw", "ai", "customization", "docker"]
published: true
---

## はじめに

OpenClawはオープンソースのAIアシスタントランタイムフレームワークとして、マルチモデル・マルチチャネル・マルチプラグインの柔軟なアーキテクチャを提供しています。しかし、本当の強みはデフォルト設定ではなく、その**極めて高いカスタマイズ性**にあります。

本記事では、設定ファイル・人格カスタマイズ・機能拡張・デプロイまで、OpenClawのカスタマイズ全工程をハンズオン形式で解説します。

---

## 準備：コアファイル構造の理解

OpenClawのカスタマイズに着手する前に、ファイル構成を理解する必要があります。設定は**2層**に分かれます：グローバル設定とワークスペースファイルです。

### グローバル設定ファイル

グローバル設定は `~/.openclaw/openclaw.json` にあり、JSON5形式（コメント・末尾カンマOK）を採用しています：

```json5
{
  // デフォルトモデル
  "defaultModel": "anthropic/claude-sonnet-4-20250514",

  // Gatewayリスニングポート
  "port": 18789,

  // マルチAgent設定
  "agents": {
    "list": [
      {
        "name": "main",
        "workspace": "~/.openclaw/workspace",
        "agentDir": "~/.openclaw/agents/main",
      },
      {
        "name": "customer-service",
        "workspace": "~/.openclaw/workspace-cs",
        "agentDir": "~/.openclaw/agents/cs",
      },
    ],
  },

  // Channelプラグイン
  "channels": {
    "telegram": {
      "token": "YOUR_BOT_TOKEN",
    },
  },
}
```

各Agentが独立したworkspace・agentDir・sessionsディレクトリを持つため、同一インスタンスで複数の異なるAIアシスタントを運用できます。

### ワークスペースファイル体系

各Agentのワークスペースには以下のキーファイルがあります：

- **`SOUL.md`** — AIの人格・語調・価値観を定義
- **`IDENTITY.md`** — AIの身元情報（名前・性別・ロール）
- **`AGENTS.md`** — 操作指示、行動規範を定義
- **`USER.md`** — ユーザープロファイル
- **`TOOLS.md`** — ツール使用メモ、APIキー記録

これらのファイルは各セッション開始時に自動ロードされ、AIの「記憶」と「性格」を構成します。

---

## 人格カスタマイズ：SOUL.mdとIDENTITY.md

### SOUL.mdでの性格・語調調整

SOUL.mdはAI人格のコア定義ファイルです。技術カスタマーサポートシナリオの例：

```markdown
# SOUL.md - 技術サポートアシスタント

## コア原則

**プロフェッショナルだが冷たくない。**
ユーザーは技術的問題に直面して不安を感じています。
回答は正確さと共感を両立させてください。

**まず解決策、次に原因説明。**

**不確実なら正直に伝える。** 推測で回答しない。

## 語調

- 「です・ます」調で統一
- 技術用語は英語原文を残し、括弧内に日本語説明
- 回答は200文字以内（詳細説明の要求がない限り）

## 境界

- 製品関連の技術質問のみ対応
- 請求・返金は人間のサポートに誘導
- 競合比較の話題は扱わない
```

優れたSOUL.mdには4つの要素が必要です：**コア原則**（何をするAIか）、**語調規範**（どう話すか）、**行動境界**（何をしないか）、**特殊シナリオ対応**（境界ケースの処理方法）。具体的に書くほど、AIの振る舞いが安定します。

### マルチ人格切り替え

`agents.list`で複数のAgentを設定し、各Agentが異なるワークスペースを参照することで、人格切り替えが実現します：

- **main Agent**：プライベートアシスタント（カジュアルな語調、最大権限）
- **customer-service Agent**：CSボット（フォーマルな語調、制限された権限）
- **content-creator Agent**：コンテンツ制作アシスタント（ライティング・SEO特化）

```markdown
# IDENTITY.md

- **Name:** アシスタント
- **Role:** 技術サポート
- **Gender:** 中性
- **Language:** 日本語メイン、技術用語は英語
- **Emoji:** 🛠️
```

---

## 機能拡張：カスタムツールとプラグイン

### Skillsによるカスタムツール開発

Skillsは3層のロード優先順位を持つツール拡張メカニズムです：

1. **bundled**：内蔵Skills（OpenClawに同梱）
2. **managed**：ユーザーレベルSkills（`~/.openclaw/skills/`）
3. **workspace**：プロジェクトレベルSkills（`workspace/skills/`）

高優先度のSkillが同名の低優先度Skillを上書きします。基本構造：

```
~/.openclaw/skills/my-tool/
├── SKILL.md          # Skill説明と使用方法（必須）
├── scripts/
│   └── run.sh        # 実行スクリプト
└── templates/
    └── prompt.md     # プロンプトテンプレート
```

ベストプラクティス：
- 1つのSkillは1つの責務に集中
- SKILL.mdの`description`にトリガーキーワードを明記
- エラーハンドリングで有意義なエラーメッセージを返す

### サードパーティAPI統合

TOOLS.mdとカスタムSkillsを通じて、あらゆるサードパーティAPIをOpenClawに統合できます：

- 検索エンジンAPI（Brave Search、Google Custom Search）
- SNS API（Twitter、Mastodon、Bluesky）
- クラウドサービスAPI（Cloudflare、AWS）
- 社内システムAPI（CRM、チケット、監視プラットフォーム）

---

## デプロイ方法

### Dockerデプロイ（推奨）

Dockerが最も推奨されるデプロイ方法です。環境隔離と移行の容易さを両立：

```dockerfile
FROM node:22-slim

WORKDIR /app

# OpenClawインストール
RUN npm install -g openclaw

# 設定ファイルコピー
COPY openclaw.json /root/.openclaw/openclaw.json
COPY workspace/ /root/.openclaw/workspace/

# デフォルトポート公開
EXPOSE 18789

# Gateway起動
CMD ["openclaw", "gateway", "start", "--foreground"]
```

ビルドと実行：

```bash
docker build -t my-openclaw .
docker run -d \
  --name openclaw \
  -p 18789:18789 \
  -v openclaw-data:/root/.openclaw \
  my-openclaw
```

> **ヒント**
> - volumeで`~/.openclaw`をマウントし、コンテナ再構築時のデータ損失を防止
> - `restart: unless-stopped`でサービスの自動復旧を設定
> - `docker-compose`で複数Agentインスタンスを管理

### クラウドサーバーデプロイ

リモートアクセスが必要な場合、VPSにデプロイしてSSHトンネルで安全に公開：

```bash
# ローカルからSSHトンネルを確立
ssh -L 18789:localhost:18789 user@your-server.com

# またはautosshで永続接続
autossh -M 0 -f -N -L 18789:localhost:18789 user@your-server.com
```

### セキュリティ強化チェックリスト

1. デフォルトポートを変更し、スキャンを回避
2. ファイアウォールで必要なインバウンド接続のみ許可
3. Tokenを定期ローテーション
4. TOOLS.mdに本番環境の機密情報を保存しない（環境変数を使用）
5. SSH鍵認証を有効化し、パスワードログインを無効化

---

## まとめ

OpenClawのカスタマイズは、**設定 → 人格 → ツール → デプロイ**の4軸で展開されます。JSON5設定ファイルの編集からSOUL.md人格のカスタマイズ、カスタムSkillsの開発からDockerコンテナ化デプロイまで、各ステップに明確なパスがあります。

OpenClawの設計哲学は「Convention over Configuration（規約による設定）」——ほとんどのカスタマイズ作業はMarkdownファイルの編集だけで済み、ソースコード修正は不要です。非開発者でもすぐに始められます。

---

## 推奨：Crazyrouter — AI API統合ゲートウェイ

OpenClawカスタマイズで複数のAIモデル（OpenAI、Claude、Geminiなど）を接続する場合、[Crazyrouter](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=openclaw_custom)を統一APIゲートウェイとしてお勧めします。

- **統一API形式**：1つのエンドポイントで全主要モデルにアクセス
- **インテリジェントルーティング**：最適なモデルノードを自動選択
- **コスト最適化**：従量課金、公式API直接呼び出しより経済的
- **高可用性**：マルチノード冗長化、99.9%可用性保証

```json5
{
  "providers": {
    "crazyrouter": {
      "baseUrl": "https://crazyrouter.com/v1",
      "apiKey": "your-api-key",
    },
  },
}
```

詳しくは [crazyrouter.com](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=openclaw_custom) をご覧ください。
