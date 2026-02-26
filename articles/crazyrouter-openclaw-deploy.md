---
title: "1コマンドで自分専用AIアシスタントをデプロイ：Crazyrouter + OpenClaw セルフホスティングガイド"
emoji: "🤖"
type: "tech"
topics: ["AI", "ChatGPT", "Telegram", "selfhosted", "OpenAI"]
published: true
---

## はじめに

ChatGPT Plus に毎月 $20 払うのに疲れていませんか？複数の AI プラットフォームを行き来するのが面倒ではありませんか？

この記事では、**たった1つのコマンド**で、300以上の AI モデルに対応するプライベート AI ゲートウェイをサーバーにデプロイし、Telegram Bot として使う方法を紹介します。

## なぜセルフホスティング AI ゲートウェイが必要なのか？

ChatGPT、Claude、Gemini を使ったことがある人なら、この悩みがわかるはずです：

- コーディングには Claude（コード能力が高い）
- 日常会話には GPT（安い）
- 長文分析には Gemini（コンテキストウィンドウが大きい）
- 推論タスクには DeepSeek R1（コスパが良い）

結果、4つの API Key、4つの請求書、4つの異なる API フォーマットを管理することに…

**Crazyrouter + OpenClaw の組み合わせがこの問題を解決します：**

- **Crazyrouter**：1つの API Key で 300以上のモデル（GPT、Claude、Gemini、DeepSeek、Kimi など）にアクセス。OpenAI 互換フォーマット、公式より安い
- **OpenClaw**：オープンソース AI ゲートウェイ。AI モデルをプライベートアシスタントに変え、Telegram、Discord、Slack、飛書、DingTalk などの IM プラットフォームに対応

この2つを組み合わせれば = **自分だけの AI インフラ**が手に入ります。

## デプロイ後のイメージ

デプロイが完了すると、以下が手に入ります：

1. サーバー上で動く AI ゲートウェイ（ポート 18789）
2. いつでもどこでも AI と会話できる Telegram Bot
3. 15以上の人気モデルがすぐに使える状態

