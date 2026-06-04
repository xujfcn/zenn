---
title: "/v1/chat/completions・/v1/responses・/v1/messages の違い：AI API の正しいエンドポイント選び"
emoji: "🔌"
type: "tech"
topics: [api, openai, claude, ai]
published: true
---

# /v1/chat/completions vs /v1/responses vs /v1/messages: どのAI APIエンドポイントを使うべき？

AI APIゲートウェイにおける一般的なサポート問題は、APIキーでもなく、モデルでもなく、SDKでもありません。エンドポイントです。

ユーザーが存在するモデルを選択しても、リクエストを間違ったエンドポイントに送信してしまいます。結果は次のようになります：

- モデル利用不可；
- サポートされていないエンドポイント；
- 無効なリクエストボディ；
- ツール呼び出しが機能しない；
- ストリーミング形式の不一致；
- Claudeモデルはあるツールでは動作するが別のツールでは失敗。

このガイドでは、3つのエンドポイントファミリーの違いを説明します：

```text
/v1/chat/completions
/v1/responses
/v1/messages
```

簡易版：

| エンドポイント | ネイティブエコシステム | 最適な用途 | リクエストスタイル |
|---|---|---|---|
| `/v1/chat/completions` | OpenAI互換のレガシー/現在のチャット | ほとんどのアプリ、SDK、LiteLLM、Cursorスタイルツール、シンプルなチャット | `messages: [{role, content}]` |
| `/v1/responses` | より新しいOpenAI Responses API | ツール使用、マルチモーダル、推論アイテム、より新しいOpenAIスタイルのエージェント | `input`、`tools`、構造化応答アイテム |
| `/v1/messages` | Anthropic Claude API | Claude ネイティブSDKとClaudeスタイルのアプリ | `messages`とトップレベルの`system`、Anthropicスキーマ |

クライアントが「OpenAI互換」と言う場合は、以下から始めてください：

```text
https://crazyrouter.com/v1/chat/completions
```

またはベースURLを次のように設定：

```text
https://crazyrouter.com/v1
```

SDKが`/chat/completions`を追加します。

ツールが特にOpenAI Responses APIを必要とする場合は、`/v1/responses`を使用します。

ツールがAnthropicネイティブでClaudeのMessages APIを期待する場合は、`/v1/messages`を使用します。

## 最も一般的な間違い

最も一般的な間違いは、モデルファミリーとエンドポイントファミリーを混ぜることです。

例えば、ClaudeモデルはOpenAI互換ゲートウェイを通じて公開できますが、それはリクエストボディがAnthropicネイティブの`/v1/messages`スキーマを使用すべきということを自動的に意味しません。

同様に、Anthropicネイティブのリクエストを`/v1/messages`に送信するツールは、モデル名を変更するだけでは修正できません。エンドポイントとリクエストボディが一致する必要があります。

このように考えてください：

```text
モデル名 + エンドポイント + リクエストスキーマが一致する必要があります
```

この3つのうち1つが間違っていると、モデル自体が正常に動作していてもモデルが利用不可に見える場合があります。

## `/v1/chat/completions`とは

`/v1/chat/completions`は古典的なOpenAI互換チャットエンドポイントです。

典型的なリクエストは次のようになります：

```bash
curl https://crazyrouter.com/v1/chat/completions \
  -H "Authorization: Bearer $CRAZYROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-5.5",
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "Explain API endpoints in one paragraph."}
    ]
  }'
```

以下の場合に`/v1/chat/completions`を使用してください：

- アプリがOpenAI互換APIをサポートしていると述べている；
- 設定が`OPENAI_BASE_URL`または`base_url`を要求している；
- リクエストボディが`role`と`content`を持つ`messages`を使用している；
- 一般的なOpenAI SDK互換モードを使用している；
- Cursorスタイルクライアント、LiteLLMスタイルルーター、FastGPTスタイルアプリ、または多くのチャットUIなどのツールを設定している。

多くのユーザーにとって、これが最も安全なデフォルトエンドポイントです。

## `/v1/responses`とは

`/v1/responses`はより新しいOpenAI Responses APIエンドポイントです。

より一般的な応答オブジェクトの周りに設計されており、以下のようなものを表現できます：

- テキスト出力；
- ツール呼び出し；
- マルチモーダル入力；
- 推論アイテム；
- 構造化出力；
- エージェントのようなワークフロー。

簡略化されたリクエストは次のようになります：

```bash
curl https://crazyrouter.com/v1/responses \
  -H "Authorization: Bearer $CRAZYROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-5.5",
    "input": "Explain the difference between chat completions and responses."
  }'
```

以下の場合に`/v1/responses`を使用してください：

- ツールがOpenAI Responses APIを使用していると明示的に述べている；
- 設定が`chat.completions`ではなく`responses`を言及している；
- リクエストボディが`messages`ではなく`input`を使用している；
- モデル/プロバイダールートがResponses APIの動作をサポートしている；
- より新しいOpenAIスタイルのツールまたは推論出力形式が必要です。

ゲートウェイがその変換を明示的に文書化している場合を除き、Chat Completionsボディを`/v1/responses`に送信しないでください。

## `/v1/messages`とは

`/v1/messages`はAnthropicのMessages APIスタイルのエンドポイントです。

典型的なClaudeネイティブリクエストは次のようになります：

```bash
curl https://crazyrouter.com/v1/messages \
  -H "Authorization: Bearer $CRAZYROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-sonnet-4-6",
    "max_tokens": 1024,
    "system": "You are a helpful assistant.",
    "messages": [
      {"role": "user", "content": "Explain Claude Messages API."}
    ]
  }'
```

