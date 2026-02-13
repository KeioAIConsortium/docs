# LLM Gateway ユーザマニュアル

このマニュアルは、AIC（AIコンソーシアム）の LLM Gateway サービスの利用者のためのマニュアルです。

ご不明点・ご要望・トラブル等がございましたら、`aic-server-group@keio.jp` までご連絡ください。

## 申請

LLM Gateway のご利用には、事前に利用登録と API キーの発行が必要です。  
ご利用を希望される方は、[こちらのフォーム]() から申請を行ってください。
申請が通り次第、API キーが送付されます。

## アクセス手順

このプロジェクトでは、OpenAI 互換の API を提供しています。
基本的には、OpenAI 互換の API に対応したツールからであれば利用できますが、完全な互換ではないため、一部のツールでは動作しない場合があります。

以下では、Continue（VS Code 拡張および CLI）を利用する手順を説明します。

## Continue（VS Code拡張）をご利用の場合

### インストール

1. VS Code の拡張機能を開きます
2. `Continue` を検索してインストールします

### 設定ファイル

`~/.continue/config.yaml` に、以下の内容で LLM Gateway 用のモデル設定を追加してください。

```yaml
name: keioaic-litellm
version: 1.0.0
schema: v1

models:
  - name: gpt-oss-120b
    provider: openai
    apiBase: https://llm-gateway.keioaic.dev/v1
    apiKey: ${{ secrets.KEIOAIC_LLM_API_KEY }}
    model: gpt-oss-120b
    roles:
      - chat
      - edit
      - apply
      - autocomplete
```

### APIキーを設定

`~/.continue/.env` に、以下の内容で API キーを設定してください。

```dotenv
KEIOAIC_LLM_API_KEY=<ここに発行されたAPIキーを貼り付け>
```

※ ファイル保存後は VS Code / CLI を再起動してください。

### 使い方

1. Continue サイドバーを開きます
2. モデルで `gpt-oss-120b` を選びます
3. チャット欄に依頼を入力して利用します

## Continue CLI（`cn`）をご利用の場合

### インストール

```bash
# macOS / Linux
curl -fsSL https://raw.githubusercontent.com/continuedev/continue/main/extensions/cli/scripts/install.sh | bash

# Node.js 20+ がインストールされている場合
npm i -g @continuedev/cli
```

### 起動方法

`cn` は VS Code拡張 と同じ設定ファイル・環境変数を利用します。
上記のVS Code拡張を利用する場合と同様に設定ファイル・環境変数を準備した上で、以下のコマンドで起動します。

Continue CLI を利用するには、デフォルトでは Continue アカウントが必要ですが、LLM Gateway のみを利用する場合はアカウント不要で利用できます。

`.continue/config.yaml` を読み込ませるために、シェルの初期化ファイル（例: `~/.bashrc`, `~/.zshrc`）に以下を追加してください。

```bash
alias cn='cn --config ~/.continue/config.yaml'
```

シェルを再起動した後、以下のコマンドで Continue CLI を起動することができます。

```bash
cn
```

## よくあるトラブル

### APIキーエラーになる場合

- `~/.continue/.env` の変数名が `KEIOAIC_LLM_API_KEY` になっているかご確認ください
- 値に引用符（`"`）や余分な空白が入っていないかご確認ください
- VS Code / `cn` を再起動してください

## 参考リンク

- Continue VS Code: https://docs.continue.dev/install/vscode
- Continue CLI 導入: https://docs.continue.dev/cli/install
- Continue CLI の使い方: https://docs.continue.dev/guides/cli
