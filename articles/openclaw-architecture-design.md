---
title: "OpenClaw アーキテクチャ解析：AI アシスタントランタイムの 4 層設計思想"
emoji: "🦞"
type: "tech"
topics: ["openclaw", "ai", "selfhosted", "architecture"]
published: true
---

## はじめに

AI アシスタントと聞くと、多くの人はチャットウィンドウの裏にある大規模言語モデルを思い浮かべるでしょう。しかし、AI アシスタントを本当に「生きた存在」にするのは、その背後にあるランタイムアーキテクチャです。OpenClaw アーキテクチャは独自の 4 層設計を採用し、コントロールプレーン・ゲートウェイ・エージェントランタイム・ノードを分離することで、軽量かつ強力な AI アシスタントランタイムを構築しています。

本記事では、OpenClaw アーキテクチャの各層を詳しく解説し、なぜこの設計で 1 つの AI アシスタントが Telegram・Discord・WhatsApp に同時接続しつつ、Mac・iPhone・Android デバイスまで操作できるのかを紐解きます。

```
┌─────────────────────────────────────────────────┐
│                 Control Plane                    │
│          (設定 · 認証 · デバイスペアリング管理)      │
└──────────────────────┬──────────────────────────┘
                       │ HTTPS / Token
┌──────────────────────▼──────────────────────────┐
│                   Gateway                        │
│        (シングルプロセス · WebSocket · メッセージルーティング) │
│   ┌─────────┐  ┌──────────┐  ┌──────────────┐   │
│   │ Channel  │  │ Provider │  │    Memory     │   │
│   │ Plugins  │  │ Plugins  │  │   Plugins     │   │
│   └─────────┘  └──────────┘  └──────────────┘   │
└──────────────────────┬──────────────────────────┘
                       │ Internal API
┌──────────────────────▼──────────────────────────┐
│               Agent Runtime                      │
│    (セッション管理 · コンテキスト組立 · ReAct · Memory Flush) │
│   ┌──────────┐  ┌──────────┐  ┌─────────────┐   │
│   │ Context  │  │  ReAct   │  │   Memory     │   │
│   │ Assembly │  │   Loop   │  │   Flush      │   │
│   └──────────┘  └──────────┘  └─────────────┘   │
└──────────────────────┬──────────────────────────┘
                       │ WebSocket
┌──────────────────────▼──────────────────────────┐
│                    Nodes                         │
│     (macOS · iOS · Android · 物理デバイス機能)      │
│   ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐   │
│   │ Camera │ │ Screen │ │Location│ │ Canvas │   │
│   └────────┘ └────────┘ └────────┘ └────────┘   │
└─────────────────────────────────────────────────┘
```

---

## OpenClaw アーキテクチャ概観：なぜシングルプロセス設計なのか（Gateway ゲートウェイ層・WebSocket プロトコル）

OpenClaw アーキテクチャの最初の重要な設計判断は、Gateway をシングルプロセスで構成することです。これは手抜きではなく、熟慮の結果です。

### OpenClaw アーキテクチャにおける Gateway のシングルプロセス哲学

従来のマイクロサービスアーキテクチャでは、メッセージルーティング・認証・プラグイン管理を複数のサービスに分割します。しかし、個人向け AI アシスタントのユースケースでは、この分割がもたらす複雑さはメリットを大きく上回ります。OpenClaw アーキテクチャの Gateway は、すべてのコア機能を 1 つの Node.js プロセスに集約しています。

- メッセージの受信とルーティング（各 Channel プラグインから）
- WebSocket 接続管理（Nodes との双方向通信）
- セッション状態の維持
- プラグインのライフサイクル管理

シングルプロセスは、内部呼び出しのネットワークオーバーヘッドがゼロ、シンプルなデプロイモデル（1 プロセスですべて完結）、そして天然の状態一貫性を意味します。

#### OpenClaw アーキテクチャの WebSocket プロトコル設計

Gateway と Nodes 間の通信は WebSocket プロトコルを採用しており、メッセージ形式は 3 種類に分類されます。

