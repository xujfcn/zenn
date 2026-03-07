---
title: "OpenClaw のアーキテクチャを徹底解説：Gateway から Agent Runtime まで全体像を理解する"
emoji: "🏗️"
type: "tech"
topics: ["openclaw", "ai", "architecture", "oss"]
published: false
---

# OpenClaw のアーキテクチャを徹底解説：Gateway から Agent Runtime まで全体像を理解する

> AI アシスタントのセルフホスティングを調べていて、OpenClaw のアーキテクチャを隅々まで読み込みました。この記事ではその全体像を整理してお伝えします。

## 1. OpenClaw とは？一言で言うと

OpenClaw は**オープンソースの AI アシスタントランタイムプラットフォーム**です。コアコンセプトは「**1つの Gateway で全チャットプラットフォームを接続し、1つの Agent Runtime で全 AI モデルを制御する**」こと。

「AI アシスタントの OS」と考えるとわかりやすいでしょう。チャットボットそのものではなく、WhatsApp・Telegram・Discord・Slack・Signal・iMessage など十数のプラットフォームで同一の AI アシスタントを動かすためのインフラです。

Dify や Coze のような「ドラッグ＆ドロップ型 AI アプリ構築プラットフォーム」とは異なり、OpenClaw はより低レイヤーに位置します。**メッセージのルーティング、コンテキストの組み立て、ツールの実行、記憶の永続化**——これらが OpenClaw の関心事です。

## 2. 全体アーキテクチャ：Gateway + Agent の二重コア

```
┌─────────────────────────────────────────────────────────┐
│                     Control Plane                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐ │
│  │ macOS App│  │   CLI    │  │  Web UI  │  │ WebChat │ │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬────┘ │
│       └──────────────┼──────────────┼─────────────┘      │
│                      │  WebSocket   │                    │
│              ┌───────▼──────────────▼───────┐            │
│              │        GATEWAY               │            │
│              │  ┌─────────────────────────┐ │            │
│              │  │ Channel Adapters        │ │            │
│              │  │ WhatsApp│Telegram│Slack  │ │            │
│              │  │ Discord│Signal│iMessage  │ │            │
│              │  └─────────────────────────┘ │            │
│              │  ┌─────────────────────────┐ │            │
│              │  │ Session Manager         │ │            │
│              │  │ Auth │ Protocol │ Cron  │ │            │
│              │  └─────────────────────────┘ │            │
│              └──────────────┬───────────────┘            │
│                             │                            │
│              ┌──────────────▼───────────────┐            │
│              │      AGENT RUNTIME           │            │
│              │  ┌──────┐ ┌──────┐ ┌──────┐ │            │
│              │  │Memory│ │Tools │ │Canvas│ │            │
│              │  └──────┘ └──────┘ └──────┘ │            │
│              │  ┌──────────────────────────┐│            │
│              │  │    Model Providers       ││            │
│              │  │ GPT│Claude│Gemini│DeepSeek││           │
│              │  └──────────────────────────┘│            │
│              └──────────────────────────────┘            │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │                    NODES                          │   │
│  │  macOS │ iOS │ Android │ Headless                │   │
│  │  Camera│Screen│Location│Canvas                   │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

システム全体は4つのレイヤーで構成されています：

1. **Control Plane（制御面）**：macOS App、CLI、Web UI、WebChat——すべてクライアント
2. **Gateway（ゲートウェイ層）**：中核ハブ。全メッセージチャネルとセッションを管理
3. **Agent Runtime（エージェントランタイム）**：AI の頭脳。コンテキスト組み立て・モデル呼び出し・ツール実行を担当
4. **Nodes（ノード層）**：分散デバイスノード。カメラ・画面・位置情報などの物理的な能力を提供

## 3. Gateway 層の深掘り

### 3.1 シングル Gateway アーキテクチャ

OpenClaw の設計思想は**シングル Gateway**です。1台のホストで1つの Gateway プロセスだけが動き、全メッセージチャネルの接続を独占します。

```bash
# Gateway の起動
openclaw gateway

