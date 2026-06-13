---
title: "CrazyrouterでClaude Codeから国内モデルへ統一接続する"
emoji: "🤖"
type: "tech"
topics: ["claudecode", "ai", "api", "crazyrouter"]
published: true
---

# CrazyrouterでClaude Codeから国内モデルへ統一接続する

Claude Codeを日常の開発に入れると、リポジトリの読解、リファクタリング、テスト作成、仕様確認などを同じCLI上で進められます。ここに国内モデルも組み合わせたい場合、課題になるのは「どのモデルを使うか」だけではありません。APIキー、エンドポイント、権限、ログ確認をどう統一するかが重要です。

その入口として使えるのがCrazyrouterです。Claude CodeやOpenAI互換アプリをCrazyrouter経由にすると、モデルごとに別々の鍵や管理画面を扱う必要が減ります。

完全なガイドはこちらです: [Claude Code Guide](https://github.com/xujfcn/claude-code-guide?utm_source=zenn&utm_medium=article&utm_campaign=claude_code_guide_daily)

## まず覚えるべきエンドポイント

Claude CodeとOpenAI互換SDKでは、設定するBase URLが違います。

- Claude Code / Anthropic系クライアント: `https://cn.crazyrouter.com`
- OpenAI互換SDK / HTTP API / アプリ: `https://cn.crazyrouter.com/v1`

Claude Codeは内部で `/v1/messages` を組み立てます。そのため、`ANTHROPIC_BASE_URL` に `/v1` まで入れると、`/v1/v1/messages` のような誤ったパスになりやすいです。

## Claude Codeの設定

macOSやLinuxでは、環境変数に以下を設定します。

```bash
export ANTHROPIC_BASE_URL=https://cn.crazyrouter.com
export ANTHROPIC_API_KEY=YOUR_CRAZYROUTER_API_KEY
claude

curl https://cn.crazyrouter.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_CRAZYROUTER_API_KEY" \
  -d '{
    "model": "deepseek-v4-pro",
    "messages": [
      {"role": "user", "content": "Crazyrouterの役割を一文で説明してください"}
    ]
  }'
```

Windows PowerShellでも同じ考え方です。ユーザー環境変数として `ANTHROPIC_BASE_URL` に `https://cn.crazyrouter.com`、`ANTHROPIC_API_KEY` にCrazyrouterのTokenを設定します。設定後はターミナルを開き直してから `claude` を実行してください。

起動後は、まず `/status` を実行すると現在の状態を確認しやすくなります。

## OpenAI互換アプリの場合

自作バックエンド、SDK、HTTPリクエスト、フロントエンドの中継サービスなどでOpenAI互換APIを使う場合は、Base URLに `/v1` を含めます。

`https://cn.crazyrouter.com/v1`

ここをClaude Codeの設定と混同しないことが重要です。

## Tokenは分けて管理する

Claude Code専用のAPI Tokenを作ることをおすすめします。理由はシンプルです。

- Claude Code用のモデル許可リストを作れる
- 予算を分けられる
- ログを他のアプリの通信と混ぜずに確認できる
- 端末紛失やメンバー変更時に対象Tokenだけをローテーションできる

チームで使う場合、ひとつのTokenを全員で共有するよりも、用途ごとに分けたほうが調査と運用が楽になります。

## モデル選択の考え方

国内モデルを使う場合でも、各ベンダーに個別接続する前提で考える必要はありません。Crazyrouterのモデル一覧とToken権限を基準にします。

- コード読解、リファクタリング、複雑な開発作業: Claude系またはClaude Code互換性を確認済みのCodingモデル
- 日本語・中国語ドキュメント、要約、コンテンツ処理: 言語処理に強くコストが合う汎用モデル
- 長文解析: コンテキストが長く、出力が安定したモデル
- バッチ処理: コストとスループットを重視し、Token予算を設定

利用可能なモデルはCrazyrouterのコンソール、または `GET https://cn.crazyrouter.com/v1/models` の結果を確認します。

## よくあるエラー

ログに `/v1/v1/messages` や `/v1/v1/models` が出る場合、Base URLに `/v1` を重複して入れている可能性が高いです。

Claude Codeでは `https://cn.crazyrouter.com`、OpenAI互換では `https://cn.crazyrouter.com/v1` と覚えてください。

`model not allowed` や403が返る場合は、Tokenのモデル許可リストを確認します。サンプルの `YOUR_CRAZYROUTER_API_KEY` は必ず実際の環境変数に置き換え、実TokenをGitへコミットしないようにします。

## まとめ

おすすめの流れは、Claude Code専用Tokenを作成し、`ANTHROPIC_BASE_URL=https://cn.crazyrouter.com` を設定し、Claude Codeの `/status` とCrazyrouterのログで疎通を確認することです。

この形にしておくと、Claude系モデルと国内モデルを同じ運用ルールで扱いやすくなります。重要なのは、Claude CodeではルートURL、OpenAI互換では `/v1` 付きURLを使うという切り分けです。
