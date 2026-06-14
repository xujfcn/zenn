---
title: "Claude CodeをCrazyrouterで使う：最初の会話、ファイル操作、文章生成の基本"
emoji: "🤖"
type: "tech"
topics: ["claudecode", "ai", "api", "crazyrouter"]
published: true
---

# Claude CodeをCrazyrouterで使う基本操作

この記事では、Claude CodeをCrazyrouter経由で使い始めた直後に必要になる操作を整理します。対象は、VS Code上でClaude Codeを使い、国内向けのモデル接続やチーム内の呼び出し経路をCrazyrouterにまとめたい開発者です。

まずエンドポイントの使い分けを確認します。Claude CodeやAnthropicネイティブクライアントでは `ANTHROPIC_BASE_URL=https://cn.crazyrouter.com` を使います。OpenAI互換SDK、HTTPリクエスト、アプリケーション側の実装では `base_url=https://cn.crazyrouter.com/v1` を使います。ここを混同すると `/v1/v1/...` のような誤ったURLになりやすいので注意してください。

参考リポジトリ: [Claude Code Guide](https://github.com/xujfcn/claude-code-guide?utm_source=zenn&utm_medium=article&utm_campaign=claude_code_guide_daily)

## 最初の会話

Claude Codeのチャット欄に質問や依頼を書き、`Enter`で送信します。複数行を書きたい場合は `Shift + Enter` を使います。最初は「このプロジェクトの構成を説明して」「READMEを要約して」「認証処理がどこにあるか探して」など、小さな依頼から始めると扱いやすくなります。

長い回答が返ってきた場合は、すぐに編集を適用せず、内容を確認します。特にコード変更を伴う場合は、先に方針を聞いてから段階的に進めるのが安全です。

## `@`でファイルを指定する

Claude CodeはVS Codeのワークスペース内のファイルを直接参照できます。もっとも確実なのは、入力欄で `@` を打って対象ファイルを選ぶ方法です。

```text
@src/services/UserService.ts このサービスの責務を説明してください。まだ編集しないでください。
@package.json scriptsと依存関係を説明してください。
@README.md セットアップ手順を現在の構成に合わせて更新してください。
```

複数ファイルを同時に指定することもできます。たとえば `@Button.tsx @Button.test.tsx` として、実装とテストの整合性を確認できます。

## ファイル操作の基本

Claude Codeには、読み取り、説明、編集、新規作成、リネーム、削除を依頼できます。ただし削除や大規模変更は慎重に行います。おすすめは次の流れです。

1. 対象ファイルを指定する
2. 現状を説明してもらう
3. 修正方針を出してもらう
4. 1ファイルまたは1機能だけ変更する
5. 差分を確認する
6. テストやlintを実行する

例として、`@app.js console.logをlogger.infoに置き換えて。ただしエラー処理は変更しないでください` のように、変更点と制約を同時に伝えると意図が伝わりやすくなります。

## 文章生成にも使う

Claude Codeはコードだけでなく、README、PR説明、リリースノート、作業チェックリスト、会議メモの整理にも使えます。よい出力を得るには、読者、長さ、形式、トーンを指定します。

悪い例は「ドキュメントを書いて」です。よい例は「バックエンド開発者向けに、環境変数、起動方法、テスト、よくあるエンドポイント設定ミスを含む5ステップのセットアップ手順をREADMEに追加してください」です。

## Crazyrouter設定の確認

接続で失敗したら、まずログを確認し、次にBase URLを確認します。Claude Codeでは `https://cn.crazyrouter.com`、OpenAI互換では `https://cn.crazyrouter.com/v1` です。APIトークンはプロジェクト単位で分け、ソース管理には含めないようにします。

小さく依頼し、差分を確認しながら進めることが、Claude Codeを日常の開発に安全に組み込むコツです。
