# OpenClaw アーキテクチャ

OpenClaw の全体構成・コンポーネント間の関係図。

---

## 🏗️ システムアーキテクチャ

```
┌─────────────────────────────────────────────────────────────────┐
│                        このMac (macOS)                          │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ OpenClaw Gateway (launchd: ai.openclaw.gateway)          │  │
│  │ PID: 80839 / localhost:18789 / token auth / mode=local   │  │
│  │                                                          │  │
│  │  ┌────────────────────────────────────────────────┐    │  │
│  │  │ Model Router                                  │    │  │
│  │  │                                                │    │  │
│  │  ├─ Primary:  anthropic/claude-haiku-4-5        │    │  │
│  │  ├─ Fallback: ollama/qwen3.6:35b-mlx (local)    │    │  │
│  │  │           anthropic/claude-sonnet-5 (cron)   │    │  │
│  │  └────────────────────────────────────────────────┘    │  │
│  │                                                          │  │
│  │  ┌──────────────┐  ┌────────────┐  ┌──────────────┐   │  │
│  │  │ Slack Router │  │ Cron Agent │  │ Web Search   │   │  │
│  │  │ (Socket Mode)│  │            │  │ (SearXNG)    │   │  │
│  │  └──────────────┘  └────────────┘  └──────────────┘   │  │
│  │          ▲               ▲               ▲             │  │
│  │          │ DM / message  │ schedule      │             │  │
│  │          │               │               │             │  │
│  └──────────┼───────────────┼───────────────┼─────────────┘  │
│             │               │               │                │
│  ┌──────────▼──────┐  ┌─────▼──────┐  ┌────▼──────────────┐ │
│  │ Anthropic API   │  │  Ollama    │  │ SearXNG           │ │
│  │                 │  │ :11434     │  │ localhost:18899   │ │
│  │ (setup-token)   │  │            │  │ (launchd)         │ │
│  │                 │  │ ornith-1.5 │  │                   │ │
│  │ claude-haiku    │  │ qwen3.6    │  │ self-hosted       │ │
│  │ claude-sonnet   │  │            │  │ DuckDuckGo bypass │ │
│  └─────────────────┘  └────────────┘  └───────────────────┘ │
│                                                                │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │ Workspace & State                                       │  │
│  │                                                          │  │
│  │ ~/.openclaw/                                            │  │
│  │  ├─ openclaw.json (settings, token, auth)              │  │
│  │  ├─ workspace/ (SOUL.md, IDENTITY.md, bin/jma_rank.py) │  │
│  │  ├─ agents/main/ (sessions, models.json)               │  │
│  │  └─ state/ (openclaw.sqlite)                           │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                                │
└─────────────────────────────────────────────────────────────────┘
         ▲
         │ WebSocket (Socket Mode)
         │
    ┌────────────────────────────────┐
    │     Slack (リモート)           │
    │                                │
    │ Workspace: openclaw-test       │
    │ App: OpenClaw                  │
    │ Auth: xapp-* / xoxb-*          │
    │ Socket Mode: enabled           │
    │                                │
    │ Channel: DM (u0btccxrpjp)      │
    └────────────────────────────────┘
```

---

## 🔌 コンポーネント詳細

### Gateway（中核）

| 項目 | 値 |
|------|-----|
| **プロセス** | `ai.openclaw.gateway`（launchd） |
| **バイナリ** | `/opt/homebrew/lib/node_modules/openclaw/gateway/bin/gateway.js` |
| **ポート** | `localhost:18789`（ローカルのみ） |
| **認証** | token auth（`gateway.auth.token`） |
| **ログ** | `~/Library/Logs/openclaw/gateway.log` |
| **起動** | `openclaw gateway start` or `launchctl start ai.openclaw.gateway` |

**主な責務**:
- チャットアプリからの入力を受け取る
- エージェント（AI）の実行・セッション管理
- モデルプロバイダへのルーティング
- cron ジョブのスケジューリング

### Agent（エージェント）

**名前**: クゥ（Kuu）🐈‍⬛