# デフォルトのリッスン先
# WebSocket: 127.0.0.1:18789
# HTTP (Canvas): 同ポート
```

なぜシングルプロセスなのか？WhatsApp（Baileys ライブラリ経由）はシングルセッション接続を要求し、Telegram Bot も単一インスタンスです。複雑な分散ロックを実装するより、1プロセスで全てを管理する方がシンプルです。

### 3.2 WebSocket プロトコル

Gateway の全通信は WebSocket で行われます。プロトコル設計は非常にクリーンです：

```json
// リクエスト
{"type": "req", "id": "abc123", "method": "agent", "params": {...}}

// レスポンス
{"type": "res", "id": "abc123", "ok": true, "payload": {...}}

// イベント（サーバープッシュ）
{"type": "event", "event": "agent", "payload": {...}, "seq": 42}
```

重要な設計判断：

- **最初のフレームは必ず `connect`**：非 JSON や connect 以外の最初のフレームは即切断
- **冪等性キー**：副作用のある操作（`send`、`agent`）には必ず冪等性キーが必要。サーバー側で短期重複排除キャッシュを維持
- **イベントのリプレイなし**：クライアントは切断後に自分で状態をリフレッシュする必要あり

### 3.3 セキュリティモデル

OpenClaw のセキュリティは2層構造です：

**デバイスペアリング：**
- 各 WS クライアントは `connect` 時にデバイス ID を提示
- 新デバイスはペアリング承認が必要、Gateway がデバイストークンを発行
- ローカル接続（loopback）は自動承認可能
- 署名ペイロード v3 は `platform` + `deviceFamily` にバインド

**トークン認証：**
- `OPENCLAW_GATEWAY_TOKEN` 設定後、全接続で `connect.params.auth.token` にトークンが必要

```bash
# SSH トンネルでリモートアクセス
ssh -N -L 18789:127.0.0.1:18789 user@host
```

## 4. Agent Runtime：AI の頭脳

### 4.1 セッション管理

セッション（Session）は Agent Runtime の中核概念です。各チャットウィンドウ、各ユーザー会話が独立したセッションになります。

セッションの主な特性：
- **独立コンテキスト**：各セッションが独自のメッセージ履歴とコンテキストウィンドウを持つ
- **自動圧縮（Compaction）**：コンテキストがモデルのトークン上限に近づくと、履歴メッセージを自動圧縮
- **メモリフラッシュ（Memory Flush）**：圧縮前にサイレントな Agent ラウンドを実行し、重要な情報を永続化メモリに書き込むよう促す

```json5
{
  agents: {
    defaults: {
      compaction: {
        reserveTokensFloor: 20000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 4000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply NO_REPLY if nothing to store."
        }
      }
    }
  }
}
```

この設計は非常に巧妙です——コンテキストが圧縮される前に、AI に「遺言」を残す機会を与え、重要な情報をファイルに書き込ませます。

### 4.2 コンテキストの組み立て

各 Agent ラウンドで、ランタイムは完全なコンテキストを組み立てます：

1. **System Prompt**：`SOUL.md`、`AGENTS.md`、`USER.md` などから読み込み
2. **メモリ注入**：`MEMORY.md`（メインセッションのみ）と `memory/YYYY-MM-DD.md` を読み込み
3. **ツール定義**：ポリシーに基づいて利用可能なツールリストをフィルタリング
4. **メッセージ履歴**：現在のセッションの会話記録
5. **ワークスペースファイル**：ワークスペースの重要ファイルの内容を注入

### 4.3 ツール実行ループ

OpenClaw のツール実行は典型的な ReAct ループです：

```
ユーザーメッセージ → モデル思考 → ツール呼び出し → 結果取得 → モデル再思考 → ... → 最終回答
```

組み込みツールは非常に豊富です：
- `exec`：シェルコマンド実行
- `read`/`write`/`edit`：ファイル操作
- `web_search`/`web_fetch`：Web 検索・スクレイピング
- `browser`：ブラウザ自動化
- `memory_search`/`memory_get`：メモリ検索
- `message`：クロスプラットフォームメッセージ送信
- `sessions_spawn`：サブエージェント編成

## 5. メモリシステム：Markdown がメモリ

OpenClaw のメモリシステムは最も感心した設計の一つです。複雑なベクトルデータベースではなく、**純粋な Markdown ファイル**をメモリの媒体として使います。

### 5.1 二層メモリアーキテクチャ

```
workspace/
├── MEMORY.md              # 長期記憶（手動キュレーション）
└── memory/
    ├── 2026-03-05.md      # 昨日のログ
    ├── 2026-03-06.md      # 今日のログ
    └── heartbeat-state.json
