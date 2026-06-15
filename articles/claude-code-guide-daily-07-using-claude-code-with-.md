---
title: "Claude CodeとCrazyrouterで文章作成・編集ワークフローを作る"
emoji: "🤖"
type: "tech"
topics: ["claudecode", "ai", "api", "crazyrouter"]
published: true
---

# Claude CodeとCrazyrouterで文章作成・編集ワークフローを作る

Claude Codeはコード支援だけでなく、文章作成、要約、議事録整理、広告文案、SNS投稿案、ドキュメント整形にも使えます。チームで利用する場合は、モデル呼び出しの入口をCrazyrouterに統一しておくと、設定と運用を管理しやすくなります。

ガイド全体はこちらです: [claude-code-guide](https://github.com/xujfcn/claude-code-guide?utm_source=zenn&utm_medium=article&utm_campaign=claude_code_guide_daily)

## エンドポイントを間違えない

まず重要なのはBase URLです。

- Claude Code / Anthropicネイティブクライアント: `https://cn.crazyrouter.com`
- OpenAI互換SDK、HTTPクライアント、Webアプリ: `https://cn.crazyrouter.com/v1`

`/v1`を二重に付けると、`/v1/v1/...` のようなパスになり、原因が分かりにくいエラーになります。

```bash
export ANTHROPIC_BASE_URL="https://cn.crazyrouter.com"
export ANTHROPIC_AUTH_TOKEN="your_crazyrouter_api_token"
mkdir -p .ai/prompts .ai/outputs
```

## 文章生成はテンプレート化する

「商品説明を書いて」だけでは、出力の品質が安定しません。次の情報を渡すと実務で使いやすくなります。

- 商品名、カテゴリ
- 主な機能
- 対象ユーザー
- 強調したい価値
- 文体
- 出力形式
- 使ってはいけない表現

例:

> ノイズキャンセリング対応のスマートイヤホンの商品説明を作成してください。対象は若いオフィスワーカー。特徴は長時間バッテリー、快適な装着感、高音質。出力は見出し、短い紹介文、箇条書き、CTAにしてください。未確認の性能主張は避けてください。

食品、健康、金融、医療に近い文案では、断定的な表現に注意します。「認証済み」「安全」「効果がある」などは、根拠がある場合だけ使うべきです。

## 広告文案は複数案で比較する

Claude Codeには、1案だけでなく角度別に複数案を出させるのが実用的です。たとえばスマート水筒なら、忙しさ、データ管理、温度表示、ギフト用途などの切り口があります。

プロンプト例:

> スマート水筒の広告文案を5案作成してください。機能は飲水リマインド、飲水量記録、温度表示。対象は会社員。各案に見出し、40字程度の本文、CTA、推奨ビジュアルを含めてください。健康効果の保証は書かないでください。

キャンペーン文案では、期間、対象商品、除外条件、併用可否を必ず指定します。見栄えのよい文章より、正確な文章のほうが重要です。

## 編集・要約・形式変換

既存文章の改善も強力な用途です。

- もっと正式にする
- 一般ユーザー向けに言い換える
- 短くする
- 箇条書きにする
- 表にする
- 粗いメモを議事録にする
- Markdownに変換する

「きれいにして」ではなく、「初心者向けに、180字以内で、技術条件は保持し、手順形式で」と指定すると安定します。

議事録なら、次のように依頼できます。

> 以下のメモを議事録にしてください。会議名、日時、参加者、決定事項、リスク、担当者付きTODO、次回予定を含めてください。不明な情報は推測せず、不明と書いてください。

「推測しない」と明示することで、もっともらしい誤情報を減らせます。

## リポジトリにプロンプトを保存する

繰り返し使う作業は `.ai/prompts/` に保存します。

- `product-description.md`
- `meeting-minutes.md`
- `social-rewrite.md`
- `campaign-brainstorm.md`

各テンプレートには、目的、必須入力、出力形式、トーン、禁止表現、レビュー項目を書いておくと、チームで再利用しやすくなります。

## 公開前チェック

公開前には、日付、価格、機能、根拠のない主張、ブランド表記、リンクを確認します。AIの出力は下書きとして扱い、最終判断は人が行うべきです。

Crazyrouter利用時は、最後にBase URLも確認します。Claude Codeは `https://cn.crazyrouter.com`、OpenAI互換用途は `https://cn.crazyrouter.com/v1` です。