![Telegram Bot チャット画面](https://raw.githubusercontent.com/xujfcn/crazyrouter-openclaw/main/screenshots/telegram-chat.jpg)
*▲ デプロイ後、Telegram で /start を送信してペアリングすれば、すぐに AI アシスタントと会話開始*

## 準備するもの

必要なのは2つだけ：

| 準備項目 | 取得方法 |
|---------|---------|
| Crazyrouter API Key | [crazyrouter.com](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=dev_community) で登録して取得 |
| Telegram Bot Token（任意） | Telegram で @BotFather から作成 |

サーバー要件：
- Linux（Ubuntu/Debian/CentOS）または macOS
- メモリ 4GB 以上
- インターネット接続

> 💡 サーバーがない？最安のクラウドサーバー（2コア4GB、月数百円〜）で十分です。

## ワンコマンドインストール

ターミナルを開いて、この1行を貼り付けるだけ：

```bash
curl -fsSL https://raw.githubusercontent.com/xujfcn/crazyrouter-openclaw/main/install.sh | bash
```

または、ダウンロードしてから実行：

```bash
wget https://raw.githubusercontent.com/xujfcn/crazyrouter-openclaw/main/install.sh
bash install.sh
```

対話入力をスキップしたい場合は環境変数で：

```bash
CRAZYROUTER_API_KEY=sk-あなたのkey curl -fsSL https://raw.githubusercontent.com/xujfcn/crazyrouter-openclaw/main/install.sh | bash
```

## インストールプロセス

スクリプトは10ステップを自動実行。プログレスバーと ASCII Art バナー付きで、体験は非常にスムーズです：

![インストール前半](https://raw.githubusercontent.com/xujfcn/crazyrouter-openclaw/main/screenshots/install-part1.png)
*▲ ステップ 1-7：システム環境検出、Node.js と OpenClaw のインストール、設定ファイル生成、安定性パッチのダウンロード*

![インストール後半](https://raw.githubusercontent.com/xujfcn/crazyrouter-openclaw/main/screenshots/install-part2.png)
*▲ ステップ 7-10：IM プラグインのインストール、systemd サービス作成、ゲートウェイ起動、「インストール完了！」*

インストール完了後、スクリプトは以下の情報を表示します：
- WebUI アクセス URL
- デフォルトモデル（claude-sonnet-4-6）
- 設定ファイルのパス
- Gateway Token（大切に保管してください！）
- 管理コマンド一覧

全体で約 2〜5 分（ネットワーク速度による）。

## プリセットモデル

インストール後、以下のモデルがすぐに使えます：

| ベンダー | モデル | 特徴 |
|---------|-------|------|
| Anthropic | Claude Opus 4.6 | 最強の総合能力 |
| Anthropic | Claude Sonnet 4.6 | コスパ最強 |
| OpenAI | GPT-5.2 | 最新フラッグシップ |
| OpenAI | GPT-5.3 Codex | コーディング特化 |
| OpenAI | GPT-5 Mini | 軽量・高速 |
| OpenAI | GPT-4.1 / 4.1 Mini | 安定の定番 |
| OpenAI | GPT-4o Mini | 超低コスト |
| Google | Gemini 3.1 Pro | 長コンテキスト |
| Google | Gemini 3 Flash | 超高速レスポンス |
| DeepSeek | R1 | 推論能力が高い |
| DeepSeek | V3.2 | 中国語最強 |
| その他 | Kimi K2.5, GLM-5, Grok 4.1, MiniMax M2.1 | それぞれの強み |

すべてのモデルは Crazyrouter 経由で統一的に呼び出し。**1つの API Key ですべて完結**。

## Telegram Bot の接続

インストールスクリプトの最後に、Telegram Bot のセットアップが自動的にガイドされます：

**ステップ 1：Bot の作成**

Telegram で @BotFather を検索し、`/newbot` を送信。指示に従って Bot に名前を付け、Token を取得：

![BotFather で Bot を作成](https://raw.githubusercontent.com/xujfcn/crazyrouter-openclaw/main/screenshots/botfather.jpg)
*▲ BotFather で Bot を作成し、Token を取得*

**ステップ 2：Token を貼り付けてペアリング**

Token をインストールスクリプトに貼り付けると、自動的に設定書き込み → ゲートウェイ再起動 → Telegram チャンネル読み込み → ペアリングガイドが行われます：

![Telegram Bot セットアップとペアリング](https://raw.githubusercontent.com/xujfcn/crazyrouter-openclaw/main/screenshots/telegram-setup.png)
*▲ インストールスクリプトの対話式ガイド：Token 貼り付け → 設定書き込み → 再起動 → 自動ペアリング*

**ステップ 3：会話開始**

ペアリング完了後、Telegram で Bot に直接話しかけるだけ。裏側では設定した AI モデルが動いています：

![Bot との初回会話](https://raw.githubusercontent.com/xujfcn/crazyrouter-openclaw/main/screenshots/telegram-chat.jpg)
*▲ /start でペアリング後、AI アシスタントが自動的に挨拶。すぐに使える*

## 対応 IM プラットフォーム

Telegram 以外にも、OpenClaw は以下に対応：

- **Discord** — ゲームコミュニティ AI アシスタント
- **Slack** — チーム協業 AI
- **飛書（Feishu/Lark）** — 企業内部 AI
- **DingTalk** — オフィス AI アシスタント
- **企業微信（WeCom）** — WeChat エコシステム AI
- **QQ Bot** — ソーシャル AI

インストールスクリプトは DingTalk、WeCom、QQ Bot プラグインを自動インストール済み。

## Docker デプロイ（オプション）

コンテナ化デプロイがお好みなら、Dockerfile も用意されています：

```bash
# リポジトリをクローン
git clone https://github.com/xujfcn/crazyrouter-openclaw.git
cd crazyrouter-openclaw

# イメージをビルド
docker build -t openclaw-bot .

# 実行（設定ディレクトリをマウント）
docker run -d \
  --name openclaw-bot \
  -p 18789:18789 \
  -v ~/.openclaw:/root/.openclaw \
  openclaw-bot
```

Dockerfile は `node:22-bookworm-slim` ベース。イメージサイズが小さく、起動が速い。

## インストール後の管理

### ステータスとログの確認

```bash
# Linux (systemd)
systemctl --user status openclaw    # ステータス確認
journalctl --user -u openclaw -f   # リアルタイムログ

# macOS (launchd)
tail -f ~/.openclaw/openclaw.log   # ログ確認
```

### 再起動と停止

```bash
# Linux
systemctl --user restart openclaw  # 再起動
systemctl --user stop openclaw     # 停止

# macOS
launchctl stop com.crazyrouter.openclaw && launchctl start com.crazyrouter.openclaw  # 再起動
launchctl stop com.crazyrouter.openclaw  # 停止
```

## 安定性の保証

スクリプトには `crash-guard.cjs` パッチが内蔵されており、Node.js 22 + undici 7.x の TLS クラッシュ問題を解決：

- **TLS Session 保護**：`setSession` のヌルポインタクラッシュを防止
- **undici エラー抑制**：一時的なネットワークエラーを自動キャッチ、プロセスダウンを防止
- **期限切れロックファイルのクリーンアップ**：`.lock` ファイルを定期的にクリーンアップ
- **Memory ディレクトリのバックアップ**：5分ごとに AI のメモリファイルを自動バックアップ

つまり、AI ゲートウェイは **24時間365日安定稼働**、人手の介入は不要です。

## プロジェクト構成

```
crazyrouter-openclaw/
├── install.sh        # ワンクリックインストールスクリプト
├── deploy.sh         # Docker デプロイスクリプト
├── Dockerfile        # コンテナイメージ定義
├── crash-guard.cjs   # TLS クラッシュ防護パッチ
└── README.md         # プロジェクトドキュメント
```

GitHub リポジトリ：[github.com/xujfcn/crazyrouter-openclaw](https://github.com/xujfcn/crazyrouter-openclaw)

## よくある質問

**Q: Crazyrouter と OpenRouter の違いは？**

Crazyrouter は価格がより安く、中国のモデル（DeepSeek、Kimi、GLM など）にも対応。API は OpenAI フォーマットと完全互換。登録してすぐ使え、海外クレジットカードは不要です。

**Q: インストールに失敗したら？**

Node.js のバージョン（22以上が必要）とネットワーク接続を確認してください。手動インストールも可能：
```bash
npm install -g openclaw@latest
```

**Q: Raspberry Pi で動きますか？**

はい！スクリプトは arm64 アーキテクチャに対応。Raspberry Pi 4/5 で問題なく動作します。

**Q: 料金は？**

Crazyrouter はトークン課金で、公式 API より安い。詳細は [crazyrouter.com](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=dev_community) をご覧ください。

## まとめ

1つのコマンドで、以下が手に入ります：

✅ 300以上の AI モデルに対応するプライベートゲートウェイ
✅ いつでも使える Telegram AI アシスタント
✅ 自動起動 + クラッシュ自動復旧の安定サービス
✅ 統一 API インターフェース、マルチプラットフォーム管理からの解放

```bash
curl -fsSL https://raw.githubusercontent.com/xujfcn/crazyrouter-openclaw/main/install.sh | bash
```

ぜひ試してみてください。2分後には自分だけの AI が手に入ります。

---

**関連リンク：**
- Crazyrouter 公式サイト：[crazyrouter.com](https://crazyrouter.com?utm_source=zenn&utm_medium=article&utm_campaign=dev_community)
- OpenClaw オープンソースプロジェクト：[github.com/openclaw/openclaw](https://github.com/openclaw/openclaw)
- デプロイリポジトリ：[github.com/xujfcn/crazyrouter-openclaw](https://github.com/xujfcn/crazyrouter-openclaw)
- Telegram チャンネル：[t.me/crazyrouter](https://t.me/crazyrouter)
