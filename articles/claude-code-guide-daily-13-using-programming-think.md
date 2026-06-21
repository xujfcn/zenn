---
title: "Claude Code と Crazyrouterで実践する「プログラミング思考」"
emoji: "🤖"
type: "tech"
topics: ["claudecode", "ai", "api", "crazyrouter"]
published: true
---

# Claude Code と Crazyrouterで実践する「プログラミング思考」

プログラミング思考は、コードを書く人だけのものではありません。大きく曖昧な問題を、小さく実行可能で検証しやすい手順に分ける考え方です。Claude Code と Crazyrouter を組み合わせる場合も、この考え方があると「AIに丸投げ」ではなく、再利用できる業務フローを作れます。

詳しい導入ガイドはこちらです: [Claude Code Guide](https://github.com/xujfcn/claude-code-guide?utm_source=zenn&utm_medium=article&utm_campaign=claude_code_guide_daily)

## まずエンドポイントを分ける

Claude Code / Anthropic ネイティブクライアントではルートを使います。

`https://cn.crazyrouter.com`

OpenAI互換SDK、HTTPリクエスト、アプリ側の `base_url` では `/v1` 付きです。

`https://cn.crazyrouter.com/v1`

ここを混ぜると、クライアントが自動で `/v1` を付けて `/v1/v1/...` になることがあります。APIエンドポイントにはUTMなどの計測パラメータも付けません。

```bash
export ANTHROPIC_BASE_URL="https://cn.crazyrouter.com"
export ANTHROPIC_AUTH_TOKEN="your_crazyrouter_api_token"

# OpenAI互換SDKでは base_url="https://cn.crazyrouter.com/v1" を使う
```

## プログラミング思考の4要素

実務では次の4つに分けると扱いやすくなります。

1. **分解**: 大きな問題を小さな作業に分ける
2. **パターン認識**: 繰り返し発生する構造を見つける
3. **抽象化**: 個別作業からテンプレートやルールを作る
4. **手順設計**: 実行順序、入力、出力、確認方法を決める

たとえば議事録作成なら、「録音・メモ取得」「発言整理」「決定事項抽出」「TODO抽出」「担当者確認」「配布」のように分解できます。構造が見えれば、Claude Code に「決定事項とTODOをMarkdownで整理し、担当者不明はTBDにする」と依頼できます。

## 複雑な問題を分解する基準

良い分解には3つの条件があります。

- **独立性**: できるだけ各タスクを単独で進められる
- **実行可能性**: 何をすればよいか具体的である
- **検証可能性**: 完了したか判断できる

「顧客満足度を上げる」は曖昧です。一方で「直近1か月の問い合わせをカテゴリ別に分類し、上位5件の原因を要約する」は実行も検証もできます。

## 解決策を設計する

AIを使う場合でも、まず入力、処理、出力、レビューを決めます。週次レポートなら次のような流れです。

1. 営業データ、CRMデータ、メモを集める
2. 表計算やスクリプトで形式をそろえる
3. Claude Code に傾向、異常値、未確認点を抽出させる
4. 固定テンプレートでレポート草案を作る
5. 人が数値、表現、結論を確認する
6. 保存・共有し、次回のテンプレートを改善する

ポイントは、人間の判断をなくすことではありません。繰り返しの整形や分類を減らし、人間が確認、判断、改善に集中できる形にすることです。

## Crazyrouterを使う意味

チームでClaude Code、Anthropic系クライアント、OpenAI互換SDK、社内アプリを併用する場合、接続先が散らばると設定ミスが増えます。Crazyrouterを共通の入口にし、Claude Code は `ANTHROPIC_BASE_URL=https://cn.crazyrouter.com`、OpenAI互換SDKは `base_url=https://cn.crazyrouter.com/v1` と決めておくと、ワークフロー設計に集中しやすくなります。

まずは議事録、週報、問い合わせ分類、リリースノートなど小さな業務から始めるのがおすすめです。分解し、Claude Codeを組み込み、結果を検証し、少しずつ改善していきましょう。
