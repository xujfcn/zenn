---
title: "Claude Code + Crazyrouter：毎日使うショートカットと安全な作業習慣"
emoji: "🤖"
type: "tech"
topics: ["claudecode", "ai", "api", "crazyrouter"]
published: true
---

# Claude Code + Crazyrouter：毎日使うショートカットと安全な作業習慣

Claude Code を日常的に使うなら、モデル選定やプロンプトだけでなく、ショートカットと運用ルールも重要です。送信、検索、履歴確認、ファイルアップロード、モデル切り替えを毎回マウスで操作していると、小さな待ち時間が積み上がります。

Crazyrouter 経由で Claude Code を使う場合、まず Base URL を整理しておきます。

- Claude Code / Anthropic ネイティブクライアント: `ANTHROPIC_BASE_URL=https://cn.crazyrouter.com`
- OpenAI 互換 SDK / HTTP / アプリケーション: `base_url=https://cn.crazyrouter.com/v1`

`/v1` を二重に付けると `/v1/v1/...` のようなリクエストになり、設定ミスの原因になります。チームの README にこの2パターンを明記しておくと安全です。

```bash
export ANTHROPIC_BASE_URL="https://cn.crazyrouter.com"
export ANTHROPIC_API_KEY="your_crazyrouter_api_token"

# OpenAI 互換 SDK では別途:
# base_url="https://cn.crazyrouter.com/v1"
```

詳しい手順は [Claude Code Guide](https://github.com/xujfcn/claude-code-guide?utm_source=zenn&utm_medium=article&utm_campaign=claude_code_guide_daily) を参照してください。

## まず覚えたいショートカット

送信は Windows/Linux なら `Ctrl + Enter`、macOS なら `Cmd + Enter`。プロンプトを何度も改善する作業では、これだけでもかなり操作が楽になります。

新しい会話は `Ctrl/Cmd + Shift + N`。別タスクの文脈を混ぜないために、DB マイグレーション、UI 修正、リリースノート作成などは会話を分けるのがおすすめです。

設定は `Ctrl/Cmd + ,`、ヘルプは `Ctrl/Cmd + /`。オンボーディング時にも便利です。

編集系は通常のアプリと同じです。コピー `Ctrl/Cmd + C`、貼り付け `Ctrl/Cmd + V`、全選択 `Ctrl/Cmd + A`、取り消し `Ctrl/Cmd + Z`。ログや差分を扱うときは、途中で欠けた内容を送らないように注意します。

## 長い会話では検索を使う

会話が長くなったらスクロールより検索です。履歴検索は `Ctrl/Cmd + F`、ファイル検索は `Ctrl/Cmd + P`、全体検索は `Ctrl/Cmd + Shift + F`。関数名、API パス、エラー文、以前決めた方針を探すときに役立ちます。

おすすめの流れは、対象ファイルを検索し、現在の挙動を説明させ、最小変更の計画を出してもらい、生成結果を人間がレビューすることです。

## クイック操作も使い分ける

キーボード派でも、ファイルアップロード、コードブロック挿入、メッセージコピー、再生成などのボタンは有用です。特に複数ファイルを渡す場合、長いコードをチャットに貼るよりアップロードのほうが整理しやすくなります。

再生成は万能ではありません。回答が悪い原因がプロンプト不足なら、再生成よりも条件を明確にして再質問したほうがよいです。

## バッチ処理は境界を決める

複数ファイルのレビュー、ログ要約、設定比較、ドキュメント整形などは Claude Code に向いています。ただし、ファイルの意味、期待する出力形式、確認すべき観点を明示しましょう。

また、よく使う依頼はテンプレート化します。例として「この diff をセキュリティ、破壊的変更、テスト不足の観点でレビューしてください」のような定型文を用意しておくと、出力を比較しやすくなります。

## AI に任せすぎない

Claude Code は下書き、説明、リファクタリング案、テスト案の作成に強い一方、最終判断者ではありません。認証、課金、権限、マイグレーション、顧客向け文面などは必ず人間がレビューします。

実務では、背景を渡す、計画を出させる、前提を確認する、最小変更を依頼する、テスト方法を確認する、理解できる変更だけ適用する、という流れが安全です。

ショートカットは操作の摩擦を減らします。正しい Base URL は無駄なデバッグを減らします。そして、レビューと記録の習慣が Claude Code をチームの信頼できる開発ツールにします。
