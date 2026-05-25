---
title: "Seedance 2.0料金解説：100万Tokenあたり46元を1秒あたりコストに換算"
emoji: "🎬"
type: "tech"
topics: [ai, video, api, pricing]
published: true
---

# Seedance 2.0料金：100万Token課金を1秒あたりコストに換算する

Seedance 2.0は秒単価ではなくToken単位で価格が示されます。しかし動画プロダクトの予算設計では、1秒または1本あたりのコストに直す必要があります。

![Seedance 2.0 pricing cover](https://raw.githubusercontent.com/xujfcn/images/main/blog/covers/seedance-2-0-pricing-token-to-second-cost-2026.webp)

## 公式Token価格の意味

公開されている参考値では、純生成は100万Tokenあたり46元、動画編集は100万Tokenあたり28元です。15秒動画は約308,880 Tokenなので、1秒あたり約20,592 Token。純生成は0.947元/秒、編集モードは0.577元/秒です。

## 1秒あたりコストの計算式

```text
tokens_per_second = 308,880 / 15 = 20,592
pure_generation_per_second = 20,592 / 1,000,000 × 46 = 0.947 CNY
editing_per_second = 20,592 / 1,000,000 × 28 = 0.577 CNY
```

## 動画尺別コスト表

| Duration | Estimated tokens | Pure generation | Video editing |
|---:|---:|---:|---:|
| 1 sec | 20,592 | 0.95 CNY | 0.58 CNY |
| 5 sec | 102,960 | 4.74 CNY | 2.88 CNY |
| 10 sec | 205,920 | 9.47 CNY | 5.77 CNY |
| 15 sec | 308,880 | 14.21 CNY | 8.65 CNY |
| 30 sec | 617,760 | 28.42 CNY | 17.30 CNY |
| 60 sec | 1,235,520 | 56.83 CNY | 34.59 CNY |

![Seedance 2.0 token to second pricing calculator](https://raw.githubusercontent.com/xujfcn/images/main/blog/posts/seedance-2-0-pricing-token-to-second-cost-2026-01.webp)

## 純生成と動画編集の違い

Pure generation is best for creating a new clip from text, image, or audio references. Editing mode is cheaper because it reuses an existing video reference and changes part of the workflow. If your product supports iteration, editing mode can reduce model cost by about 39%.

## プロダクト価格設計の考え方

For SaaS pricing, do not charge only the raw model cost. Add retry rate, failed generations, storage, CDN bandwidth, queue priority, payment fees, support cost, and margin. A practical product price is often 1.5x to 3x the raw model cost depending on success rate and customer value.

![Seedance 2.0 video generation pricing workflow](https://raw.githubusercontent.com/xujfcn/images/main/blog/posts/seedance-2-0-pricing-token-to-second-cost-2026-02.webp)

## API提供状況の注意点

Published pricing is useful for budgeting, but production access can depend on account region, provider availability, enterprise approval, resolution, queue tier, and current API rollout. Always verify the latest provider console before committing customer pricing.

## FAQ

### Seedance 2.0は1秒いくらですか？純生成は約0.95元/秒、編集は約0.58元/秒です。

Seedance 2.0は1秒いくらですか？純生成は約0.95元/秒、編集は約0.58元/秒です。

### 1秒あたり何Tokenですか？約20,592 Tokenです。

1秒あたり何Tokenですか？約20,592 Tokenです。

### 15秒動画はいくらですか？純生成は約14.21元、編集は約8.65元です。

15秒動画はいくらですか？純生成は約14.21元、編集は約8.65元です。

### 最終価格はモデル原価だけで決められますか？いいえ。リトライ、保存、CDN、決済手数料、利益を含める必要があります。

最終価格はモデル原価だけで決められますか？いいえ。リトライ、保存、CDN、決済手数料、利益を含める必要があります。

### APIは誰でも使えますか？最新の提供状況はVolcengine側で確認してください。

APIは誰でも使えますか？最新の提供状況はVolcengine側で確認してください。

## Bottom line

Seedance 2.0 pure generation is roughly **0.95 CNY per second**, while video editing mode is roughly **0.58 CNY per second**, based on the public 15-second token reference. For product pricing, use cost per successful usable video, not only token cost.

If you want one API layer for video, image, chat, and other AI models, try [Crazyrouter](https://crazyrouter.com?utm_source=crazyrouter_blog&utm_medium=article&utm_campaign=seedance_pricing_2026).

---

Original: https://crazyrouter.com/blog/seedance-2-0-pricing-token-to-second-cost-2026-ja