```typescript
// req: リクエスト-レスポンスパターン
{ type: "req", id: "abc123", method: "camera_snap", params: { facing: "front" } }

// res: レスポンス
{ type: "res", id: "abc123", result: { image: "base64..." } }

// event: 一方向イベントプッシュ
{ type: "event", name: "location_update", data: { lat: 35.68, lng: 139.76 } }
```

この 3 種類のメッセージタイプで、すべてのインタラクションシナリオをカバーしています。同期呼び出しには `req/res`、非同期通知には `event` を使用します。

### OpenClaw アーキテクチャのセキュリティモデル：デバイスペアリングと Token 認証

セキュリティは OpenClaw アーキテクチャの最低限の要件です。各 Node は初回接続時にデバイスペアリングフローを経る必要があります。

1. Node がペアリングリクエストを送信し、Gateway がワンタイムペアリングコードを生成
2. ユーザーが信頼済みデバイスでペアリングを承認
3. ペアリング成功後、Gateway が長期 Token を発行
4. 以降の接続は Token 認証を使用し、再ペアリングは不要

```bash
# ペアリング待ちデバイスの確認
openclaw gateway status

# ペアリング後の Token はローカルに保存
cat ~/.openclaw/nodes/<device-id>/token
```

この方式は Bluetooth ペアリングの考え方を参考にしています。初回の信頼関係が確立されれば、以降の接続は自動認証されます。

---

## OpenClaw アーキテクチャの中核：Agent Runtime（コンテキスト組立・ReAct ループ・Memory Flush）

Agent Runtime は OpenClaw アーキテクチャの頭脳です。ユーザーメッセージを完全な AI インタラクションに変換する役割を担います。

### OpenClaw アーキテクチャのコンテキスト組立メカニズム

AI が応答する前に、Agent Runtime は 5 つのソースからコンテキストを組み立てます。

1. **システムプロンプト（System Prompt）**：AI のアイデンティティ、能力、行動境界を定義
2. **ワークスペースファイル（Workspace Files）**：SOUL.md、USER.md、AGENTS.md などの永続化設定
3. **メモリファイル（Memory）**：MEMORY.md（長期記憶）+ 日次メモリファイル
4. **セッション履歴（Session History）**：現在の会話のコンテキストウィンドウ
5. **ツール結果（Tool Results）**：先行するツール呼び出しの戻り値

```
コンテキスト組立フロー：

System Prompt ──┐
Workspace Files ─┤
Memory Files ────┼──→ [Context Assembly] ──→ LLM API Call
Session History ─┤
Tool Results ────┘
```

このマルチソース組立により、AI は毎回の応答時に完全な「世界観」を持つことができます。自分が誰で、ユーザーが誰で、これまで何が起きたか、手元にどんなツールがあるかを把握しています。

#### OpenClaw アーキテクチャにおけるコンテキストウィンドウの最適化戦略

コンテキストウィンドウには制限があるため、OpenClaw アーキテクチャは以下の戦略でトークン使用量を最適化しています。

- 古いメッセージの自動圧縮またはトランケーション
- ツール出力が長すぎる場合の自動要約
- メモリファイルの関連性によるソート（最も関連性の高い内容を優先注入）

### OpenClaw アーキテクチャの ReAct ループと Memory Flush

#### OpenClaw アーキテクチャにおける ReAct ループ

ReAct（Reasoning + Acting）は Agent Runtime のコア実行モデルです。AI がツールを使用する必要がある場合、ループに入ります。

```
ユーザーメッセージ → LLM 思考 → ツール呼び出しを決定 → ツール実行 → 結果取得 → LLM 再思考 → ...
```

```typescript
// ReAct ループの擬似コード
while (true) {
  const response = await llm.chat(context);
  
  if (response.hasToolCalls) {
    const results = await executeTools(response.toolCalls);
    context.append(results);
    continue; // ループ継続
  }
  
  // ツール呼び出しなし、最終レスポンスを返す
  return response.text;
}
```

このループにより、AI は多段階の推論が可能になります。まず Web を検索し、次にファイルを読み、コマンドを実行し、最後にすべての情報を統合して回答を生成します。

