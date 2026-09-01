# 定期実行（Cron）ガイド

OpenClaw の cron ジョブ機能。定期的に自動実行タスク（気象ダイジェスト・経済情報など）を設定・運用。

**現在の運用**: 3ジョブ登録済み（朝 09:10-09:16 実行）

---

## 📅 現在のジョブ一覧（2026-09-01）

| Job ID | Name | Schedule | 内容 | モデル | デリバリ |
|--------|------|----------|------|--------|---------|
| `445881d3…` | `weather-jp-am` | 毎日 09:10 | 日本の気象・防災ダイジェスト | haiku + sonnet | Slack DM |
| `b18e0f97…` | `econ-jp-am` | 毎日 09:13 | 日本株・為替・経済指標 | haiku + sonnet | Slack DM |
| `db8a0d0b…` | `ai-weekly-digest` | 月曜 09:16 | 週次 AI 動向ダイジェスト | haiku + sonnet | Slack DM |

すべて `anthropic/claude-haiku-4-5` + fallback `anthropic/claude-sonnet-5`。配信先: 自分の Slack DM。

---

## 🚀 Cron ジョブ作成

### 基本コマンド

```bash
openclaw cron add \
  --name "weather-jp-am" \
  --schedule "10 9 * * *" \
  --message "[気象ダイジェスト生成]" \
  --channel slack \
  --to slack:U0XXXXXXXXX \
  --model anthropic/claude-haiku-4-5 \
  --fallbacks anthropic/claude-sonnet-5 \
  --timeout-seconds 300 \
  --announce
```

### パラメータ説明

| パラメータ | 説明 |
|-----------|------|
| `--name` | ジョブ名（識別用） |
| `--schedule` | cron 式（`分 時 日 月 曜日`） |
| `--message` | エージェントへのプロンプト |
| `--channel` | 配信先チャネル（`slack` / `telegram` など） |
| `--to` | 配信先詳細（ユーザーID など） |
| `--model` | 使用モデル |
| `--fallbacks` | フォールバックモデル（複数可） |
| `--timeout-seconds` | タイムアウト時間 |
| `--announce` | 完了時に配信 |
| `--tz` | タイムゾーン（`Asia/Tokyo` など） |

### Cron 式リファレンス

```
分(0-59) 時(0-23) 日(1-31) 月(1-12) 曜日(0-6:日-土)
```

| 例 | 説明 |
|----|------|
| `0 9 * * *` | 毎日 09:00 |
| `10 9 * * *` | 毎日 09:10 |
| `0 9 * * 1` | 月曜 09:00 |
| `0 */6 * * *` | 6時間ごと（00:00 / 06:00 / 12:00 / 18:00） |
| `30 7-18 * * *` | 毎日 07:30 ～ 18:30 |

**注**: cron は **UTC でなく、指定タイムゾーン（JST）で解釈**。

---

## 📊 Cron ジョブ管理

### リスト表示

```bash
# すべてのジョブ表示
openclaw cron list

# 出力例
ID         Name              Schedule      Status   Model
445881d3…  weather-jp-am     10 9 * * *    enabled  haiku
b18e0f97…  econ-jp-am        13 9 * * *    enabled  haiku
db8a0d0b…  ai-weekly-digest  16 9 * * 1    enabled  haiku
```

### 詳細確認

```bash
openclaw cron get <job_id>

# 実行履歴
openclaw cron runs <job_id>
# or
openclaw cron runs <job_id> --recent 5  # 最新 5 件

# 1 件の実行詳細
openclaw cron runs <job_id> --id <run_id>
```

### ジョブ編集

```bash
openclaw cron edit <job_id>
# エディタが開く

# or CLI で直接パッチ
openclaw cron update <job_id> \
  --model anthropic/claude-sonnet-5 \
  --timeout-seconds 600
```

### ジョブ削除

```bash
openclaw cron remove <job_id>

# 確認付き
openclaw cron remove <job_id> --confirm
```

### 手動実行（テスト）

```bash
# 即座に実行（スケジュール無視）
openclaw cron run <job_id> --wait

# 実行状況を監視
openclaw cron run <job_id> --wait --follow
```

---

## 📋 実装例

### 例 1: 毎日朝 09:10 の気象ダイジェスト

```bash
openclaw cron add \
  --name "weather-jp-am" \
  --schedule "10 9 * * *" \
  --message "日本の気象・防災情報をまとめてください。気温ランキング、降水量、警報などを含めて。" \
  --channel slack \
  --to slack:U0XXXXXXXXX \
  --model anthropic/claude-haiku-4-5 \
  --fallbacks anthropic/claude-sonnet-5 \
  --timeout-seconds 300 \
  --announce \
  --tz Asia/Tokyo
```

### 例 2: 毎週月曜 09:16 の AI 動向ダイジェスト

```bash
openclaw cron add \
  --name "ai-weekly-digest" \
  --schedule "16 9 * * 1" \
  --message "先週の AI 業界動向をまとめてください。新しいモデルリリース、企業ニュース、重要な論文などを含めて。" \
  --channel slack \
  --to slack:U0XXXXXXXXX \
  --model anthropic/claude-haiku-4-5 \
  --fallbacks anthropic/claude-sonnet-5 \
  --timeout-seconds 600 \
  --announce \
  --tz Asia/Tokyo
```

### 例 3: 毎日 18:00 の経済レポート

