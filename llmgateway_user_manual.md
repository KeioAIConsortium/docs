# LLM Gateway ユーザマニュアル

このマニュアルは、AIC（AIコンソーシアム）の LLM Gateway を初めてご利用になる方向けの手順書です。

ご不明点・ご要望・トラブル等がございましたら、`aic-server-group@keio.jp` までご連絡ください。

## 申請

LLM Gateway のご利用には、事前に利用登録と API キーの発行が必要です。  
ご利用を希望される方は、運営へご連絡のうえ、案内に沿って申請をお願いいたします。

申請完了後、以下の情報をご案内します。

- 利用用 API キー
- 利用するモデル名（通常 `gpt-oss-120b`）
- 接続先 URL（`https://llm-gateway.keioaic.dev/v1`）

## アクセス手順

実際のご利用には、`Codex` または `OpenCode` のいずれかをご使用ください。  
初回は、どちらか 1 つのみ設定していただければ問題ありません。

### 1. APIキーを環境変数へ設定

まず、ターミナルで次のコマンドを実行してください。

```bash
export KEIOAIC_LLM_API_KEY="発行されたAPIキー"
```

毎回入力する手間を減らしたい場合は、`~/.bashrc` または `~/.zshrc` に同じ行を追加してください。

### 2-A. Codex をご利用の場合

#### インストール

```bash
# npm
npm install -g @openai/codex
```

macOS をご利用の場合は、Homebrew でもインストールできます。

```bash
brew install --cask codex
```

Windows 環境では、WSL 上でのご利用を推奨しています。

#### 設定ファイル

`~/.codex/config.toml` を作成し、次の内容を記載してください。

```toml
model_provider = "litellm"
model = "gpt-oss-120b"
web_search = "disabled"

[model_providers.litellm]
name = "LiteLLM"
base_url = "https://llm-gateway.keioaic.dev/v1"
env_key = "KEIOAIC_LLM_API_KEY"
wire_api = "responses"
requires_openai_auth = false
```

#### 起動方法

```bash
codex
```

### 2-B. OpenCode をご利用の場合

#### インストール

```bash
curl -fsSL https://opencode.ai/install | bash
```

また、npm からインストールすることも可能です。

```bash
npm install -g opencode-ai
```

#### 設定ファイル

プロジェクト直下に `opencode.jsonc` を作成し、次の内容を記載してください。

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "litellm": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LiteLLM (LLM Gateway)",
      "options": {
        "baseURL": "https://llm-gateway.keioaic.dev/v1",
        "apiKey": "${KEIOAIC_LLM_API_KEY}"
      },
      "models": {
        "gpt-oss-120b": {}
      }
    }
  },
  "model": "litellm/gpt-oss-120b"
}
```

#### 起動方法

```bash
opencode
```

## よくあるトラブル

### APIキーエラーになる場合

- API キーのコピー時に、余分な空白や改行が入っていないかご確認ください
- `KEIOAIC_LLM_API_KEY` を設定したシェルと同じシェルで実行しているかご確認ください
- API キーの期限切れや停止の可能性がある場合は、運営へお問い合わせください

### 設定ファイルを書いたのに反映されない場合

- ファイルの保存場所をご確認ください
- `toml` / `jsonc` の記法に崩れ（カンマ抜けなど）がないかご確認ください
- 一度ターミナルを開き直し、再度実行してください

### Windows でうまく動かない場合

- WSL（Ubuntu など）上で設定・実行してください
- VS Code をご利用の場合は、WSL 接続状態のターミナルで実行してください

### どのツールを使えばよいかわからない場合

- 迷われた場合は、まず `Codex` のご利用をおすすめします
- エディタ連携を重視される場合は、`OpenCode` もご検討ください

## 参考リンク

- Codex CLI: https://github.com/openai/codex
- Codex ドキュメント: https://developers.openai.com/codex/
- OpenCode ドキュメント: https://opencode.ai/docs
