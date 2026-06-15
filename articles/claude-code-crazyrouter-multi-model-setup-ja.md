---
title: "Claude Codeを複数モデル対応にする：CrazyrouterでClaude、GPT、Geminiを1つのキーにまとめる"
emoji: "🤖"
type: "tech"
topics: [claude, ai, api, coding]
published: true
---

# Claude Codeを複数モデル対応にする：CrazyrouterでClaude、GPT、Geminiを1つのキーにまとめる

Claude Codeを使っていると、最初は「Claudeだけで十分」と感じるかもしれません。しかし実際の開発では、タスクによって使いたいモデルが変わります。

複雑な設計レビューにはClaude Opusを使いたい。軽いコード生成や要約にはGPTやGeminiでも十分。DeepSeek系モデルを試したい場面もあります。

問題は、モデルごとにAPIキー、Base URL、課金、環境変数を管理し始めると、ローカル開発環境がすぐに複雑になることです。

そこで使えるのが、このセットアップページです。

https://xujfcn.github.io/crazyrouter-claude-code/

Crazyrouterを経由してClaude Codeを設定することで、1つのトークンから複数の主要モデルにアクセスできます。

## 何が便利なのか

Claude Codeは単なるチャットではありません。コードを読み、変更案を出し、エラーを説明し、開発ワークフローに直接入ってくるAI coding agentです。

そのため、モデル設定は開発体験に直結します。

よくある失敗は次のようなものです。

- APIキーが違う
- Base URLが間違っている
- `/v1` が抜けている
- 新しいターミナルを開いていない
- macOSとWindowsで設定方法が違う
- チームメンバーごとに環境がばらばら

この種の問題は、手作業よりもスクリプトで統一した方が安全です。

## Claude Codeがすでに入っている場合

すでにClaude Codeをインストール済みなら、configure-onlyのコマンドを使います。

macOS / Linux:

```bash
curl -fsSL https://raw.githubusercontent.com/xujfcn/crazyrouter-claude-code/main/configure.sh | bash
```

Windows PowerShell:

```powershell
irm https://raw.githubusercontent.com/xujfcn/crazyrouter-claude-code/main/windows/configure.ps1 | iex
```

## まだClaude Codeを入れていない場合

Claude Code自体がまだない場合は、full installを使います。

macOS / Linux:

```bash
curl -fsSL https://raw.githubusercontent.com/xujfcn/crazyrouter-claude-code/main/setup.sh | bash
```

Windows PowerShell:

```powershell
irm https://raw.githubusercontent.com/xujfcn/crazyrouter-claude-code/main/windows/setup.ps1 | iex
```

先にCrazyrouterのトークンを用意しておきます。

https://cn.crazyrouter.com

## 設定される環境変数

スクリプトはユーザーレベルの環境変数を設定します。

```text
ANTHROPIC_BASE_URL=https://cn.crazyrouter.com
ANTHROPIC_AUTH_TOKEN=<your token>
ANTHROPIC_MODEL=claude-opus-4-8
CLAUDE_MODEL=claude-opus-4-8
OPENAI_API_KEY=<your token>
OPENAI_BASE_URL=https://cn.crazyrouter.com/v1
```

ポイントは、Anthropic形式とOpenAI互換形式の両方を設定することです。

OpenAI互換ツールでは、Base URLに `/v1` が必要になることが多いです。この違いを手作業で管理するとミスが起きやすいため、スクリプト化する価値があります。

## どんな開発者に向いているか

このセットアップは次のような人に向いています。

- Claude Codeを日常的に使う
- 複数モデルを試したい
- APIキー管理を簡単にしたい
- WindowsとmacOSの両方で使いたい
- チームに同じ設定手順を配りたい
- OpenAI互換ツールも同じキーで使いたい

## まとめ

AI coding agentの品質は、モデルそのものだけでなく、設定の再現性にも左右されます。

Crazyrouter Claude Code setupは、Claude Codeを複数モデル対応にするための実用的な入口です。

https://xujfcn.github.io/crazyrouter-claude-code/

Claude Codeを使っていて、1つのキーでClaude、GPT、Gemini、DeepSeekを扱いたいなら、まず試す価値があります。


セットアップページ: https://xujfcn.github.io/crazyrouter-claude-code/?utm_source=zenn&utm_medium=article&utm_campaign=claude_code_crazyrouter_setup