```bash
openclaw cron add \
  --name "econ-jp-pm" \
  --schedule "0 18 * * *" \
  --message "日本と世界の経済ニュースをまとめてください。日経平均、為替、金利、重要経済指標などを含めて。" \
  --channel slack \
  --to slack:U0XXXXXXXXX \
  --model anthropic/claude-haiku-4-5 \
  --fallbacks anthropic/claude-sonnet-5 \
  --timeout-seconds 300 \
  --announce \
  --tz Asia/Tokyo
```

---

## ⚙️ 設定（`openclaw.json`）

### グローバル cron 設定

```json
{
  "cron": {
    "enabled": true,
    "maxConcurrentRuns": 8,
    "sessionRetention": "24h",
    "runLog": {
      "maxBytes": "2mb",
      "keepLines": 2000
    }
  }
}
```

| パラメータ | 説明 | デフォルト |
|-----------|------|----------|
| **enabled** | cron 機能有効化 | `true` |
| **maxConcurrentRuns** | 同時実行数上限 | 8 |
| **sessionRetention** | セッション保持期間 | `24h` |
| **runLog.maxBytes** | ログ保持容量 | `2mb` |
| **runLog.keepLines** | ログ保持行数 | `2000` |

### 有効化・無効化

```bash
# 有効化
openclaw config set cron.enabled true

# 無効化
openclaw config set cron.enabled false

# 最大同時実行数変更
openclaw config set cron.maxConcurrentRuns 16
```

---

## 🔍 トラブルシューティング

### ジョブが実行されない

```bash
# cron が有効か確認
openclaw config get cron.enabled

# スケジュール確認
openclaw cron get <job_id>

# ログを確認
openclaw gateway logs --follow
```

### ジョブのタイムアウト

```bash
# タイムアウト時間を増やす
openclaw cron update <job_id> --timeout-seconds 600

# または削除して再作成
openclaw cron remove <job_id>
openclaw cron add ... --timeout-seconds 600
```

### 配信が失敗する

```bash
# 配信先チャネルが online か確認
openclaw status

# チャネルの health check
openclaw gateway logs --follow | grep slack

# Slack 接続を再度テスト
openclaw cron run <job_id> --wait
```

### モデルが存在しないエラー

```bash
# 利用可能なモデル確認
openclaw models list

# ジョブで指定したモデルが存在するか確認
openclaw models list --provider anthropic

# 登録されていない場合は先に認証
openclaw models auth login --provider anthropic
```

### 実行ログが保存されない

```bash
# ログ設定確認
openclaw config get cron.runLog

# ログディレクトリを確認
ls -la ~/Library/Logs/openclaw/

# SQLite を直接確認
sqlite3 ~/.openclaw/state/openclaw.sqlite \
  "SELECT * FROM cron_runs ORDER BY created_at DESC LIMIT 5;"
```

---

## 📈 運用ベストプラクティス

### 実行時間の分散

複数ジョブが同時実行されないよう間隔を開ける:

```
09:10 気象ダイジェスト（気温ランキング exec あり、時間掛かる）
09:13 経済レポート（web_search）
09:16 AI 週次（web_search）
```

（他の自動実行との競合避けに 09:00-12:00 が空きを作った）

### タイムアウト設定

| ジョブ種類 | 内容 | 推奨タイムアウト |
|----------|------|-----------------|
| 気象 | exec + web_search | 300～600秒 |
| 経済 | web_search のみ | 180～300秒 |
| AI 週次 | web_search のみ | 300～600秒 |

### モデル・fallback 戦略

- **Primary**: `anthropic/claude-haiku-4-5`（コスト最適）
- **Fallback 1**: `anthropic/claude-sonnet-5`（高品質が必要な場合）
- **Fallback 2**: `ollama/qwen3.6:35b-mlx`（課金枠完全切れ時）

ローカル Ollama は遅いので、fallback チェーンの最後に。

### モニタリング

```bash
# 週 1 回チェック
openclaw cron runs <job_id> --recent 7

# 実行時間が増加していないか
# モデルが切り替わっていないか（fallback 動作）
# エラー率が高くないか
```

---

## 📡 Webhook / 外部システム連携

cron ジョブの完了時に外部 webhook を呼び出す:

```bash
openclaw cron add \
  ... \
  --delivery-webhook "https://example.com/webhook" \
  --delivery-mode webhook
```

Webhook 設定:

```json
{
  "delivery": {
    "mode": "webhook",
    "to": "https://example.com/webhook"
  }
}
```

---

## 📊 実行履歴分析

```bash
# 全ジョブの最新実行状況
for job in $(openclaw cron list --format json | jq -r '.[].id'); do
  echo "Job: $job"
  openclaw cron runs $job --recent 1
done

# SQLite 直接クエリ
sqlite3 ~/.openclaw/state/openclaw.sqlite << 'SQL'
SELECT 
  job_name,
  COUNT(*) as runs,
  AVG(duration_ms) as avg_duration_ms,
  SUM(CASE WHEN status = 'success' THEN 1 ELSE 0 END) as successes,
  SUM(CASE WHEN status = 'failed' THEN 1 ELSE 0 END) as failures
FROM cron_runs
WHERE created_at > datetime('now', '-7 days')
GROUP BY job_name;
SQL
```

---

## 📖 関連ドキュメント

- [CONFIG-REFERENCE.md](CONFIG-REFERENCE.md) — cron 設定詳細
- [OPERATIONAL-GUIDE.md](OPERATIONAL-GUIDE.md) — 日常運用
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) — トラブル診断
- 公式: <https://docs.openclaw.ai/automation/cron-jobs>

**最後に更新**: 2026-09-01
