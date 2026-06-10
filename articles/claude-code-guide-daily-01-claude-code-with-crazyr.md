---
title: "Claude CodeをCrazyrouterに接続する：Base URL設定と実務ワークフロー"
emoji: "🤖"
type: "tech"
topics: ["claudecode", "ai", "api", "crazyrouter"]
published: true
---

# Claude CodeをCrazyrouterに接続する：Base URL設定と実務ワークフロー

Claude CodeをCrazyrouter経由で使うとき、最初に確認すべき点はBase URLです。Claude CodeやAnthropicネイティブのクライアントでは `ANTHROPIC_BASE_URL=https://cn.crazyrouter.com` を使います。一方、OpenAI互換SDK、HTTPリクエスト、通常のアプリケーション実装では `base_url=https://cn.crazyrouter.com/v1` を使います。この2つを混同すると `/v1/v1/...` のようなパスになり、認証やルーティング以前の問題で失敗します。

GitHub: [claude-code-guide](https://github.com/xujfcn/claude-code-guide?utm_source=zenn&utm_medium=article&utm_campaign=claude_code_guide_daily)

## 前提環境

Node.js、npm、Gitが必要です。Claude Codeはnpmでグローバルインストールできます。Node.jsは18 LTS以降を推奨します。設定後はターミナルを再起動し、環境変数が反映されているか確認してください。

```bash
node --version
npm --version
git --version
npm install -g @anthropic-ai/claude-code
claude --version

export ANTHROPIC_BASE_URL=https://cn.crazyrouter.com
export ANTHROPIC_API_KEY=YOUR_CRAZYROUTER_API_KEY
claude
```

Windowsの場合も設定する変数は同じです。PowerShellのユーザー環境変数やターミナルプロファイルに保存しておくと、毎回入力する必要がありません。API Tokenは個人用・検証用・チーム用で分け、権限を最小化しておくと運用しやすくなります。

## 最初に覚えるコマンド

`claude --help` は必ず一度見ておくと便利です。よく使う操作は `/clear`、`/compact`、`/cost`、`/model`、`/status`、`/doctor` です。直前の会話に戻るなら `claude -c`、履歴から選ぶなら `claude -r`、スクリプト的に1回だけ実行するなら `claude -p` が使えます。

コンテキストが長くなったら、会話が崩れる前に `/compact` します。作業の区切りで「完了したこと、残タスク、変更ファイル」を要約させてから続けると、再開時の精度が安定します。

## CLAUDE.mdを運用メモにする

`CLAUDE.md` はプロジェクトの常駐メモとして使えます。アーキテクチャ、テスト方法、ローカルポート、コーディング規約、禁止事項を書いておくと、セッションをまたいでも同じ前提で作業しやすくなります。ただし長すぎるファイルは逆効果です。議事録ではなく、再利用するルールだけを残します。

おすすめのルールは「成功を宣言するときは、実行したコマンド、結果、関連ファイルを必ず示す」です。これにより、実際にはテストしていないのに完了と言ってしまう問題を減らせます。

## モードと安全性

小さな修正では自動編集が便利ですが、大きな変更では先にPlanを出させます。影響ファイル、変更手順、テスト方針が曖昧なら、実装に進ませず計画を直します。`claude --dangerously-skip-permissions` のような高権限モードは、コンテナ、サンドボックス、一時ブランチでだけ使うのが安全です。作業ディレクトリにSSH鍵や本番用シークレットを置かないことも重要です。

## まとめ

Claude CodeでCrazyrouterを使う設定はシンプルですが、Base URLの使い分けが最重要です。Claude Codeは `https://cn.crazyrouter.com`、OpenAI互換SDKは `https://cn.crazyrouter.com/v1`。そのうえで、Gitのチェックポイント、`CLAUDE.md`、`/compact`、利用量確認を組み合わせると、日常開発に組み込みやすいワークフローになります。
