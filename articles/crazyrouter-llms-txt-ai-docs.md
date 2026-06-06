---
title: "AIコーディングツールにCrazyrouter公式ドキュメントを読ませる方法：llms.txt入門"
emoji: "🤖"
type: "tech"
topics: ["ai", "api", "llm", "cursor"]
published: true
---

# AIコーディングツールにCrazyrouter公式ドキュメントを読ませる方法：`llms.txt`入門

ChatGPT、Claude、Cursor、Cline、Aider、Codex CLI などのAIコーディングツールは便利ですが、API連携ではよく次のようなミスが起きます。

- base URL を間違える
- `/v1` を二重に付ける
- Chat Completions と Responses API を混同する
- Claude / Gemini / OpenAI 互換APIのルートを混ぜる
- 画像・動画・音声モデルに対して間違ったエンドポイントを使う
- 古い記憶からモデル名や価格を推測する

Crazyrouterでは、AIツール向けのドキュメント入口として `llms.txt` を用意しています。

```text
https://docs.crazyrouter.com/llms.txt
```

使い方ガイドはこちらです。

https://docs.crazyrouter.com/llms-guide?utm_source=zenn&utm_medium=article&utm_campaign=llms_txt_docs

## 何が便利なのか

`llms.txt` は、AIツールが最初に読むためのコンパクトなドキュメント索引です。

AIにこのURLを読ませると、以下のドキュメントを探しやすくなります。

- Quick Start
- Authentication
- API Endpoints
- Models API
- OpenAI互換 Chat Completions
- OpenAI Responses API
- Claude Messages API
- Gemini Native API
- 画像生成
- 動画生成
- TTS / STT
- Cursor / Cline / Claude Code / Codex CLI 連携

## おすすめプロンプト

AIツールに次のように伝えます。

```text
まず https://docs.crazyrouter.com/llms.txt を読んでください。
その後、Crazyrouter公式ドキュメントに基づいて回答してください。

やりたいこと：[ここに要件を書く。例：PythonでCrazyrouter経由のチャットAPIを呼び出す]

注意：
- OpenAI互換SDKのbase_urlは https://cn.crazyrouter.com/v1 を使う
- モデル名、価格、課金方式は https://crazyrouter.com/pricing を正とする
- モデルが利用可能か不明な場合は Pricing ページまたは /v1/models で確認するように伝える
- 実行可能なコード例を出す
```

## OpenAI互換SDKのbase URL

OpenAI互換SDKでは次を使います。

```text
https://cn.crazyrouter.com/v1
```

raw HTTP の Chat Completions なら次のようになります。

```text
https://cn.crazyrouter.com/v1/chat/completions
```

APIエンドポイントにはUTMパラメータを付けないでください。UTMは人間がクリックする記事リンク用です。

## モデルと価格はどこで確認する？

公開モデル、価格、課金方式は次を確認します。

```text
https://crazyrouter.com/pricing
```

APIキーごとに実際に呼び出せるモデルを確認する場合：

```bash
curl https://cn.crazyrouter.com/v1/models \
  -H "Authorization: Bearer YOUR_API_KEY"
```

本物のAPIキーを公開チャットや信頼できないAIツールに貼らないでください。コード例では `YOUR_API_KEY` のようなプレースホルダーを使います。

## `llms-full.txt` を使う場合

`llms.txt` は軽量な索引です。

AIツールが長いコンテキストを扱える場合は、完全版も利用できます。

```text
https://docs.crazyrouter.com/llms-full.txt
```

## Markdownページも読める

多くのドキュメントページは `.md` を付けるとMarkdownとして読めます。

```text
https://docs.crazyrouter.com/quickstart.md
https://docs.crazyrouter.com/api-endpoint.md
https://docs.crazyrouter.com/video/sora.md
```

## まとめ

AIコーディングツールにCrazyrouter連携コードを書かせるときは、最初にこれを渡すのが安全です。

```text
https://docs.crazyrouter.com/llms.txt
```

詳しい使い方：

https://docs.crazyrouter.com/llms-guide?utm_source=zenn&utm_medium=article&utm_campaign=llms_txt_docs