#### OpenClaw アーキテクチャにおける Memory Flush メカニズム

Memory Flush は OpenClaw アーキテクチャで最も巧妙な設計の 1 つです。会話が終了（サイレント期間を検出）した後、Agent Runtime は以下を実行します。

1. 今回の会話の重要情報を振り返る
2. 記憶すべき内容を圧縮・抽出する
3. `memory/YYYY-MM-DD.md`（日次メモリ）に書き込む
4. 必要に応じて `MEMORY.md`（長期記憶）を更新する

これは人間が就寝前に一日の出来事を振り返り、重要なことを短期記憶から長期記憶に移すのと同じです。AI アシスタントはこれにより、セッションをまたいだ連続性を獲得します。もはや「金魚の記憶」のチャットボットではありません。

---

## OpenClaw アーキテクチャのメモリシステム：Markdown がデータベース（二層メモリ・ベクトル検索）

OpenClaw アーキテクチャは大胆な選択をしました。Markdown ファイルをメモリの保存形式として使用しています。SQLite も Redis もなく、純粋なテキストファイルです。

### OpenClaw アーキテクチャの二層メモリ設計

| 層 | ファイル | 用途 | 例え |
|------|------|------|------|
| 長期記憶 | `MEMORY.md` | ユーザーの好み、重要な決定、永続的な知識 | 人間の長期記憶 |
| 日次メモリ | `memory/YYYY-MM-DD.md` | 当日の会話要約、イベント記録 | 人間の日記 |

```markdown
<!-- MEMORY.md の例 -->
# Long-Term Memory

## User Preferences
- 日本語でのコミュニケーションを好む、技術用語は英語
- タイムゾーン：UTC+9（東京）
- よく使うツール：VS Code, iTerm2

## Key Decisions
- 2026-03-01: ブログを Astro フレームワークに移行することを決定
- 2026-02-28: API 料金体系をトークン課金に確定
```

#### OpenClaw アーキテクチャで Markdown をデータベースとして使う利点

なぜ従来のデータベースではなく Markdown なのでしょうか？

- **可読性**：人間が直接メモリファイルを読み書きできる
- **バージョン管理**：Git がネイティブにサポートし、メモリの変更を追跡可能
- **ポータビリティ**：純粋なテキストファイルで依存関係ゼロ
- **AI フレンドリー**：LLM は Markdown 形式の処理が得意

### OpenClaw アーキテクチャのベクトル検索機能

メモリファイルが一定量に達すると、単純なテキストマッチングでは不十分になります。OpenClaw アーキテクチャはベクトル検索機能を統合しており、複数の embedding プロバイダーをサポートしています。

```yaml
# ベクトル検索設定の例
memory:
  embedding:
    provider: openai        # 対応: openai / gemini / voyage / mistral / local
    model: text-embedding-3-small
    dimensions: 1536
```

ベクトル検索のワークフロー：

1. メモリファイルをチャンク分割し、embedding ベクトルを生成
2. ユーザーメッセージも同様に embedding を生成
3. コサイン類似度で最も関連性の高いメモリ断片を検索
4. 関連メモリをコンテキストに注入

これにより、数百日分のメモリファイルがあっても、AI は現在の会話に最も関連する履歴情報をミリ秒単位で見つけることができます。

---

## OpenClaw アーキテクチャのプラグインシステムと拡張性（4 大プラグインスロット）

OpenClaw アーキテクチャの拡張性は、4 大プラグインスロットの上に構築されています。各スロットは特定のカテゴリの能力のインターフェース仕様を定義しており、開発者は必要に応じて実装できます。

### OpenClaw アーキテクチャの 4 大プラグインスロット詳解

```
┌──────────────────────────────────────────┐
│            OpenClaw Plugin System         │
├──────────┬──────────┬────────┬───────────┤
│ Channel  │ Provider │ Memory │   Tool    │
│ プラグイン│ プラグイン│プラグイン│ プラグイン │
├──────────┼──────────┼────────┼───────────┤
│ Telegram │ OpenAI   │ Local  │ Browser   │
│ Discord  │ Anthropic│ Vector │ Exec      │
│ WhatsApp │ Gemini   │ Custom │ Web Search│
│ Slack    │ Mistral  │        │ Camera    │
│ WeChat   │ Local LLM│        │ Custom    │
└──────────┴──────────┴────────┴───────────┘
```

