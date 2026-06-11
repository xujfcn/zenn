---
title: "Claude CodeをCrazyrouterに接続する実践的なチーム運用"
emoji: "🤖"
type: "tech"
topics: ["claudecode", "ai", "api", "crazyrouter"]
published: true
---

# Claude CodeをCrazyrouterに接続する実践的なチーム運用

Claude Codeは単なる補完ツールではなく、リポジトリを読み、修正方針を考え、テストやGit操作まで含めて開発を支援するCLI型のAIコーディングアシスタントです。個人利用でも便利ですが、チームで使う場合は「どのモデルを使うか」よりも「接続先、トークン、レビュー、権限をどう統一するか」が重要になります。

参考リポジトリ: [Claude Code Guide](https://github.com/xujfcn/claude-code-guide?utm_source=zenn&utm_medium=article&utm_campaign=claude_code_guide_daily)

## Base URLの使い分け

Crazyrouterに接続するときは、クライアントの種類によってURLを分けます。

- Claude Code / Anthropicネイティブクライアント: `ANTHROPIC_BASE_URL=https://cn.crazyrouter.com`
- OpenAI互換SDK、HTTPリクエスト、Web/バックエンドアプリ: `base_url=https://cn.crazyrouter.com/v1`

ここを間違えると `/v1/v1/...` のようなパスになり、認証やルーティング以前の問題で失敗します。エンドポイントは次のまま使います。

- `https://cn.crazyrouter.com`
- `https://cn.crazyrouter.com/v1`

## 基本設定例

```bash
# Claude Code / Anthropic系
export ANTHROPIC_BASE_URL="https://cn.crazyrouter.com"
export ANTHROPIC_AUTH_TOKEN="your_crazyrouter_api_token"

# OpenAI互換SDK / アプリケーション側
export OPENAI_BASE_URL="https://cn.crazyrouter.com/v1"
export OPENAI_API_KEY="your_crazyrouter_api_token"
```

実際の変数名は利用するSDKや実行環境に合わせてください。重要なのは、Claude Code向けにはルートURL、OpenAI互換には `/v1` 付きURLを使うことです。

## チーム導入のポイント

まず、API Tokenは個人共用にしない方が安全です。プロジェクト単位、環境単位、チーム単位で分け、必要に応じてモデルの利用範囲も制限します。開発、ステージング、本番でトークンを分けておくと、障害対応やローテーションが楽になります。

次に、プロジェクトルートに `.claude/CLAUDE.md` を用意し、Claude Codeに守ってほしいルールを書きます。たとえば、命名規則、Conventional Commits、テスト必須、TypeScriptで`any`を避ける、Reactでは関数コンポーネントを優先する、といった内容です。曖昧な命名や複数概念に同じ名前を使う設計は、AIの誤解を増やします。

大きな変更では、いきなり編集させずにPlan Mode相当の進め方を取ります。「まず修正計画、対象ファイル、リスク、テスト方針を出して。まだ変更しないで」と依頼すると、方向違いの実装を早い段階で止められます。

## Gitを安全装置にする

AIが生成したコードも、最終責任は開発者にあります。ブランチを切り、小さくコミットし、差分を頻繁に確認します。Claude Codeが同じ修正を繰り返して失敗する場合は、会話をクリアし、最後の安定状態に戻し、失敗した方針を明示して再開します。

レビューは3段階に分けると現実的です。

1. 動作確認: アプリ起動、テスト実行、要件との一致確認
2. AIセルフレビュー: 重複、例外処理、テスト漏れ、型安全性を確認させる
3. 人間レビュー: 業務ロジック、セキュリティ、保守性、設計判断を見る

差分が大きすぎて1行ずつ追えない場合、タスクの切り方を小さくするべきです。

## 使いやすい場面

Claude Codeは、UIコンポーネント更新、API移行、CRUD実装、テスト追加、ドキュメント整備、既存コードのリファクタリングのように、方針が明確で繰り返しが多い作業に向いています。Playwright MCPを組み合わせると、ブラウザ上の再現、ログ収集、修正、確認まで進めやすくなります。

一方で、ライセンス検証、コアアルゴリズム、営業秘密、認証情報を含む設定ファイルなどはAIに読ませない設計が必要です。ignore設定やリポジトリ分割、権限制御を使い、プロンプトだけに頼らないようにします。

Crazyrouterは、Claude Codeとアプリケーション側SDKの接続先を整理し、チームの利用を統一するためのルーティング層として使えます。導入時は小さなリポジトリから始め、URL、トークン、レビュー手順を先に標準化するのがおすすめです。
