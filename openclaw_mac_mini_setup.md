# OpenClaw 環境構築 & LINE連携ガイド (Mac Mini編)

Mac Mini (macOS) 環境で、Google Geminiを搭載したOpenClawをセットアップし、LINEから操作できるようにするための手順書です。

## 1. 事前準備 (macOS)

1.  **Homebrew のインストール** (未導入の場合):
    ```bash
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    ```
2.  **Node.js (v22以上) のインストール**:
    ```bash
    brew install node@22
    brew link node@22
    ```

---

## 2. OpenClawのインストールと初期設定

### ステップA: インストール
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

### ステップB: Gemini APIの設定
Google AI Studioで取得したAPIキーを使用して実行します。

```bash
openclaw onboard --gemini-api-key [あなたのAPIキー] --install-daemon --accept-risk --non-interactive --skip-channels --skip-ui
```

---

## 3. LINE連携の手順 (Messaging API)

### ① LINE側の準備 (LINE Developers)
1.  [LINE Developers Console](https://developers.line.biz/console/) にログイン。
2.  **Messaging API** チャネルを新規作成。
3.  以下を取得：
    *   **Channel Secret** (「チャネル基本設定」タブ)
    *   **Channel Access Token** (「Messaging API設定」タブ)
4.  「Messaging API設定」タブで **「Webhookの利用」** を **オン** にする。

### ② OpenClaw側の設定
```bash
openclaw configure --section channels.line
```
※ `Channel Secret` と `Access Token` を入力します。

### ③ Webhook URLの設定
外部からアクセス可能にするため、OpenClawの機能を利用します。
```bash
openclaw onboard --tailscale funnel
```
発行されたURL（例: `https://xxx.ts.net/webhook/line`）を、LINE Developersの **「Webhook URL」** に登録します。

---

## 4. Mac Miniで環境を再現するためのプロンプト

> **AIへの依頼プロンプト:**
> 「Mac Mini (macOS) 上に OpenClaw の環境を構築してください。Google Gemini を使用し、バックグラウンドサービスとして稼働させる設定を含めてください。また、LINE Messaging APIとの連携準備もお願いします。」
