---
title: "1つのMCPサーバーで627以上のAIモデルにアクセスする方法"
emoji: "🤖"
type: "tech"
topics: ["mcp", "ai", "api", "claude", "openai"]
published: true
---

# Crazyrouter MCP の設定（2026-10-12 更新）

Node.js 22+ が必要です。現在の `crazyrouter-mcp@1.0.1` は `chat`、`list_models`、`generate_image` の3ツールを提供します。動画・音楽生成はこの MCP の機能ではありません。

```json
{
  "mcpServers": {
    "crazyrouter": {
      "command": "npx",
      "args": ["-y", "crazyrouter-mcp@1.0.1"],
      "env": { "CRAZYROUTER_API_KEY": "YOUR_CRAZYROUTER_API_KEY" }
    }
  }
}
```

モデル一覧は API キーで取得します。カタログに載っていても、すべてのモデルを同じツールで呼び出せるわけではありません。画像生成は1枚の同期 Images API に対応し、URL と base64 の結果を扱います。

実際の呼び出しには通常の API 料金がかかります。一律割引や無料クレジットは保証しません。

[最新 README](https://github.com/xujfcn/crazyrouter-mcp) · [料金](https://crazyrouter.com/en/pricing) · [API ドキュメント](https://docs.crazyrouter.com)
