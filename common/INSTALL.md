# OpenClaw インストール・初期セットアップ

最新版（`2026.7.1-2`）のインストール手順と初期セットアップ。

**前提環境**:
- Node.js 22.22.3+、24.15+、25.9+
- npm 11.15+（`--allow-scripts` サポート）
- macOS / Linux / WSL2 / Windows
- インターネット接続

**公式ドキュメント**: <https://docs.openclaw.ai/>

---

## 📋 前提条件の確認

### Node.js / npm バージョン確認

```bash
node --version    # v25.9.0 以上
npm --version     # 11.15+ 推奨
```

**現在のこのMac**:
- Node: v25.9.0 ✅
- npm: 11.12.1 ⚠️（要確認）
- nvm: ~/.nvm にあり

### npm のアップグレード（必要に応じて）

```bash
npm install -g npm@latest  # 最新版へ
npm install -g npm@12      # or 具体的なバージョン
```

### Ollama のセットアップ（ローカルモデル使用時）

```bash
# Ollama インストール（macOS Homebrew）
brew install ollama

# 起動・確認
ollama serve                      # バックグラウンド起動
ollama list                       # インストール済みモデル
ollama pull qwen3.6:35b-mlx       # モデル追加（必要に応じて）
```

**確認**: http://localhost:11434/api/generate で疎通確認

---

## 🚀 ステップ 1: OpenClaw インストール

### オプション A: npm グローバルインストール（推奨）

```bash
npm install -g openclaw@2026.7.1-2
```

**確認**:
```bash
which openclaw                    # /opt/homebrew/bin/openclaw
openclaw --version               # 2026.7.1-2 (0790d9f)
```

### オプション B: ソースからビルド

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
npm install
npm run build                     # TypeScript → JavaScript
npm install -g .                  # グローバルインストール
```

---

## 🎯 ステップ 2: オンボーディング

```bash
openclaw onboard
```

対話的にセットアップを進めます。以下の選択肢が出ます:

### 2.1 ゲートウェイモード

```
? Gateway mode? (local / loopback / lan)
→ local      # ローカルのみ（推奨）
```

### 2.2 バインドアドレス

```
? Bind to? (loopback / lan / tailnet / custom)
→ loopback   # localhost のみ（デフォルト）
```

### 2.3 ポート（デフォルト 18789）

```
? Gateway WebSocket port?
→ 18789     # デフォルト
```

### 2.4 認証方式

```
? Auth mode? (none / token / password / trusted-proxy)
→ token     # トークン認証（推奨）
```

自動生成されたトークンが `~/.openclaw/openclaw.json` に保存されます。

### 2.5 ワークスペースパス

```
? Workspace directory?
→ ~/.openclaw/workspace    # デフォルト
```

### 2.6 モデル選択

```
? Model? (anthropic / openai / ollama / ...)
→ anthropic               # クラウド推奨
  or
→ ollama                  # ローカル
```

**現在のセットアップ**: `anthropic/claude-haiku-4-5`（後ほど設定で変更可）

### 2.7 launchd 登録（macOS）

```
? Install as launchd service?
→ y         # yes（常時起動設定）
```

**生成ファイル**:
```
~/Library/LaunchAgents/ai.openclaw.gateway.plist
```

ゲートウェイが自動起動するようになります。

---

## ✅ ステップ 3: セットアップ確認

### 3.1 ワークスペース確認

```bash
ls -la ~/.openclaw/
```

**期待される構成**:
```
~/.openclaw/
├── openclaw.json              # 設定（token 含む）
├── workspace/                 # エージェントワークスペース
│  ├── IDENTITY.md
│  ├── SOUL.md
│  ├── USER.md
│  ├── MEMORY.md
│  ├── TOOLS.md
│  └── bin/
├── agents/
│  └── main/
│     ├── sessions/
│     └── agent/
├── state/
│  └── openclaw.sqlite
└── ...
```

### 3.2 Gateway 起動確認

```bash
openclaw status
```

**期待される出力**:
```
OpenClaw status

Overview
├─ Gateway: local · ws://127.0.0.1:18789 (local loopback) · reachable
├─ Gateway service: LaunchAgent loaded · running (pid XXXXX)
├─ Agents: 1 · sessions 0 · default main active
└─ ...
```

### 3.3 ダッシュボード確認

```bash
openclaw dashboard
```

ブラウザで `http://127.0.0.1:18789/` が開きます。  
トークン認証画面 → ログイン → Control UI が表示されます。

### 3.4 ゲートウェイログ確認

```bash
openclaw gateway logs --follow
```

起動ログ・エラーを確認。`[info] Gateway ready` で OK。

---

## 🔑 ステップ 4: モデル認証セットアップ

### 4.1 Anthropic（Claude）使用時

#### a) setup-token の取得

Claude CLI をインストール:

```bash
npm install -g @anthropic-ai/claude-cli
```

API キーをセットアップ:

```bash
claude setup-token
```

ブラウザで <https://console.anthropic.com/> にログイン → API キーを生成 → ペースト。

#### b) OpenClaw に登録

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
# anthropic/claude-haiku-4-5
# anthropic/claude-sonnet-5
# ...
```

### 4.2 Ollama（ローカルモデル）使用時

```bash
# Ollama がローカルで起動していることを確認
ollama serve &

