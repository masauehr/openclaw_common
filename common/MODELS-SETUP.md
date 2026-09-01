# モデル認証・切り替え ガイド

OpenClaw で利用可能なモデルプロバイダーの認証セットアップと切り替え手順。

---

## 📊 現在のモデル構成（2026-09-01）

| 用途 | プロバイダー | モデル | 認証方式 | 状態 |
|------|---------|--------|--------|------|
| **対話（Slack など）** | Anthropic | `claude-haiku-4-5` | setup-token | ✅ 利用中 |
| **フォールバック（対話）** | Ollama | `ollama/qwen3.6:35b-mlx` | ローカル | ⚠️ 課金枠切れ時 |
| **cron ジョブ** | Anthropic | `claude-haiku-4-5` | setup-token | ✅ 利用中 |
| **cron フォールバック** | Anthropic | `claude-sonnet-5` | setup-token | 🔄 予備 |
| **ローカル検証用** | Ollama | `ornith-1.5:35b` など | ローカル | ❌ 非推奨 |

---

## 🔐 Anthropic（Claude）セットアップ

### 前提

- Claude API アカウント（<https://console.anthropic.com/>）
- Claude CLI または API キーアクセス

### 手順 1: setup-token の取得

#### A) Claude CLI を使用（推奨）

```bash
# Claude CLI インストール
npm install -g @anthropic-ai/claude-cli

# トークン取得インタラクティブ
claude setup-token
```

ブラウザが開き、Anthropic アカウントにログイン → API キーの認可画面に進みます。

キーをコピー → CLI に貼り付け → クレデンシャルが `~/.anthropic/credentials.json` に保存されます。

#### B) API キーを直接使用

```bash
# ~/.anthropic/credentials.json を手動作成
mkdir -p ~/.anthropic
cat > ~/.anthropic/credentials.json << 'EOF'
{
  "default": {
    "api_key": "sk-ant-xxxxxxxxxxxxxxxx"
  }
}
EOF
```

### 手順 2: OpenClaw に登録

```bash
openclaw models auth login --provider anthropic
```

プロンプト:

```
? Auth method for anthropic?
  1. Anthropic setup-token (recommended)
  2. API key directly
  3. Skip
→ 1     # setup-token を選択
```

**確認**:

```bash
openclaw models list --provider anthropic
```

期待される出力:

```
anthropic/claude-haiku-4-5
anthropic/claude-sonnet-5
anthropic/claude-opus-4-6
```

### 手順 3: デフォルトモデル設定

```bash
# Primary モデル設定
openclaw config set agents.defaults.model.primary anthropic/claude-haiku-4-5

# Fallback 設定
openclaw config set agents.defaults.model.fallbacks '["anthropic/claude-sonnet-5"]'
```

### 手順 4: セッション別の確認

```bash
# 対話セッション確認
openclaw sessions list
# Model: anthropic/claude-haiku-4-5

# 特定セッションのモデル確認
openclaw sessions get <session-key> --model
```

---

## 🖥️ Ollama（ローカルモデル）セットアップ

### 前提

- Ollama インストール済み
- Ollama サーバ起動（`ollama serve`）
- localhost:11434 で疎通可能

### 手順 1: Ollama インストール・起動

```bash
# macOS Homebrew
brew install ollama

# 確認
ollama --version
```

### 手順 2: モデルのダウンロード

```bash
# 現在のこのMac構成では非推奨ですが、テスト用途の場合:

ollama pull qwen3.6:35b-mlx        # fallback 用（若干マシ）
ollama pull ornith-1.5:35b         # テスト用（性能低い）
ollama pull nemotron:12b-instruct  # 試験用
```

### 手順 3: 動作確認

```bash
# Ollama が起動していることを確認
curl -s http://localhost:11434/api/generate \
  -d '{"model":"qwen3.6:35b-mlx","prompt":"Hello"}'
```

### 手順 4: OpenClaw に登録

Ollama は通常自動検出されます。確認:

```bash
openclaw models list --provider ollama
```

出力例:

```
ollama/qwen3.6:35b-mlx
ollama/ornith-1.5:35b
ollama/nemotron:12b-instruct
```

### 手順 5: Fallback として設定

```bash
openclaw config set agents.defaults.model.fallbacks '["ollama/qwen3.6:35b-mlx"]'
```

---

## 🔄 モデル切り替え（セッション単位）

### 一時的な変更（対話中）

```bash
# 現在のセッションで別モデルに切り替え
/model sonnet
# or
/model qwen
```

**効果**: その turn 以降のみ適用。セッション終了時に戻される。

### 永続的な変更（デフォルト）

```bash
# config で primary 変更
openclaw config set agents.defaults.model.primary anthropic/claude-sonnet-5

# Gateway 再起動は不要（即座に反映）
```

### 確認

```bash
# 現在の設定確認
openclaw config get agents.defaults.model
```

出力例:

```json
{
  "primary": "anthropic/claude-sonnet-5",
  "fallbacks": ["anthropic/claude-haiku-4-5", "ollama/qwen3.6:35b-mlx"]
}
```

---

