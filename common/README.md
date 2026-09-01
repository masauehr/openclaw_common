# OpenClaw Common — 共通ドキュメント・リファレンス

OpenClaw（セルフホスト型 AI アシスタント・ゲートウェイ）の共通設定・ドキュメント。  
導入作業の詳細ログは別途 [`openclaw_setup`](https://github.com/masauehr/openclaw_setup) を参照。

**最終更新**: 2026-09-01（`2026.7.1-2`）

---

## 📌 このフォルダについて

| ファイル | 内容 |
|---------|------|
| **[ARCHITECTURE.md](ARCHITECTURE.md)** | 全体構成図・コンポーネント関係 |
| **[INSTALL.md](INSTALL.md)** | インストール・初期セットアップ手順（最新） |
| **[CONFIG-REFERENCE.md](CONFIG-REFERENCE.md)** | 設定ファイル（`openclaw.json`）の完全リファレンス |
| **[MODELS-SETUP.md](MODELS-SETUP.md)** | モデル認証・切り替え手順（Anthropic / Ollama） |
| **[CHANNELS.md](CHANNELS.md)** | Slack / Telegram / Discord など、チャットアプリ連携 |
| **[AUTOMATION.md](AUTOMATION.md)** | cron ジョブ・定期実行の設定・運用 |
| **[TOOLS-SKILLS.md](TOOLS-SKILLS.md)** | スキル・ツール設定（web_search / JMA / 補助スクリプト） |
| **[OPERATIONAL-GUIDE.md](OPERATIONAL-GUIDE.md)** | 日常の運用・メンテナンス（ログ確認・セッション管理） |
| **[TROUBLESHOOTING.md](TROUBLESHOOTING.md)** | トラブルシューティング（最新） |
| **[SECRETS-MANAGEMENT.md](SECRETS-MANAGEMENT.md)** | シークレット管理・セキュリティ |
| **[MODELS.json](MODELS.json)** | サポートモデル一覧（リファレンス） |

---

## 🎯 現在の構成（2026-09-01）

### 基本情報

| 項目 | 値 |
|------|-----|
| **バージョン** | `2026.7.1-2` |
| **OS** | macOS 26.6.2（arm64） |
| **Gateway** | local / loopback:18789 / token auth |
| **ワークスペース** | `~/.openclaw/workspace/`（git 管理） |
| **設定ファイル** | `~/.openclaw/openclaw.json`（ローカルのみ） |

### モデル・認証

| 用途 | モデル | 認証方式 |
|------|--------|--------|
| **対話**（Slack DM など） | `anthropic/claude-haiku-4-5` | Anthropic（setup-token） |
| **フォールバック** | `ollama/qwen3.6:35b-mlx` | ローカル（課金枠切れ時） |
| **cron ジョブ** | `anthropic/claude-haiku-4-5` | Anthropic（同上） |
| **cron フォールバック** | `anthropic/claude-sonnet-5` | Anthropic（同上） |

**注**: ローカル Ollama（`ornith-1.5:35b` / `nemotron`）は日本語対話に不適切（幻覚・中国語混入・遅延）のため非推奨。

### Web 検索

**SearXNG**（自己ホスト）
- エンドポイント: `http://127.0.0.1:18899`
- launchd: `local.searxng`
- 特徴: キー不要・上限なし・DuckDuckGo のボット判定をバイパス

### エージェント（人格）

- **名前**: クゥ（Kuu）🐈‍⬛
- **応答言語**: 日本語固定（`~/.openclaw/workspace/SOUL.md` の Language セクション）
- **ファイル**: `~/.openclaw/workspace/IDENTITY.md`（名前・絵文字・テーマ）

### 定期実行（cron）

| job_id | name | 実行時刻（JST） | 内容 |
|--------|------|----------------|------|
| `445881d3…` | `weather-jp-am` | 毎日 09:10 | 日本の気象・防災まとめ |
| `b18e0f97…` | `econ-jp-am` | 毎日 09:13 | 日本株・為替・経済指標 |
| `db8a0d0b…` | `ai-weekly-digest` | 月曜 09:16 | 週次 AI 動向ダイジェスト |

すべて `anthropic/claude-haiku-4-5` + fallback `anthropic/claude-sonnet-5`。

### 補助スクリプト

- **`~/.openclaw/workspace/bin/jma_rank.py`**  
  気象庁（JMA）アメダス観測値のランキング抽出。  
  用途: 気温・風速・降水量などのトップ地点を Slack で返す。  
  例: `python3 ~/.openclaw/workspace/bin/jma_rank.py temp 10`

---

## 🚀 クイックスタート

### 1. インストール（初回のみ）

```bash
npm install -g openclaw@2026.7.1-2
openclaw onboard  # モデルアクセス・ワークスペース作成・launchd 登録
```

詳細は **[INSTALL.md](INSTALL.md)** を参照。

### 2. モデル認証（Anthropic 使用時）

```bash
claude setup-token       # Anthropic CLI で API キーを取得
openclaw models auth login --provider anthropic
  # 「Anthropic setup-token」を選択
```

詳細は **[MODELS-SETUP.md](MODELS-SETUP.md)** を参照。

### 3. Slack 連携

1. Slack ワークスペースで App「OpenClaw」を作成
2. Socket Mode 有効化（xapp- / xoxb- トークン）
3. `~/.openclaw/openclaw.json` の `channels.slack` に設定
4. `openclaw gateway start`

詳細は **[CHANNELS.md](CHANNELS.md)** を参照。

### 4. 確認

```bash
openclaw status                    # サービス・セッション確認
openclaw dashboard                 # Web UI を開く
openclaw gateway logs --follow     # Gateway ログを表示
```

---

## 📚 深く学ぶ

- **設定の詳細**: [CONFIG-REFERENCE.md](CONFIG-REFERENCE.md)
- **トラブル時**: [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
- **セキュリティ**: [SECRETS-MANAGEMENT.md](SECRETS-MANAGEMENT.md)
- **定期実行**: [AUTOMATION.md](AUTOMATION.md)
- **日常運用**: [OPERATIONAL-GUIDE.md](OPERATIONAL-GUIDE.md)

---

## 📖 公式ドキュメント

- **OpenClaw 公式**: <https://openclaw.ai/>
- **ドキュメント**: <https://docs.openclaw.ai/>
- **GitHub**: <https://github.com/openclaw/openclaw>

---

## 🔗 関連プロジェクト

| リポジトリ | 内容 |
|-----------|------|
| [`openclaw_setup`](https://github.com/masauehr/openclaw_setup) | 導入・試行のログ（PROGRESS / INSTALL / CONFIG 等） |
| [`automated-data-collector`](https://github.com/masauehr/automated-data-collector) | 気象・経済・AI の自動データ収集 |
| `~/.openclaw/workspace/` | ローカルワークスペース（SOUL / IDENTITY / 補助スクリプト） |

---

**最後に更新**: 2026-09-01 10:04 JST  
**次のレビュー予定**: 設定変更 or OpenClaw バージョン更新時