- **Channel プラグイン**：メッセージの入口と出口を定義。各 Channel プラグインがメッセージの受信・送信・フォーマット変換を実装
- **Provider プラグイン**：異なる LLM プロバイダーとの接続。OpenAI、Anthropic、Google Gemini、Mistral、ローカルモデルまで対応
- **Memory プラグイン**：メモリの保存と検索方法を制御。デフォルトはローカル Markdown ファイルだが、カスタム実装で外部ストレージに接続可能
- **Tool プラグイン**：AI の能力境界を拡張。ブラウザ制御、Shell コマンド実行、Web 検索、ファイル操作など

#### OpenClaw アーキテクチャにおけるプラグインの登録メカニズム

プラグインは宣言的な設定で Gateway に登録します。

```yaml
# openclaw.yaml
plugins:
  channels:
    - telegram:
        token: ${TELEGRAM_BOT_TOKEN}
    - discord:
        token: ${DISCORD_BOT_TOKEN}
  providers:
    - openai:
        apiKey: ${OPENAI_API_KEY}
        model: gpt-4o
    - anthropic:
        apiKey: ${ANTHROPIC_API_KEY}
        model: claude-sonnet-4-20250514
  tools:
    - browser
    - exec
    - web_search
```

### OpenClaw アーキテクチャにおける Nodes のデバイス機能拡張

Nodes は OpenClaw アーキテクチャで最も革新的な部分です。物理デバイスの機能を AI アシスタントに公開します。

- **Camera**：写真撮影、ビデオクリップ録画（前面/背面カメラ）
- **Screen**：スクリーンショット、画面録画
- **Location**：デバイスの地理位置情報取得（精度制御対応）
- **Canvas**：デバイス上での UI コンテンツのレンダリングと表示

```typescript
// Node 経由で写真を撮影する呼び出し例
const photo = await nodes.camera_snap({
  node: "jeffs-iphone",
  facing: "back",
  quality: 0.8
});

// デバイスの位置情報を取得
const location = await nodes.location_get({
  node: "jeffs-macbook",
  desiredAccuracy: "balanced"
});
```

現在、macOS・iOS・Android の 3 大プラットフォームに対応しています。各 Node は WebSocket で Gateway との常時接続を維持し、AI はいつでもデバイス機能を呼び出せます。

---

## まとめ：OpenClaw アーキテクチャの設計トレードオフ

OpenClaw アーキテクチャのコア理念は 3 つの言葉で要約できます：**シンプル・透明・拡張可能**。

- シングルプロセス Gateway は水平スケーリング能力を犠牲にし、デプロイとメンテナンスの究極のシンプルさを獲得
- Markdown メモリシステムはクエリ性能を犠牲にし、人間可読性とバージョン管理を獲得
- プラグイン化設計はすぐに使える豊富さを犠牲にし、無限の拡張可能性を獲得

これらのトレードオフの背後にあるロジックは一貫しています。OpenClaw アーキテクチャが対象とするのは個人向け AI アシスタントのシナリオであり、エンタープライズ SaaS ではありません。このシナリオでは、1 人のメッセージ量に分散システムは不要ですが、AI が本当にユーザーを理解し、記憶し、デバイスを操作できることが求められます。

---

> 🚀 安定した高コスパの AI API ゲートウェイで OpenClaw インスタンスを駆動したい方には、[Crazyrouter](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=openclaw_arch) がおすすめです。OpenAI・Claude・Gemini など主要モデルの統一 API 接続に対応し、従量課金で月額固定費はかかりません。

---

*本記事は OpenClaw の最新バージョンに基づいて執筆しています。ご質問があれば Twitter [@metaviiii](https://twitter.com/metaviiii) または Telegram でお気軽にどうぞ。*
