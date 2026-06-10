---
title: "Claude CodeをCrazyrouterに接続する実践メモ"
emoji: "🤖"
type: "tech"
topics: ["claudecode", "ai", "api", "crazyrouter"]
published: true
---

# Claude CodeをCrazyrouterに接続する実践メモ

Claude Codeは、特定の開発フローを押し付けない低レベルなCLI型コーディングエージェントです。その自由度は便利ですが、チームで使うなら接続設定、権限、プロジェクト文脈、GitHub連携を先に整えておくべきです。

参考リポジトリ: [Claude Code Guide](https://github.com/xujfcn/claude-code-guide?utm_source=zenn&utm_medium=article&utm_campaign=claude_code_guide_daily)

## Base URLを間違えない

Crazyrouter接続で最も多いミスは、エンドポイントの使い分けです。

- Claude Code / Anthropicネイティブクライアント: `ANTHROPIC_BASE_URL=https://cn.crazyrouter.com`
- OpenAI互換SDKやHTTPアプリ: `base_url=https://cn.crazyrouter.com/v1`

Claude Code側に `/v1` を付けると、クライアント実装によっては `/v1/v1/...` のような不正なパスになります。まずここを固定ルールにしましょう。

```bash
export ANTHROPIC_BASE_URL="https://cn.crazyrouter.com"
export ANTHROPIC_AUTH_TOKEN="your_crazyrouter_api_token"
claude --version
claude
```

API Tokenは個人用を使い回さず、プロジェクトまたはチーム単位で分けるのが安全です。後からローテーション、利用状況の確認、モデル許可リストの管理がしやすくなります。

## `CLAUDE.md`を小さく育てる

Claude Codeは会話開始時に `CLAUDE.md` を文脈として読み込みます。ここにはビルド、テスト、型チェック、ディレクトリ構成、コード規約、PRルール、既知の注意点を書きます。

ただし長い社内Wikiを貼るのは逆効果です。プロンプト文脈を消費するため、短く具体的にします。例えば「PR前に `npm run typecheck` を必ず実行」「生成ファイルは直接編集しない」「GitHub操作は `gh` を使う」のような指示が有効です。

配置場所はリポジトリルートが基本です。monorepoなら親ディレクトリや各パッケージ配下にも置けます。個人共通ルールは `~/.claude/CLAUDE.md` に置けます。

## 権限は段階的に許可する

Claude Codeはファイル編集、bash実行、MCPツールなどに対して確認を求めます。これは面倒に見えますが、実コードベースでは重要な安全策です。

権限管理は、セッション中の「常に許可」、`/permissions`、`.claude/settings.json`、起動時の `--allowedTools` で行えます。最初から全許可にせず、繰り返し発生する安全な操作だけを追加するのが実務向きです。

## GitHub CLIとMCPを活用する

GitHubを使うなら `gh` を入れて `gh auth login` を済ませます。ClaudeはIssue作成、PR作成、レビューコメント確認、履歴調査などを `gh` 経由で扱えます。

外部ツール連携にはMCPも使えます。ブラウザ操作、Sentry、社内検索などを `.mcp.json` に定義すれば、チーム全体で同じ道具を使えます。問題調査には `claude --mcp-debug`、状態確認には `/mcp` が便利です。

## おすすめの作業フロー

汎用的には「探索、計画、実装、検証、コミット」が安定します。最初に関連ファイルを読ませ、「まだコードを書かない」と明示します。次に計画を作らせ、必要なら修正してから実装に進みます。

仕様が明確な変更ではTDDが有効です。先にテストを書かせ、失敗を確認し、テストを固定してから実装させます。UIではスクリーンショットやモック画像を渡し、実装後の見た目を反復確認させると精度が上がります。

長い会話では `/clear` を使って文脈を整理します。大きな移行やlint修正ではMarkdownのチェックリストを作らせ、1件ずつ修正・検証させると追跡しやすくなります。

## 自動化と複数セッション

`claude -p` は非対話モードです。CI、pre-commit、ログ解析、差分レビューなどに向いています。JSON出力が必要なら `--output-format json` や `--output-format stream-json` を使います。

大きな作業では複数のClaudeを分けるのも有効です。1つ目が実装し、2つ目がレビューする。あるいはGit worktreeで別ブランチを別ディレクトリに展開し、独立したタスクを並列に進めます。

Crazyrouter接続自体はシンプルです。重要なのは、正しいBase URL、短い `CLAUDE.md`、適切な権限、検証しやすいワークフローをそろえることです。
