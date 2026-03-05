---
title: "【2026年版】OpenRouterの代替サービス7選：AI APIゲートウェイ徹底比較"
emoji: "🔀"
type: "tech"
topics: ["AI", "OpenAI", "API", "LLM", "ChatGPT"]
published: true
---

OpenRouterは「1つのエンドポイントで数百のモデルにアクセス」という手軽さで人気ですが、利用量が増えるとマークアップ料金・マルチモーダル非対応・セルフホスト不可が課題になります。

7つの代替サービスを実際にAPIを叩いて比較しました。

## 一覧比較表

| プラットフォーム | モデル数 | 料金 | セルフホスト | マルチモーダル | 最適な用途 |
|:--|:--|:--|:--|:--|:--|
| **OpenRouter** | 300+ | 公式+10-30%加算 | ❌ | LLMのみ | プロトタイピング |
| **Crazyrouter** | 627+ | 公式の約55% | ❌ | ✅ 全対応 | コスト重視 |
| **Portkey** | 1,600+(BYOK) | Free〜$49/月 | ✅ | LLMのみ | エンタープライズ |
| **LiteLLM** | 100+ | 無料(OSS) | ✅ | LLMのみ | セルフホスト |
| **Helicone** | BYOK | Free〜$20/月 | ✅ | LLMのみ | 可視化 |
| **Unify AI** | 80+ | 従量課金 | ❌ | LLMのみ | 自動ルーティング |
| **Kong AI GW** | プラグイン | エンタープライズ | ✅ | LLMのみ | Kong既存ユーザー |
| **CF AI GW** | BYOK | 無料 | ❌ | LLMのみ | CFユーザー |

## 1. Crazyrouter — 最安のマルチモーダルAPIゲートウェイ

https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=openrouter_alt

OpenRouterの最大の不満が**価格**なら、Crazyrouterが最も直接的な代替です。627以上のモデルを公式の約55%で提供。月額料金なし。

### 他サービスとの違い

LLMだけでなく、AIモデルの全領域をカバー：

- **LLM**：GPT-5/5.2、Claude Opus 4.6、Gemini 3 Pro、DeepSeek V3.2/R1、Grok 4、Qwen 3
- **画像生成**：DALL-E 3、Midjourney全シリーズ、Flux Pro/Dev、SD 3.5
- **動画生成**：Sora 2、Kling V2.6、Veo 3/3.1、Runway Gen4
- **音楽生成**：Suno V4、Chirp V4

### 料金比較

| モデル | 公式 | OpenRouter | Crazyrouter | 削減率 |
|:--|:--|:--|:--|:--|
| GPT-5.2 | $3/$12 | ~$3.3/$13.2 | ~$1.65/$6.6 | 45% |
| Claude Opus 4.6 | $15/$75 | ~$16.5/$82.5 | ~$8.25/$41.25 | 45% |
| Claude Sonnet 4.6 | $3/$15 | ~$3.3/$16.5 | ~$1.65/$8.25 | 45% |
| Gemini 3 Pro | $1.25/$10 | ~$1.38/$11 | ~$0.69/$5.5 | 45% |
| DeepSeek V3.2 | $0.27/$1.1 | ~$0.3/$1.21 | ~$0.15/$0.61 | 45% |

*入力/出力 per 1Mトークン*

### コード例：2行変更で切替

```python
from openai import OpenAI

# 変更前（OpenRouter）
# client = OpenAI(base_url="https://openrouter.ai/api/v1", api_key="sk-or-xxx")

# 変更後（Crazyrouter）
client = OpenAI(
    base_url="https://crazyrouter.com/v1",
    api_key="your-crazyrouter-key"
)

response = client.chat.completions.create(
    model="gpt-5-mini",
    messages=[{"role": "user", "content": "2+2は？"}]
)
print(response.choices[0].message.content)
# 出力: 四
```

実際のレスポンス（2026年3月テスト済み）：

```json
{
  "model": "gpt-5-mini",
  "choices": [{"message": {"content": "Four"}, "finish_reason": "stop"}],
  "usage": {"prompt_tokens": 19, "completion_tokens": 1, "total_tokens": 20}
}
```

Anthropic SDKとGeminiフォーマットにもネイティブ対応。

✅ 最安（公式の約55%）、627+モデル、3フォーマット互換、日本リージョンあり
❌ セルフホスト不可、コミュニティ小さい

## 2. Portkey — エンタープライズ向け

https://portkey.ai

- 1,600+ LLM（BYOK）、ガードレール、SOC 2、RBAC
- 分散トレーシング、コストダッシュボード、プロンプトA/Bテスト

✅ 最も包括的なガバナンス ❌ BYOK（トークン節約なし）、複雑

## 3. LiteLLM — OSSセルフホスト

https://github.com/BerriAI/litellm

```bash
pip install litellm && litellm --config config.yaml
```

✅ MIT、データ外部流出なし ❌ インフラ管理必要、LLMのみ

## 4. Helicone — オブザーバビリティ特化

https://helicone.ai

AI APIのDatadog。1行で導入、10万リクエスト/月無料。

✅ 最高の可視化 ❌ モデルアグリゲーターではない

## 5. Unify AI — ベンチマーク駆動ルーティング

https://unify.ai

ベンチマーク・コスト・レイテンシで最適モデルに自動ルーティング。

✅ インテリジェントルーティング ❌ モデル少ない（80+）

## 6. Kong AI Gateway

https://konghq.com

Kong API GatewayのAI拡張。Kong既存ユーザー向け。

## 7. Cloudflare AI Gateway

https://developers.cloudflare.com/ai-gateway/

無料エッジキャッシュ。Cloudflareユーザー向け。

## 機能比較マトリクス

| 機能 | Crazyrouter | Portkey | LiteLLM | Helicone | Unify | Kong | CF |
|:--|:--|:--|:--|:--|:--|:--|:--|
| モデル数 | 627+ | 1,600+ | 100+ | BYOK | 80+ | - | BYOK |
| 料金割引 | ~45% | なし | なし | なし | - | なし | なし |
| 画像/動画/音楽 | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| セルフホスト | ❌ | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| スマートルーティング | ❌ | ✅ | 基本 | ❌ | ✅ | ❌ | ❌ |

## どれを選ぶ？

- **コスト重視** → Crazyrouter（~45%割引）
- **エンタープライズ** → Portkey（SOC 2、RBAC）
- **セルフホスト** → LiteLLM（OSS）
- **可視化** → Helicone（10万リクエスト無料）
- **マルチモーダル** → Crazyrouter（唯一のフル対応）
