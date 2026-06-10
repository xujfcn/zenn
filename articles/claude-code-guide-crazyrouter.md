---
title: "Claude Code 中文実践ガイド：Crazyrouter で Claude / GPT / 国内モデルを統合する"
emoji: "🤖"
type: "tech"
topics: ["claudecode", "ai", "api", "crazyrouter"]
published: true
---

Claude Code は強力な AI コーディングツールですが、実際の導入では設定ミスがよく起きます。

特に多いのが Base URL の違いです。

Claude Code / Anthropic 互換クライアントでは、ルート URL を使います。

```bash
export ANTHROPIC_BASE_URL=https://cn.crazyrouter.com
export ANTHROPIC_API_KEY=YOUR_CRAZYROUTER_API_KEY
claude
```

一方、OpenAI 互換 SDK や HTTP API では `/v1` を含む URL を使います。

```text
https://cn.crazyrouter.com/v1
```

この違いを間違えると、`/v1/v1/messages` のようなパス重複エラーが発生することがあります。

この問題を整理するために、Claude Code の中国語実践ガイドを公開しました。

GitHub:
https://github.com/xujfcn/claude-code-guide?utm_source=zenn&utm_medium=article&utm_campaign=claude_code_guide

内容：

- Claude Code のインストールと基本操作
- Crazyrouter との連携
- Base URL の正しい使い分け
- プロンプト設計
- PRD からコード生成までの流れ
- Figma / Mermaid を使った設計ワークフロー
- AI Coding 実践例

Claude Code を使った開発ワークフローを体系的に学びたい人向けのリポジトリです。
