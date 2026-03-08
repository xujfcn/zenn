---
title: "1つのMCPサーバーで627以上のAIモデルにアクセスする方法"
emoji: "🤖"
type: "tech"
topics: ["mcp", "ai", "api", "claude", "openai"]
published: true
---

## 複数のAI APIキーを管理する煩わしさ

OpenAI用、Anthropic用、Google用...それぞれのAPIキーを管理していませんか？

**Crazyrouter MCP Server**を使えば、1つのAPIキーで627以上のAIモデルにアクセスできます。

## できること

| ツール | 機能 |
|--------|------|
| `chat` | GPT-5, Claude, Gemini, DeepSeek等627+モデルと会話 |
| `list_models` | カテゴリ別モデル一覧（chat/image/video/audio/music） |
| `generate_image` | DALL-E 3, Midjourney, Flux, Imagen 4.0で画像生成 |
| `generate_video` | Sora 2, Kling V2, Veo 3で動画生成 |

## セットアップ（3分）

### 1. APIキー取得

[crazyrouter.com](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=dev_community)でアカウント作成

### 2. インストール

```bash
git clone https://github.com/xujfcn/crazyrouter-mcp.git
cd crazyrouter-mcp
npm install && npm run build
```

### 3. Claude Desktopに設定

macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`

```json
{
  "mcpServers": {
    "crazyrouter": {
      "command": "node",
      "args": ["/path/to/crazyrouter-mcp/dist/index.js"],
      "env": {
        "CRAZYROUTER_API_KEY": "sk-your-key"
      }
    }
  }
}
```

## テスト結果

実際にAPIを叩いて確認済み：

```
✅ chat (gpt-4o-mini) → 即時応答、22トークン
✅ list_models → APIで539モデル確認
✅ generate_image (DALL-E 3) → 約8秒で画像URL返却
```

## なぜCrazyrouter？

| 項目 | OpenAI MCP | Crazyrouter MCP |
|------|-----------|----------------|
| モデル数 | OpenAIのみ | 627+（全プロバイダー） |
| 画像生成 | DALL-Eのみ | DALL-E + MJ + Flux + SD + Imagen |
| 動画生成 | ❌ | Sora + Kling + Veo等 |
| 必要なAPIキー | プロバイダーごと | 1つだけ |
| 料金 | 公式価格 | 公式より安い |

**GitHub**: [xujfcn/crazyrouter-mcp](https://github.com/xujfcn/crazyrouter-mcp?utm_source=zenn&utm_medium=article&utm_campaign=dev_community)