```

- **`MEMORY.md`**：長期記憶。人間の「長期記憶」に相当。意思決定、好み、重要な事実を保存。セキュリティ上、メインセッションでのみ読み込み。
- **`memory/YYYY-MM-DD.md`**：日次ログ。人間の「作業記憶」に相当。追記方式で、各セッションで今日と昨日分を読み込み。

### 5.2 ベクトル検索

メモリは Markdown ファイルですが、OpenClaw はその上にベクトルインデックスを構築しています：

- ファイル変更を自動監視（デバウンス付き）
- 複数の Embedding プロバイダーをサポート：OpenAI、Gemini、Voyage、Mistral、ローカルモデル
- `memory_search` ツールでセマンティック検索

### 5.3 なぜ Markdown なのか？

この設計選択の背景にある哲学は「**ファイルが唯一の真実の源**」です：

- 任意のエディタで閲覧・編集可能
- Git フレンドリー、バージョン管理可能
- データベース不要
- モデルが「覚えている」ことは、ディスクに書かれたもの——透明で監査可能

## 6. プラグインシステム：4つの拡張方向

OpenClaw のプラグインシステムは4つのコアスロットで設計されています：

| スロット | 役割 | 例 |
|---------|------|-----|
| **Channel** | メッセージチャネル適応 | WhatsApp、Telegram、Discord、Slack、Signal、iMessage、IRC、Matrix、LINE... |
| **Memory** | メモリの保存と検索 | memory-core（デフォルト）、カスタムベクトルバックエンド |
| **Tool** | ツール能力の拡張 | ブラウザ制御、ファイル操作、API 呼び出し、Feishu 連携 |
| **Provider** | モデルプロバイダー | OpenAI、Anthropic、Google、DeepSeek... |

Channel プラグインの充実度は印象的です——主流の WhatsApp/Telegram/Discord から、ニッチな Nostr/IRC まで対応しています。

## 7. マルチデバイス連携：Canvas、A2UI、Nodes

### 7.1 Canvas：Agent が編集可能な Web インターフェース

Gateway の HTTP サーバーは2つの特別なパスを提供します：

- `/__openclaw__/canvas/`：Agent が動的に生成・編集できる HTML/CSS/JS ページ
- `/__openclaw__/a2ui/`：A2UI（Agent-to-UI）ホスト

Canvas により、AI アシスタントはテキスト返信に限定されません——インタラクティブな Web UI、データ可視化、ミニアプリを生成できます。

### 7.2 Nodes：分散デバイス能力

Nodes は OpenClaw の最も想像力豊かな設計の一つです。任意のデバイス（macOS、iOS、Android、Headless）が Node として Gateway に接続できます：

```json
{
  "role": "node",
  "caps": ["camera", "screen", "location", "canvas"],
  "commands": ["camera.snap", "screen.record", "location.get"]
}
```

つまり AI アシスタントは：
- スマホのカメラで写真を撮影
- PC の画面を録画
- デバイスの位置情報を取得
- デバイス上に Canvas UI を表示

といったことが可能になります。

## 8. メッセージの完全なライフサイクル

ユーザーがメッセージを送信してから AI が返信するまでの完全な経路を追跡してみましょう：

```
1. ユーザーが Telegram で「今日の天気を調べて」と送信
   │