## 🎯 推奨モデル選定ガイド

### 対話用（Slack DM など）

| モデル | 速度 | 品質 | 日本語 | コスト | 推奨度 |
|--------|------|------|--------|--------|--------|
| **claude-haiku-4-5** | 🟢 速い | 🟡 中 | ✅ 優秀 | 💰 安 | ⭐⭐⭐⭐⭐ |
| claude-sonnet-5 | 🟡 普通 | 🟢 優秀 | ✅ 優秀 | 💰💰 中 | ⭐⭐⭐⭐ |
| claude-opus-4-6 | 🔴 遅い | 🟢🟢 最高 | ✅ 優秀 | 💰💰💰 高 | ⭐⭐⭐ |
| qwen3.6:35b-mlx | 🔴 遅い | 🔴 低い | ⚠️ 混入 | 無料 | ⭐ |
| ornith-1.5:35b | 🔴🔴 超遅 | 🔴 無視 | ❌ 無反応 | 無料 | ❌ |

**結論**: 課金枠がある限り **haiku**（対話）か **sonnet**（高品質必須時）が最適。

### cron ジョブ用

| モデル | 処理時間 | 品質 | コスト | 推奨度 |
|--------|----------|------|--------|--------|
| **claude-haiku-4-5** | 8-12秒 | 中 | 💰 安 | ⭐⭐⭐⭐⭐ |
| claude-sonnet-5 | 15-25秒 | 優秀 | 💰💰 中 | ⭐⭐⭐⭐（必要な場合） |
| qwen3.6:35b-mlx | 45-120秒 | 低い | 無料 | ⭐（GPU不足時の保険） |

**結論**: cron 3ジョブは **haiku** + fallback **sonnet** で十分。

---

## 💳 Anthropic 課金枠管理

### 残高確認

```bash
# Claude CLI で確認
claude models list
```

### 課金管理ダッシュボード

<https://console.anthropic.com/account/billing/overview>

- **Pay-as-you-go**: 実績課金（月額）
- **Pro サブスク**: 月額固定（余剰利用なし）

**このMacの運用**: Claude Pro サブスク（月額 $20）。余った分は消失。

### 低電力運用のコツ

1. **対話は haiku**（5倍コスト安 vs sonnet）
2. **cron も haiku**（予備は sonnet）
3. **ローカル Ollama** を fallback に設定
4. **セッション肥大防止**（`openclaw sessions compact`）

---

## 🔧 カスタムプロバイダー（自社 API など）

### OpenRouter（プロキシサービス）

複数モデルを一元管理:

```bash
openclaw config set models.providers.openrouter.apiKey "sk-or-xxx"
openclaw models list --provider openrouter
```

### 自社 LLM サーバ

カスタム base URL 指定:

```json
{
  "models": {
    "providers": {
      "custom": {
        "baseUrl": "https://llm.example.com/v1",
        "apiKey": "***"
      }
    }
  }
}
```

詳細: [CONFIG-REFERENCE.md](CONFIG-REFERENCE.md)

---

## 🚨 トラブルシューティング

### モデル認証が失敗する

```bash
# 認証情報クリア
rm ~/.anthropic/credentials.json
openclaw models auth logout --provider anthropic

# 再認証
openclaw models auth login --provider anthropic
```

### モデルがリストに出ない

```bash
# Anthropic の場合
openclaw models list --provider anthropic --verbose

# Ollama の場合（localhost:11434 確認）
curl http://localhost:11434/api/tags
```

### 対話が遅い・エラー

```bash
# セッション肥大を確認
openclaw sessions list --detailed

# 肥大した場合は compaction
openclaw sessions compact <session-key> --max-lines 500
```

### fallback が機能しない

```bash
# 設定確認
openclaw config get agents.defaults.model

# fallback が配列か確認
openclaw config set agents.defaults.model.fallbacks '["anthropic/claude-sonnet-5"]'

# 設定がロードされたか確認（gateway ログ）
openclaw gateway logs --follow
```

---

## 📝 設定例

### シナリオ: コスト削減（Haiku 優先）

```bash
openclaw config set agents.defaults.model.primary anthropic/claude-haiku-4-5
openclaw config set agents.defaults.model.fallbacks '["anthropic/claude-sonnet-5","ollama/qwen3.6:35b-mlx"]'
```

### シナリオ: 高品質重視（Sonnet）

```bash
openclaw config set agents.defaults.model.primary anthropic/claude-sonnet-5
openclaw config set agents.defaults.model.fallbacks '["anthropic/claude-opus-4-6"]'
```

### シナリオ: ローカルのみ（オフライン対応）

```bash
openclaw config set agents.defaults.model.primary ollama/qwen3.6:35b-mlx
openclaw config unset agents.defaults.model.fallbacks
```

---

**関連ドキュメント**:
- [CONFIG-REFERENCE.md](CONFIG-REFERENCE.md) — 詳細な設定オプション
- [ARCHITECTURE.md](ARCHITECTURE.md) — モデルルーティング図
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) — トラブル詳細
- 公式: <https://docs.openclaw.ai/concepts/models>

**最後に更新**: 2026-09-01