以下の場合に`/v1/messages`を使用してください：

- クライアントはAnthropicネイティブ；
- SDKはClaude Messages API形式を期待；
- リクエストが配列内のシステムメッセージではなくトップレベルの`system`を持つ；
- リクエストがAnthropic固有のコンテンツブロックまたはツールスキーマを必要とする；
- ドキュメントが明示的に`/v1/messages`を呼び出すように述べている。

すべてのClaudeモデルが`/v1/messages`で呼び出されるべきだと仮定しないでください。OpenAI互換ゲートウェイまたはSDKを使用している場合、Claudeモデルは代わりに`/v1/chat/completions`を通じて呼び出される場合があります。

## エンドポイント間違いがモデルを利用不可に見させる理由

モデルが失敗すると、ユーザーはモデルがダウンしていると仮定することがよくあります。しかし多くの場合、モデルは問題なく、リクエスト形式が間違っています。

一般的な不一致の例：

| 間違い | 発生すること | 修正方法 |
|---|---|---|
| `messages`チャットボディを`/v1/responses`に送信 | 無効なリクエストボディまたは無視されたフィールド | `/v1/chat/completions`を使用するか、ボディを`input`に変更 |
| Anthropicの`system` + `messages`ボディを`/v1/chat/completions`に送信 | スキーマの不一致 | `/v1/messages`を使用するか、OpenAIメッセージ形式に変換 |
| OpenAI SDKで`/v1/messages`を使用 | SDKが間違ったパスを追加するか、間違った応答を解析する場合がある | OpenAI互換ベースURLをチャットコンプリーションで使用 |
| ベースURLで`/v1`を欠落 | 404またはルート不明 | ベースURLを`https://crazyrouter.com/v1`に設定 |
| `/chat/completions`をベースURLに追加してSDKが再び追加 | `/chat/completions/chat/completions`のような二重パス | ベースURLは通常`/v1`で終わるべき |
| APIエンドポイントにUTMパラメータを追加 | 認証/ルーティングエラー | UTMはWebサイトリンクにのみ属する |

## ベースURLとフルエンドポイント

多くのツールはフルエンドポイントではなくベースURLを要求します。

正しいベースURL：

```text
https://crazyrouter.com/v1
```

次にSDKが呼び出す：

```text
https://crazyrouter.com/v1/chat/completions
```

ほとんどのSDKにとって間違ったベースURL：

```text
https://crazyrouter.com/v1/chat/completions
```

なぜですか？ SDKが再度`/chat/completions`を追加する場合があるからです。

ルール：

- フィールドが`base_url`と呼ばれている場合は、`/v1`を使用します。
- フィールドが`endpoint`と呼ばれ、フルURLを要求する場合は、ツールが必要とするフルパスを使用します。

## どのエンドポイントを選ぶべき？

この決定木を使用してください：

1. ツールが「OpenAI互換API」と述べていますか？  
   `/v1/chat/completions`またはベースURL`https://crazyrouter.com/v1`を使用します。

2. ツールが特に「Responses API」と述べていますか？  
   `/v1/responses`を使用します。

3. ツールはAnthropic SDKまたはClaudeのMessages APIを使用しますか？  
   `/v1/messages`を使用します。

4. 確実でありませんか？  
   ほとんどのサードパーティクライアントがそれをサポートしているので、`/v1/chat/completions`から始めてください。

## 推奨されるCrazyrouterの設定

ほとんどのOpenAI互換ツール向け：

```text
API Key: あなたのCrazyrouter APIキー
Base URL: https://crazyrouter.com/v1
Model: Cazyrouterモデルリストからモデルを選択
```

グローバルルートが不安定な地域のユーザー向け：

```text
Base URL: https://cn.crazyrouter.com/v1
```

ユーザー向けリンクはUTMトラッキングを使用できます：

```text
https://crazyrouter.com/models?utm_source=blog&utm_medium=article&utm_campaign=api_endpoints
```

APIエンドポイントはそうあるべきではありません：

```text
https://api-endpoint.example.invalid/v1?utm_source=blog
```

## クイックデバッグチェックリスト

「モデル利用不可」と報告する前に、これら5つのことを確認してください：

1. モデル名はモデルリストに表示される通りに正確に綴られていますか？
2. エンドポイントファミリーは正しいですか：チャットコンプリーション、レスポンス、またはメッセージ？
3. リクエストボディはそのエンドポイントのスキーマと一致していますか？
4. ベースURLはちょうど`/v1`ですか、それを欠落していたり、エンドポイントパスを複製していたりしていませんか？
5. 正しいSDKモードを使用していますか：OpenAI互換またはAnthropicネイティブ？

## 最終的な推奨

一般的なアプリ統合を構築している場合は、`/v1/chat/completions`から始めてください。これは最も広い互換性パスです。

クライアントまたはワークフローがResponses APIを明示的に必要とする場合は、`/v1/responses`を使用します。

Anthropicネイティブツールを使用している場合は、`/v1/messages`を使用します。

ほとんどのエンドポイント問題は、このルールを思い出すと消えます：

```text
OpenAI互換クライアント → /v1/chat/completions
OpenAI Responsesクライアント → /v1/responses
Anthropicネイティブクライアント → /v1/messages
```

モデルが存在するが利用不可に見える場合は、モデル名だけを変更しないでください。最初にエンドポイントとリクエストスキーマを確認してください。

---

Canonical: https://crazyrouter.com/blog/chat-completions-vs-responses-vs-messages-api-ja?utm_source=zenn&utm_medium=article&utm_campaign=api_endpoints
