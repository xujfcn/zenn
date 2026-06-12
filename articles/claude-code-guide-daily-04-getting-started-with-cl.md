---
title: "Claude CodeをCrazyrouterで始める：Trae、Z Code、Base URLの基本"
emoji: "🤖"
type: "tech"
topics: ["claudecode", "ai", "api", "crazyrouter"]
published: true
---

## Claude CodeをCrazyrouterで素早く使い始める

Claude Codeは、自然言語でリポジトリを読み、設計相談、コード生成、テスト作成、リファクタ案の整理を進められるAI開発支援ツールです。非エンジニアでも使いやすい理由は、複雑なコマンドを覚えるより先に「このエラーを説明して」「この機能の実装計画を作って」と会話できる点にあります。

参考リポジトリはこちらです。  
[Claude Code Guide](https://github.com/xujfcn/claude-code-guide?utm_source=zenn&utm_medium=article&utm_campaign=claude_code_guide_daily)

## まず覚えるBase URL

Crazyrouterを使う場合、クライアントの種類でエンドポイントを分けます。

- Claude Code / Anthropicネイティブ系: `https://cn.crazyrouter.com`
- OpenAI互換SDK、HTTPリクエスト、アプリ実装: `https://cn.crazyrouter.com/v1`

ここを混同すると、`/v1/v1/...` のようなパスになり、接続失敗の原因になります。APIエンドポイントにはUTMなどを付けないでください。

```bash
export ANTHROPIC_BASE_URL="https://cn.crazyrouter.com"
export ANTHROPIC_API_KEY="your_crazyrouter_token"

export OPENAI_BASE_URL="https://cn.crazyrouter.com/v1"
export OPENAI_API_KEY="your_crazyrouter_token"
```

## Traeで使う場合

TraeはVS Codeに近い操作感のIDEです。拡張機能ビューを開き、Claude Codeを検索してインストールします。Windows/Linuxでは`Ctrl+Shift+X`、macOSでは`Cmd+Shift+X`で拡張機能を開けます。

インストール後、画面上のClaude Codeアイコンからチャットを開きます。ログインやAPI設定が必要な場合、Claude Code向けのBase URLには必ず `https://cn.crazyrouter.com` を指定します。`/v1`付きはOpenAI互換SDK向けです。

Traeは、VS Codeに慣れている人、既存の拡張機能を使いたい人、通常のIDE作業とAI支援を同じ画面で扱いたい人に向いています。

## 画面で見るべきポイント

Claude Codeの中心はチャット入力欄です。依頼内容、制約、対象ファイル、期待する出力形式を具体的に書くと安定します。`/`でコマンド、`@`でファイル参照を呼び出せるUIもあります。

編集モードは慎重に選びます。最初はPlan modeかAsk before editsがおすすめです。自動編集は便利ですが、Gitで差分を確認し、テストできる状態で使うべきです。

## Z Codeで使う場合

Z CodeはCrazyrouterが提供する軽量なAI協同開発ツールです。コマンドライン操作に慣れていない開発者でも、Claude CodeなどのAIツールを視覚的なUIから扱いやすくすることを目的にしています。

基本手順は、Z Codeをインストールし、設定画面でClaude Codeを追加し、API Keyを入力して接続を確認する流れです。Alpha段階のツールとして扱い、重要なコードはGitで管理し、生成された変更は必ず確認しましょう。

## 使い分け

TraeはフルIDEに近い作業向けです。長期プロジェクト、拡張機能、通常の編集・確認作業を重視するなら適しています。

Z Codeは軽量なAI操作画面が欲しい場合に向いています。複数のAIツールを試す、プロトタイプを素早く作る、CLIを避けたい、といった用途に合います。

## 最初のおすすめワークフロー

いきなり大きな実装を任せるのではなく、まずは「構成を説明して」「関連ファイルを探して」「実装計画を出して」と段階化します。その後、小さな差分を生成し、テストし、問題なければコミットします。

設定で迷ったら、まずBase URLを確認してください。Claude Codeは `https://cn.crazyrouter.com`、OpenAI互換のアプリ実装は `https://cn.crazyrouter.com/v1`。この区別だけで、多くの初期トラブルを避けられます。
