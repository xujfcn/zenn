---
title: "Seedance 2.0の実課金：固定の秒単価ではなくoutput tokensで決まる理由"
emoji: "🎬"
type: "tech"
topics: [ai, video, api, pricing]
published: true
---

# Seedance 2.0の実課金：秒単価固定ではなく、完了後のoutput tokensで決まる

顧客からよく聞かれるのは「Seedance 2.0は1秒いくらか」です。正確には、Seedance 2.0には全タスク共通の固定秒単価はありません。タスク完了後に上流が返す実際のoutput tokensをもとに課金されます。

![Seedance 2.0 billing guide](https://raw.githubusercontent.com/xujfcn/images/main/blog/covers/seedance-2-0-pricing-token-to-second-cost-2026.webp)

## 結論

The correct billing flow is:

```text
submit task -> task completes -> upstream returns usage tokens -> calculate USD by tokens -> convert to Crazyrouter quota
```

It is **not**:

```text
submit task -> charge directly by requested duration seconds
```

So we should not say:

```text
1 second = fixed N tokens
```

Instead, after the task finishes, we can calculate:

```text
observed tokens/sec = billedTokens / requestedDurationSeconds
observed USD/sec = actualPriceUSD / requestedDurationSeconds
```

This is an observed value for that task, not a universal fixed rate.

## 現在の公開対応範囲

Current public capability boundary for `doubao-seedance-2-0` and `doubao-seedance-2-0-fast`:

- `480p` supported
- `720p` supported
- `1080p` is not currently in the public supported range

The measured examples below use `720p` and `4s`.

## 課金ルール

Seedance 2.0 billing mainly depends on whether the request contains video input.

| Model | Condition | Billing key | Unit price |
|---|---|---|---|
| `doubao-seedance-2-0` | no video input | `doubao-seedance-2-0:video0` | `46 / 7 USD / 1M tokens` |
| `doubao-seedance-2-0` | with video input | `doubao-seedance-2-0:video1` | `28 / 7 USD / 1M tokens` |
| `doubao-seedance-2-0-fast` | no video input | `doubao-seedance-2-0-fast:video0` | `37 / 7 USD / 1M tokens` |
| `doubao-seedance-2-0-fast` | with video input | `doubao-seedance-2-0-fast:video1` | `22 / 7 USD / 1M tokens` |

`video0` means there is no video reference input. `video1` means the request includes video reference input, such as `reference_video`.

## 最終課金式

After a successful task, the system first reads `TotalTokens`. If unavailable, it falls back to `CompletionTokens`.

```text
actualPriceUSD =
  unitPriceUSDPer1MTokens
  * (billedTokens / 1_000_000)
  * quantityMultiplier
  * groupRatio
  * discount
```

Without extra multipliers or discounts:

```text
actualPriceUSD = unitPriceUSDPer1MTokens * billedTokens / 1_000_000
```

Crazyrouter quota conversion:

```text
actualQuota = int(actualPriceUSD * QuotaPerUnit)
QuotaPerUnit = 500000
```

![Seedance 2.0 billing calculator](https://raw.githubusercontent.com/xujfcn/images/main/blog/posts/seedance-2-0-pricing-token-to-second-cost-2026-01.webp)

## 実測ケース1：text-to-video

Request profile:

- Model: `doubao-seedance-2-0`
- Type: text-to-video
- Input: text only, no image or video reference
- Resolution: `720p`
- Duration: `4s`
- Task ID: `cgt-20260617212928-5s755`
- `completion_tokens = 87300`
- `total_tokens = 87300`

Because there is no video reference input, the billing key is:

```text
billing_key = doubao-seedance-2-0:video0
unitPrice = 46 / 7 = 6.571428 USD / 1M tokens
```

Calculation:

```text
tokensPerSecond = 87300 / 4 = 21825 tokens/s
actualPrice = 87300 / 1000000 * 6.571428 = 0.5736 USD
pricePerSecond = 0.5736 / 4 = 0.1434 USD/s
```

Result: this simple 4-second 720p text-to-video task was observed at about **$0.14/sec**.

## 実測ケース2：reference video generation

Request profile:

- Model: `doubao-seedance-2-0`
- Type: reference-video generation
- Input: text + previous generated video as `reference_video`
- Resolution: `720p`
- Duration: `4s`
- Task ID: `cgt-20260617214300-rsnsx`
- `completion_tokens = 173700`
- `total_tokens = 173700`

Because the request contains video reference input, the billing key is:

```text
billing_key = doubao-seedance-2-0:video1
unitPrice = 28 / 7 = 4.000000 USD / 1M tokens
```

Calculation:

```text
tokensPerSecond = 173700 / 4 = 43425 tokens/s
actualPrice = 173700 / 1000000 * 4.000000 = 0.6948 USD
pricePerSecond = 0.6948 / 4 = 0.1737 USD/s
```

Result: this 4-second 720p reference-video task was observed at about **$0.17/sec**.

## 2つの実測比較

| Scenario | Duration | Returned tokens | Tokens/sec | Unit price | Total price | Observed $/sec |
|---|---:|---:|---:|---:|---:|---:|
| Simple text-to-video, no video input | 4s | 87,300 | 21,825 | $6.571428 / 1M tokens | $0.5736 | $0.1434/s |
| Reference video generation, with video input | 4s | 173,700 | 43,425 | $4.000000 / 1M tokens | $0.6948 | $0.1737/s |

This comparison shows why a single per-second price is misleading: video-input requests have a lower unit token price, but may return more tokens, so the final observed per-second cost can still be higher.

![Seedance 2.0 billing workflow](https://raw.githubusercontent.com/xujfcn/images/main/blog/posts/seedance-2-0-pricing-token-to-second-cost-2026-02.webp)

## 顧客向け説明

Seedance 2.0には固定の秒単価はありません。タスク完了後に返される実際のoutput tokensで課金されます。完了済みタスクから平均コスト/秒を逆算することはできますが、全タスク共通のtokens/sec換算はありません。

If customers ask why there is no fixed second price, the reason is simple: duration is a task parameter, but the final billing unit is the actual output tokens returned after generation.

## 社内向け提案

To give customers more stable estimates, collect more real task samples and group them by:

- text-to-video without reference input
- image-to-video
- reference-video generation
- with or without audio
- duration
- prompt complexity
- resolution

Then calculate `p50`, `p75`, and `p90` tokens/sec. This is much more reliable than guessing a universal tokens/sec number.

## Bottom line

顧客向けには、Seedance 2.0は実際のoutput tokensで課金される、と説明するのが安全です。今回の720p/4秒テストでは、単純なtext-to-videoが約$0.14/s、reference video生成が約$0.17/sでした。

For one API layer across video, image, chat, and other AI models, try [Crazyrouter](https://crazyrouter.com?utm_source=crazyrouter_blog&utm_medium=article&utm_campaign=seedance_actual_billing_20260618).