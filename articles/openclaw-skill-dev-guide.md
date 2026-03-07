---
title: "OpenClaw Skill開発ガイド：AIアシスタントに新しいスキルを教える方法"
emoji: "🧩"
type: "tech"
topics: ["openclaw", "ai", "plugin", "skill", "automation"]
published: true
---

## はじめに

なぜ一部のAIアシスタントはチャットしかできず、別のAIアシスタントは天気確認・動画ダウンロード・メール管理・レポート自動生成までこなせるのでしょうか？その秘密は**OpenClaw Skill**——AIアシスタントがRPGのスキルポイントのように新しい能力を獲得できる拡張メカニズムです。

本記事では、OpenClaw Skillの仕組みをゼロから理解し、最初のSkillを開発する方法を解説。5つの実践的なケースで威力を体感していただきます。

---

## OpenClaw Skillの仕組み

開発に入る前に、コア設計哲学を理解しましょう。**Skillsはコードライブラリではなく、指示書です。** AIアシスタントに「特定の問題をどう解決するか」を教えるもので、実行プログラムそのものではありません。

### 3層ロードメカニズム

OpenClawは3層の優先順位でSkillsを管理します：

| レイヤー | 場所 | 優先度 | 説明 |
|---------|------|-------|------|
| **Bundled** | インストールパッケージに同梱 | 最低 | 内蔵スキル（`weather`、`tmux`、`healthcheck`など） |
| **Managed/Local** | `~/.openclaw/skills/` | 中 | ClawHubからインストールまたは手動追加 |
| **Workspace** | `<workspace>/skills/` | 最高 | プロジェクトレベルスキル |

**同名競合の解決ルール**：高優先度レイヤーのSkillが低優先度の同名Skillを上書きします。ワークスペースにカスタム`weather` Skillを置けば、内蔵版が置き換わります。システムファイルを変更せずに、任意のスキルを柔軟に上書き・拡張できる設計です。

### Skillのロードフロー

Agentが応答する前に自動実行される処理：

1. `available_skills`リスト内の全登録Skillをスキャン
2. ユーザーメッセージと各Skillの`description`フィールドをマッチング
3. マッチしたSkillの`SKILL.md`を読み込み、詳細な実行指示を取得
4. 指示に従って対応するToolsを呼び出しタスクを完了

### SkillとToolの違い

| 軸 | Skill（スキル） | Tool（ツール） |
|----|----------------|---------------|
| **本質** | 指示書——AIに「やり方」を教える | 能力——AIに「手段」を提供する |
| **形式** | Markdownドキュメント＋オプションスクリプト | システムレベル関数（exec、read、writeなど） |
| **ロード** | オンデマンド読み込み、マッチ後にロード | 常時利用可能 |
| **カスタマイズ** | ユーザーが自由に作成可能 | プラットフォームが提供 |

例えるなら：Toolは手持ちのハンマーとドライバー、Skillは「先にネジを締めてから釘を打つ」と教える操作マニュアルです。

---

## 最初のSkillを開発する

### ディレクトリ構造規範

標準的なSkillディレクトリ構造：

```
my-skill/
├── SKILL.md          # エントリーファイル（必須）
├── scripts/          # スクリプトファイル（オプション）
│   ├── run.sh
│   └── helper.py
├── templates/        # テンプレートファイル（オプション）
│   └── output.md
└── assets/           # 静的リソース（オプション）
    └── config.json
```

**核心原則**：`SKILL.md`が唯一のエントリーファイルです。AgentはこのファイルのみからSkillの使い方を理解します。

### SKILL.mdの書き方テンプレート

優秀な`SKILL.md`に含むべき要素：

```markdown
# My Awesome Skill

## Description
このSkillが何をするか、いつトリガーされるかを一文で説明。

## When to Use
- ユーザーがXXXに言及した時
- ユーザーがYYYを要求した時
- ZZZのシナリオには適用しない

## Prerequisites
- 環境変数の設定が必要：MY_API_KEY
- インストール必要：特定の依存パッケージ

## Instructions

### Step 1: 入力取得
ユーザーメッセージからキーパラメータを抽出...

### Step 2: 操作実行
スクリプトを実行：`./scripts/run.sh <パラメータ>`

### Step 3: 結果返却
出力をフォーマットしてユーザーに返す...

## Error Handling
- APIタイムアウト時：1回リトライ
- パラメータ不足時：ユーザーに補完を依頼

## Examples
ユーザー：「東京の天気を教えて」
→ weather skill実行、パラメータ location=Tokyo
```

#### ベストプラクティス