| 項目 | ファイル / 値 |
|------|---------|
| **ワークスペース** | `~/.openclaw/workspace/` |
| **人格** | `IDENTITY.md`（名前・絵文字・テーマ） |
| **ソウル** | `SOUL.md`（応答言語「日本語固定」） |
| **セッション** | `~/.openclaw/agents/main/sessions/sessions.json` |
| **状態 DB** | `~/.openclaw/agents/main/agent/openclaw-agent.sqlite` |

**主な責務**:
- 受け取ったメッセージを理解・応答生成
- ツール呼び出し（web_search / exec / file read など）
- メモリ・セッション管理

### Models（モデルプロバイダ）

#### 1. **Anthropic**（クラウド）

```
┌─────────────────────────────────────┐
│ Claude（Anthropic API）             │
│                                     │
│ - claude-haiku-4-5 （対話・cron）   │
│ - claude-sonnet-5  (cron fallback)  │
│                                     │
│ 認証: setup-token (Claude CLI)      │
│ 種類: Freemium / Pro サブスク        │
└─────────────────────────────────────┘
```

**セットアップ**:
```bash
claude setup-token
openclaw models auth login --provider anthropic
```

#### 2. **Ollama**（ローカル）

```
┌─────────────────────────────────────┐
│ Ollama (localhost:11434)            │
│                                     │
│ - qwen3.6:35b-mlx (fallback only)   │
│ - ornith-1.5:35b (非推奨)           │
│                                     │
│ 認証: なし（ローカル）               │
│ 実行: GPU/CPU                       │
└─────────────────────────────────────┘
```

**注**: Ollama ローカルモデルは日本語対話に不向き（幻覚・中国語混入）。  
課金枠切れ時の fallback 用のみ推奨。

### Web Search（SearXNG）

```
┌─────────────────────────────────────┐
│ SearXNG (localhost:18899)           │
│                                     │
│ - メタサーチ（複数エンジン統合）    │
│ - DuckDuckGo ボット判定 bypass      │
│ - API キー不要・上限なし             │
│                                     │
│ launchd: local.searxng              │
│ 設定: ~/searxng/settings.yml        │
└─────────────────────────────────────┘
```

**利用**:
- `web_search` / `web_fetch` ツール
- 気象・経済・AI ダイジェスト生成時

### Storage（ストレージ）

| パス | 内容 | 備考 |
|------|------|------|
| `~/.openclaw/openclaw.json` | 設定ファイル（token 含む） | git 非管理 |
| `~/.openclaw/workspace/` | エージェント人格・スクリプト | git 管理 |
| `~/.openclaw/agents/main/` | セッション・state DB | git 非管理 |
| `~/Library/Logs/openclaw/` | ログファイル | 自動ローテーション |
| `~/.openclaw/state/` | SQLite（メモリ・セッション） | git 非管理 |

---

## 📊 データフロー（例：Slack DM）

```
1. ユーザー: Slack で "今日の天気は?"
                      ▼
2. Slack Socket Mode → OpenClaw Gateway
                      ▼
3. Gateway: エージェント（クゥ）に渡す
   └─ モデル: claude-haiku-4-5 (Anthropic API)
   └─ ツール許可: web_search / exec / file_read など
                      ▼
4. エージェント: "SearXNG で気象情報を検索"
   └─ `web_search("日本 天気 今日")`
   └─ SearXNG (localhost:18899)
                      ▼
5. 検索結果 → エージェント処理 → 応答生成
                      ▼
6. Gateway: 応答を Slack DM に返す
```

---

## ⚡ Cron ジョブフロー

```
┌──────────────────────┐
│  システム時刻        │
│  毎日 09:10:00       │
└──────────────────────┘
         ▼
┌──────────────────────┐
│ Gateway Cron Handler │
│ (job: weather-jp-am) │
└──────────────────────┘
         ▼
┌──────────────────────────────────────────┐
│ 新規エージェントスレッド実行             │
│                                          │
│ - Model: anthropic/claude-haiku-4-5      │
│ - Fallback: anthropic/claude-sonnet-5   │
│ - Message: "[気象ダイジェスト生成]"      │
│ - Context: JMA 観測値ランキング          │
│   (exec: bin/jma_rank.py temp 10)        │
└──────────────────────────────────────────┘
         ▼
┌──────────────────────────────────────────┐
│ 完了時のデリバリ                         │
│                                          │
│ deliver=true → Slack DM に配信           │
│ channel: slack                           │
│ to: slack:U0XXXXXXXXX (オーナー)         │
│ announce: true                           │
└──────────────────────────────────────────┘
```

