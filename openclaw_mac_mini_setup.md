# OpenClaw 導入・構築ガイド (Mac Mini 向け)

このドキュメントは、Mac Mini 環境に OpenClaw（Gemini 連携エージェント）をインストールし、LINE メッセージング API 経由で外部から操作できるようにするための全手順をまとめたものです。

---

## 1. 前提知識と環境準備
* **OS**: macOS (Mac Mini)
* **必須要件**: Node.js バージョン **22.16.0 以上** が必要です。
  * `brew install node@22` 等でインストール後、シンボリックリンクなどの競合（`npm/LICENSE`）が発生した場合は `brew link --overwrite node@22` で強制リンクしてパスを通します。

## 2. OpenClaw 本体と API キーのセットアップ
1. **インストール**
   ```bash
   curl -fsSL https://openclaw.ai/install.sh | bash
   ```
2. **オンボーディング（初期設定）**
   Google Gemini (AI Studio) の API キーが必要です。
   ```bash
   openclaw onboard
   ```
   * プロバイダーとして **Google** を選択し、保持している API キーを入力します。
   * デフォルトモデルは **google/gemini-3.1-pro-preview** などを選択。
   * LINE チャネルや Web 検索 (Gemini グラウンディング) などの設定もここで行います。

## 3. LINE Messaging API との設定 (設定ファイルの直接編集)
`openclaw configure` コマンドで LINE の設定変更がうまくいかない場合は、設定ファイル (`~/.openclaw/openclaw.json`) を直接編集するのが確実です。

* **編集ファイル**: `~/.openclaw/openclaw.json`
* 以下の構造になるよう、LINE Developers Console で取得した **「チャネルシークレット (Channel Secret)」** と **「チャネルアクセストークン (Channel Access Token)」** を追加します。

```json
  "channels": {
    "line": {
      "enabled": true,
      "channelSecret": "あなたのチャネルシークレット",
      "channelAccessToken": "あなたのアクセストークン"
    }
  }
```

## 4. 外部公開（Tailscale Funnel）の設定
LINE サーバーからあなたの Mac Mini（ローカル環境）の OpenClaw へ通信を届けるため、Tailscale を使ってセキュアな「トンネル（公開 URL）」を作ります。

1. **Tailscale のインストールとログイン**
   ```bash
   brew install tailscale
   sudo brew services start tailscale
   sudo tailscale up
   ```
   ※ターミナルに表示される URL (`https://login.tailscale.com/...`) にアクセスし、認証を完了させます。

2. **OpenClaw (18789番ポート) を外部に公開する**
   以下のコマンドを実行し、LINE 用の Webhook 経路を作ります。
   ```bash
   tailscale serve --bg --yes 18789
   sudo tailscale funnel 18789
   ```
3. **公開 URL の確認**
   ```bash
   tailscale funnel status
   ```
   表示された `https://[あなたの英数字].ts.net/` がベースの URL になります。

## 5. LINE 側の Webhook 設定
1. **[LINE Developers Console](https://developers.line.biz/console/)** にログイン。
2. 対象チャネルの **「Messaging API設定」** タブを開く。
3. **Webhook URL** 欄に先ほどの URL に `/line/webhook` を足して設定します。
   * 例: `https://[あなたの英数字].ts.net/line/webhook`
4. **「更新 (保存)」** を押し、**「検証」** ボタンを押して「成功」と出ることを確認。
5. 直下の **「Webhookの利用」** を **オン** にします。

> **【重要】応答メッセージの無効化**
> （[LINE Official Account Manager](https://manager.line.biz/) または Webhook 設定の下部リンクから）
> 「応答設定」の **「応答メッセージ」 を必ず「オフ（無効）」** にしてください。これがオンだとボットが自動で「個別のお問い合わせを受け付けておりません」と返してしまいます。

## 6. セキュリティアクセス承認 (初回のみ)
OpenClaw は初期状態でセキュリティが高く設定されており、見知らぬ LINE パートナーからのアクセスは「未承認 (access not configured)」としてブロックされます。

1. LINE アプリからボットへ適当なメッセージ（「こんにちは」など）を送信します。
   （※この時点ではブロックされ、何も返ってきません）
2. Mac のターミナルでリクエスト一覧を確認します。
   ```bash
   openclaw pairing list
   ```
   送信者のリストと **8桁のコード (例: C7TNDBS5)** が表示されます。
3. アクセスを承認します。
   ```bash
   openclaw pairing approve <該当のコード>
   ```

## 7. バックグラウンドサービスの永続起動
最後に OpenClaw のゲートウェイサービスを起動させます。
```bash
openclaw gateway --force
```

これで、いつでもどこからでもスマホの LINE 経由で、Mac Mini に居座る AI アシスタント（Gemini）を操作できるようになります。