# OpenClaw は自動検出
openclaw models list --provider ollama
# ollama/qwen3.6:35b-mlx
# ollama/...
```

特に認証設定は不要（ローカルのため）。

---

## 🔌 ステップ 5: モデル設定（デフォルト変更）

### 現在の設定確認

```bash
openclaw config get agents.defaults.model
```

出力例:

```json
{
  "primary": "anthropic/claude-haiku-4-5",
  "fallbacks": ["anthropic/claude-sonnet-5"]
}
```

### デフォルトモデル変更

```bash
# primary を変更
openclaw config set agents.defaults.model.primary anthropic/claude-sonnet-5

# fallback を追加
openclaw config set agents.defaults.model.fallbacks '["anthropic/claude-sonnet-5", "ollama/qwen3.6:35b-mlx"]'
```

**注**: 設定は即座に反映（Gateway 再起動不要）。

---

## 💬 ステップ 6: チャットアプリ連携（Slack の例）

### 6.1 Slack App 作成

1. <https://api.slack.com/apps> にアクセス
2. 「Create New App」→ 「From scratch」
3. App name: `OpenClaw` / Workspace: `openclaw-test` を選択
4. 作成

### 6.2 Socket Mode 有効化

左メニュー → 「Socket Mode」→ 「Enable Socket Mode」

トークン 2 つが生成:
- **App-Level Token** (`xapp-*`)
- **Bot Token** (`xoxb-*`) ← 「OAuth & Permissions」から確認

### 6.3 OpenClaw に設定

```bash
openclaw config set channels.slack.enabled true
openclaw config set channels.slack.appToken "xapp-1-xxx"
openclaw config set channels.slack.botToken "xoxb-xxx"
```

または `~/.openclaw/openclaw.json` を直接編集:

```json
{
  "channels": {
    "slack": {
      "enabled": true,
      "appToken": "xapp-1-xxx",
      "botToken": "xoxb-xxx",
      "dmPolicy": "allowlist",
      "allowFrom": ["slack:U0XXXXXXXXX"]
    }
  }
}
```

### 6.4 再起動・確認

```bash
openclaw gateway restart
openclaw status
```

Slack の Bot が online になれば OK。

詳細: [CHANNELS.md](CHANNELS.md)

---

## 📝 ステップ 7: ワークスペース設定

### 7.1 エージェント人格の設定

`~/.openclaw/workspace/IDENTITY.md`:

```markdown
# IDENTITY.md

- **Name**: クゥ（Kuu）
- **Creature**: 黒猫
- **Emoji**: 🐈‍⬛
- **Avatar**: (データ URI or URL)
```

### 7.2 応答言語・性格の設定

`~/.openclaw/workspace/SOUL.md`:

```markdown
## Language

Always reply in 日本語 (Japanese). This is the default for every channel.
Only switch languages if the user explicitly asks.
```

### 7.3 ユーザー情報の設定

`~/.openclaw/workspace/USER.md`:

```markdown
# USER.md

- **Name**: (あなたの名前)
- **Timezone**: Asia/Tokyo
- **Language**: 日本語
- **Notes**: (カスタムメモ)
```

---

## 🔧 ステップ 8: 補助スクリプトの設定

気象庁（JMA）アメダス観測値ランキングを利用する場合:

### 8.1 スクリプト配置

```bash
# bin/jma_rank.py をコピー
mkdir -p ~/.openclaw/workspace/bin
cp ~/projects/openclaw_setup/bin/jma_rank.py ~/.openclaw/workspace/bin/
chmod +x ~/.openclaw/workspace/bin/jma_rank.py
```

### 8.2 TOOLS.md に記録

`~/.openclaw/workspace/TOOLS.md`:

```markdown
## 気象庁（JMA）データ取得

スクリプト: `~/.openclaw/workspace/bin/jma_rank.py`

用法:
```bash
python3 ~/.openclaw/workspace/bin/jma_rank.py <element> [top]
  element: temp | wind | gust | precip1h | precip24h | humidity
  top: 上位件数（既定 10）
```

例:
```bash
python3 ~/.openclaw/workspace/bin/jma_rank.py temp 10
python3 ~/.openclaw/workspace/bin/jma_rank.py precip24h 5
```
```

---

## 🎉 セットアップ完了

### 確認チェックリスト

- [ ] Node.js / npm バージョン確認済み
- [ ] OpenClaw インストール済み（`openclaw --version`）
- [ ] オンボーディング完了（launchd 登録含む）
- [ ] Gateway が running（`openclaw status`）
- [ ] ダッシュボード開閉確認（`openclaw dashboard`）
- [ ] モデル認証完了（Anthropic or Ollama）
- [ ] チャットアプリ連携済み（Slack など）
- [ ] ワークスペース設定完了（IDENTITY / SOUL など）
- [ ] 補助スクリプト配置済み（JMA など必要なら）

### 次のステップ

1. **cron ジョブ設定**: [AUTOMATION.md](AUTOMATION.md)
2. **詳細設定**: [CONFIG-REFERENCE.md](CONFIG-REFERENCE.md)
3. **運用ガイド**: [OPERATIONAL-GUIDE.md](OPERATIONAL-GUIDE.md)
4. **トラブル**: [TROUBLESHOOTING.md](TROUBLESHOOTING.md)

---

**最後に更新**: 2026-09-01  
**バージョン**: OpenClaw `2026.7.1-2`