---

## 🔐 セキュリティ層

| 層 | 対策 |
|----|------|
| **Gateway 認証** | Token auth（`gateway.auth.token`） |
| **Slack 認証** | Socket Mode（xapp- / xoxb-） |
| **Anthropic 認証** | setup-token（Claude CLI） |
| **Ollama** | ローカルのみ（認証なし） |
| **SearXNG** | ローカルのみ（secret_key） |

**注**: `gateway.auth.token` は `~/.openclaw/openclaw.json` に平文保存。  
リポジトリ・共有禁止。詳細は [SECRETS-MANAGEMENT.md](SECRETS-MANAGEMENT.md)。

---

## 📦 ファイルツリー

```
~/.openclaw/
├── openclaw.json                      # 設定（token含む）
├── openclaw.json.bak                  # 自動バックアップ
├── workspace/
│  ├── IDENTITY.md                     # エージェント人格
│  ├── SOUL.md                         # エージェント性格・言語設定
│  ├── USER.md                         # ユーザー情報
│  ├── MEMORY.md                       # 長期メモリ
│  ├── TOOLS.md                        # ツール・環境固有設定
│  ├── memory/
│  │  └── YYYY-MM-DD.md                # 日次ログ
│  └── bin/
│     └── jma_rank.py                  # 気象庁ランキング抽出
├── agents/
│  └── main/
│     ├── agent/
│     │  ├── openclaw-agent.sqlite     # 認証ストア
│     │  └── models.json               # モデル設定
│     ├── sessions/
│     │  ├── sessions.json             # セッション一覧
│     │  └── <session-id>.json         # 個別セッション
│     └── memory/
│        └── (memory plugin data)
└── state/
   ├── openclaw.sqlite
   ├── openclaw.sqlite-shm
   └── openclaw.sqlite-wal

/opt/homebrew/lib/node_modules/openclaw/
├── bin/
│  ├── openclaw                        # CLI
│  └── ... (その他スクリプト)
├── src/
│  ├── gateway/                        # Gateway 実装
│  ├── agent/                          # エージェント実装
│  └── ... (コア実装)
└── ... (ドキュメント・設定)

~/Library/LaunchAgents/
├── ai.openclaw.gateway.plist          # Gateway launchd
└── local.searxng.plist                # SearXNG launchd

~/searxng/
├── settings.yml                       # SearXNG 設定
└── ... (キャッシュ・ログ)
```

---

## 🔄 生命サイクル

### Gateway 起動シーケンス

```
1. launchctl start ai.openclaw.gateway
2. → $HOME/.nvm/versions/node/<v>.x.x/bin/node
   └─ /opt/homebrew/lib/node_modules/openclaw/bin/openclaw gateway run

3. Gateway プロセス起動
   ├─ 設定ロード（~/.openclaw/openclaw.json）
   ├─ モデルカタログロード
   ├─ Slack プラグインロード → Socket Mode 接続試行
   ├─ Cron スケジューラ起動
   └─ HTTP サーバ起動 (localhost:18789)

4. Ready ▶ 入力受け取り開始
```

### セッション生命サイクル

```
1. ユーザーが Slack で DM を送信
   ├─ 既存セッションがあれば再利用（dmScope: per-channel-peer）
   ├─ なければ新規セッション作成

2. セッション実行
   ├─ トークンカウント → 肥大時は compaction
   ├─ メモリ検索（memory plugin）
   ├─ メッセージ処理 → モデル呼び出し
   ├─ ツール実行（必要に応じて）
   └─ 応答生成 → Slack に返送

3. セッション終了
   └─ 履歴を sessions.json に保存
```

---

**関連ドキュメント**:
- [CONFIG-REFERENCE.md](CONFIG-REFERENCE.md) — 詳細な設定オプション
- [OPERATIONAL-GUIDE.md](OPERATIONAL-GUIDE.md) — 日常運用・メンテナンス
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) — トラブル時の診断