1. **Descriptionは正確に**：Agentがマッチング判断に使うため、曖昧だと誤トリガーや未トリガーの原因に
2. **Instructionsは具体的に**：賢いが文脈を知らない同僚への操作マニュアルのように書く
3. **相対パス参照**：スクリプトやリソースはSKILL.mdからの相対パスで指定
4. **エラーハンドリング必須**：異常時のグレースフルデグラデーション方法を記載

### スクリプトファイルの管理

Skill内のスクリプトは通常`exec` Toolで実行されます：

```bash
#!/bin/bash
# scripts/fetch_data.sh - データ取得スクリプト

API_KEY="${MY_API_KEY:?Error: MY_API_KEY not set}"
QUERY="$1"

if [ -z "$QUERY" ]; then
  echo "Usage: fetch_data.sh <query>"
  exit 1
fi

curl -s "https://api.example.com/data?q=${QUERY}" \
  -H "Authorization: Bearer ${API_KEY}" | jq '.'
```

---

## 実践：5つのSkill開発ケース

### ケース1：天気照会Skill

OpenClaw内蔵の定番Skillです。無料サービスwttr.inを利用し、API Key不要で入門に最適：

```markdown
# Weather Skill

## Description
天気予報を取得。ユーザーが天気・気温・予報に言及した時にトリガー。

## Instructions
1. curlでwttr.inにリクエスト：
   curl -s "wttr.in/{location}?format=j1"
2. JSONから現在気温・体感温度・天気状況を解析して返却
3. 予報が必要なら3日分を表示
```

### ケース2：動画ダウンロードSkill

yt-dlpツールと組み合わせた設計：

```markdown
# Video Download Skill

## Description
YouTube、Twitterなどから動画をダウンロード。動画リンクが提供された時にトリガー。

## Instructions
1. リンク元のプラットフォームを識別
2. 実行：yt-dlp -f "best[filesize<50M]" -o "output.mp4" <URL>
3. 50MBを超える場合はffmpegで圧縮
4. ユーザーにファイルを送信
```

プラットフォーム別のサイズ制限対応も重要です：Telegram 50MB（Premium 2GB）、Discord 25MB（Nitro 500MB）。

### ケース3：データベース照会Skill（セキュリティ重視）

```markdown
# Database Query Skill

## Security
- SELECT操作のみ許可
- DROP、DELETE、UPDATE、ALTERは禁止
- 機密フィールドは自動マスキング
```

### ケース4：メール管理Skill

メール送受信・テンプレート管理・一括送信を統合。送信前に必ずHTMLプレビューを生成し確認を取る設計。

### ケース5：自動レポートSkill

GA4 API + Search Console APIからデータを取得し、Markdownレポートを自動生成。Cronと連携すれば毎朝9時に自動生成・Telegram通知で完全自動化運営が実現します。

---

## テストと公開

### テストチェックリスト

公開前に以下のテストを実施：

1. **トリガーテスト**：descriptionが正しくマッチされるか
2. **正常パス**：主要機能が正常動作するか
3. **異常処理**：パラメータ不足・APIタイムアウト・権限不足の境界ケース
4. **多層テスト**：WorkspaceとManagedディレクトリそれぞれでロードし、優先順位の動作を検証

#### デバッグのヒント

- SKILL.mdに`## Debug`セクションを追加し、デバッグモードで詳細ログを出力
- `skill-creator`内蔵Skillを活用してSkill構造の生成と検証を補助
- `available_skills`リストでSkillが正しく登録されているか確認

### ClawHubへの公開

[ClawHub](https://clawhub.com)はOpenClaw公式のスキル共有プラットフォームです。Skill名は小文字＋ハイフン（例：`weather-pro`）、明確なdescriptionとusage説明、2つ以上の使用例の提供が推奨されます。

---

## まとめ

OpenClaw Skillメカニズムは、AIアシスタントの能力拡張をエレガントに解決します。コードプラグインの実装もモデルの再訓練も不要——Markdownで「やり方」を書くだけで、AIが新しいスキルを習得します。

3層ロードメカニズムが柔軟性を保証し、SKILL.md規範が開発ハードルを下げ、ClawHubプラットフォームがスキル共有をアプリインストール並みに簡単にしました。

---

> 🚀 各種AI モデルAPIの統合をお考えなら、[Crazyrouter](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=openclaw_skill)がおすすめです。統一AI APIゲートウェイで、1つのKeyでOpenAI・Claude・Geminiなど主要モデルを呼び出せます。OpenClaw開発者の理想的なパートナーです。