2. Telegram Bot API → Gateway Channel Adapter (grammY)
   │  メッセージ解析、テキスト抽出、セッション識別
   │
3. Gateway Session Manager
   │  セッション検索/作成、権限チェック
   │
4. Agent Runtime が新ラウンドを開始
   │
   ├─ 4a. コンテキスト組み立て
   │      System Prompt + SOUL.md + MEMORY.md
   │      + memory/2026-03-06.md + ツール定義 + 履歴
   │
   ├─ 4b. AI モデル呼び出し
   │      POST /v1/chat/completions
   │      → モデルが weather ツールの呼び出しを返す
   │
   ├─ 4c. ツール実行
   │      weather("東京") → 天気データ取得
   │
   ├─ 4d. モデル再呼び出し
   │      ツール結果をコンテキストに注入
   │      → モデルが最終回答を生成
   │
5. Gateway が返信を Telegram にルーティング
   │
6. ユーザーが Telegram で天気情報を受信
```

## 9. 競合との比較

| 特性 | OpenClaw | Dify | Coze | AutoGPT |
|------|----------|------|------|---------|
| **位置づけ** | AI アシスタントランタイム | AI アプリ構築プラットフォーム | AI Bot プラットフォーム | 自律型 AI Agent |
| **デプロイ** | セルフホスト（シングルプロセス） | セルフホスト/クラウド | クラウド | セルフホスト |
| **チャットプラットフォーム** | 15+ ネイティブ対応 | 追加統合が必要 | 限定的 | なし |
| **メモリ** | Markdown + ベクトル | ベクトル DB | クラウド | ファイルシステム |
| **マルチデバイス** | Canvas + Nodes | なし | なし | なし |
| **OSS** | ✅ | ✅ | ❌ | ✅ |

## 10. 実践：AI モデルプロバイダーの設定

OpenClaw を動かすには AI モデルの API 設定が必要です。ここでは [Crazyrouter](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=openclaw_arch) を統一 API ゲートウェイとして使うことをお勧めします。1つのキーで 627+ モデル（GPT-5、Claude 4.6、Gemini 3、DeepSeek V3 など）を呼び出せて、価格は公式の約 55% です。

```json5
{
  providers: {
    openai: {
      baseUrl: "https://crazyrouter.com/v1",
      apiKey: "sk-your-crazyrouter-key"
    }
  }
}
```

> 💡 Crazyrouter は OpenAI/Anthropic/Gemini の3つの API フォーマットに対応しており、OpenClaw のマルチモデルアーキテクチャと自然に噛み合います。新規登録で $0.2 のクレジット付与、有効期限なし。

## 11. まとめ：OpenClaw のアーキテクチャから学べること

1. **シングルプロセス > マイクロサービス**：個人 AI アシスタントには十分。Kubernetes もメッセージキューも不要。
2. **ファイル > データベース**：Markdown メモリ、JSON 設定、ファイルシステムワークスペース。シンプルで透明で移植可能。
3. **プロトコルファースト**：WebSocket プロトコルを先に定義し、マルチプラットフォーム対応を自然に実現。
4. **適度なプラグイン化**：4つのコアスロットで 90% の拡張ニーズをカバー。
5. **セキュリティ内蔵**：デバイスペアリング、トークン認証、署名検証——セキュリティは後付けパッチではなくアーキテクチャの一部。

---

**関連リンク：**
- OpenClaw 公式ドキュメント：https://docs.openclaw.ai
- OpenClaw GitHub：https://github.com/openclaw/openclaw
- OpenClaw Discord：https://discord.com/invite/clawd
- Crazyrouter（AI API ゲートウェイ）：[https://crazyrouter.com](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=openclaw_arch)
