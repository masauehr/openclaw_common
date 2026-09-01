# 日常運用・メンテナンスガイド

OpenClaw の日々の運用・メンテナンスと定期チェック項目。

**対象**: このMac での常時運用

---

## 📅 日次チェック

毎日 `openclaw status` を確認:

```bash
openclaw status
```

### チェック項目

| 項目 | 期待値 | 異常時 |
|------|--------|--------|
| **Gateway** | `running` | `opensraw gateway start` |
| **Sessions** | 前日比±5 | セッション肥大チェック |
| **Slack** | `healthy` / `SETUP` | トークン再確認 |
| **Model** | `anthropic/claude-haiku-4-5` | 意図しない切り替えチェック |
| **Cron jobs** | 次実行時刻正しい | スケジュール確認 |

---

## 🔄 定期メンテナンス

### 毎週（日曜夜）

#### 1. セッション肥大チェック

```bash
openclaw sessions list --detailed
```

出力例:

```
Key                              Tokens    Age
agent:main:slack:direct:…        87,000    7 days ← 大きい
agent:main:cron:…               10,000    1 day
```

**87,000 トークン超** = 肥大状態

#### 2. セッション圧縮（必要に応じて）

```bash
# 肥大セッションを確認
openclaw sessions list | grep "87.*days"

# 圧縮実行
openclaw sessions compact <session-key> --max-lines 500

# 圧縮前後を比較
openclaw sessions get <session-key> --tokens
```

#### 3. メモリファイル整理

```bash
# 日次メモリを確認
ls -lh ~/.openclaw/workspace/memory/

# 古いファイルを summary に統合
# MEMORY.md を更新
```

#### 4. Cron ジョブ実行履歴確認

```bash
# 各ジョブの実行状況を確認
for job in $(openclaw cron list --format json | jq -r '.[].id'); do
  echo "=== Job: $job ==="
  openclaw cron runs $job --recent 7
done
```

成功率・エラー率をチェック。

#### 5. ログローテーション確認

```bash
# Gateway ログサイズ
ls -lh ~/Library/Logs/openclaw/gateway.log*

# SearXNG ログ
ls -lh ~/Library/Logs/openclaw/searxng.log*

# 古いログを削除（7日以上前）
find ~/Library/Logs/openclaw/ -mtime +7 -delete
```

### 毎月（1 日）

#### 1. 設定ファイル確認

```bash
# openclaw.json の差分確認
ls -la ~/.openclaw/openclaw.json*

# バージョン確認
openclaw --version
```

#### 2. モデル認証の有効性確認

```bash
# Anthropic 認証状態
openclaw models list --provider anthropic

# 試験的に haiku でリクエスト
echo "Hello" | openclaw models run anthropic/claude-haiku-4-5
```

#### 3. ストレージ容量確認

```bash
du -sh ~/.openclaw/
du -sh ~/Library/Logs/openclaw/

# SQLite サイズ確認
ls -lh ~/.openclaw/state/openclaw.sqlite*
```

#### 4. Slack 接続確認

```bash
# Slack Bot が online か
openclaw status | grep -i slack

# ワークスペース情報
openclaw config get channels.slack.botToken | head -c 20
```

### 四半期（3ヶ月）

#### 1. バージョンアップ確認

```bash
npm outdated -g openclaw

# 最新版の確認
npm info openclaw | grep latest
```

#### 2. 依存パッケージ更新

```bash
npm update -g openclaw

# インストール後の確認
openclaw --version
```

#### 3. セキュリティパッチ確認

```bash
npm audit -g openclaw
# 脆弱性があれば対応
```

#### 4. バックアップ確認

```bash
# ワークスペース状態確認
git -C ~/.openclaw/workspace status

# 必要に応じて commit
git -C ~/.openclaw/workspace add .
git -C ~/.openclaw/workspace commit -m "quarterly backup: $(date +%Y-%m-%d)"
```

---

## 📊 モニタリングコマンド集

### セッション健全性監視

```bash
# セッション一覧（詳細）
openclaw sessions list --detailed --format json | jq '.[].tokens' | sort -nr | head -10

# トークン使用量が多いセッション抽出
openclaw sessions list --detailed | awk '$2 > 50000 { print }'
```

### Cron 実行パフォーマンス

```bash
# SQLite から統計取得
sqlite3 ~/.openclaw/state/openclaw.sqlite << 'SQL'
SELECT 
  job_name,
  COUNT(*) as total_runs,
  SUM(CASE WHEN status='success' THEN 1 ELSE 0 END) as success_count,
  ROUND(AVG(duration_ms)/1000) as avg_duration_sec,
  MIN(created_at) as first_run,
  MAX(created_at) as last_run
FROM cron_runs
GROUP BY job_name
ORDER BY last_run DESC;
SQL
```

### Gateway メモリ使用量

```bash
# Gateway プロセス確認
ps aux | grep "[o]penclaw.*gateway"

# メモリ使用量（RSS）
ps aux | grep openclaw | awk '{print $6}'  # KB単位
```

### ネットワーク接続確認

```bash
# Anthropic API への接続確認
curl -s https://api.anthropic.com/v1/models | head -c 100

# SearXNG への接続確認
curl -s http://127.0.0.1:18899 | grep -q "SearXNG" && echo "SearXNG OK"
```

---

## 🔧 トラブル対応パターン

### パターン 1: Gateway が応答しない

```bash
# 状態確認
launchctl list | grep openclaw

# ログ確認
openclaw gateway logs --follow &

# 再起動
openclaw gateway restart

# 再度状態確認
openclaw status
```

### パターン 2: Slack メッセージが来ない

```bash
# Bot は online か
openclaw status | grep slack

# Socket Mode トークンは有効か
grep -o 'xapp-[^"]*' ~/.openclaw/openclaw.json | head -c 20

# 再接続
openclaw config set channels.slack.enabled false
sleep 5
openclaw config set channels.slack.enabled true
```

### パターン 3: セッションが遅い（3分超）

```bash
# セッション肥大を確認
openclaw sessions get <session-key> --tokens

# 圧縮実行
openclaw sessions compact <session-key> --max-lines 300

# 再度テスト
# (Slack で簡単な質問)
```

### パターン 4: Cron ジョブが失敗する

```bash
# ジョブ詳細確認
openclaw cron get <job_id>

# 最新実行ログ
openclaw cron runs <job_id> --recent 1 --detailed

# モデルが存在するか確認
openclaw models list --provider anthropic

# 手動テスト実行
openclaw cron run <job_id> --wait
```

---

## 📈 性能チューニング

### 対話レスポンス時間短縮

1. **セッション圧縮** - 肥大防止

```bash
openclaw sessions compact <session-key> --max-lines 500
```

2. **モデルをハイアウト（haiku）に統一**

```bash
openclaw config set agents.defaults.model.primary anthropic/claude-haiku-4-5
```

3. **Web 検索を自動化から手動に**

```bash
openclaw config set tools.web.search.autoSearch false
```

### Cron ジョブ実行時間短縮

1. **タイムアウトを調整**

```bash
# 現在
openclaw cron get <job_id> | grep timeout

# 短縮（300秒に）
openclaw cron update <job_id> --timeout-seconds 300
```

2. **exec を最小化**

気象スクリプト（`jma_rank.py`）は必要なとき以外非実行。

3. **Web 検索キャッシング**

```bash
openclaw config set tools.web.cache.enabled true
openclaw config set tools.web.cache.ttlSeconds 3600
```

---

## 📝 ログ管理

### ログ場所

```
~/Library/Logs/openclaw/
├── gateway.log              # Gateway 主要ログ
├── gateway.log.1            # ローテーション
└── ...
```

### ログフォーマット

```
2026-09-01T10:15:59.123Z [INFO] Gateway ready at localhost:18789
2026-09-01T10:16:00.456Z [DEBUG] Slack Socket Mode connected
2026-09-01T10:16:01.789Z [WARN] Session compaction needed for ...
2026-09-01T10:16:02.012Z [ERROR] Anthropic API timeout
```

### ログレベル設定

```bash
# 詳細ログ有効化
openclaw config set logging.level debug

# 本番環境では
openclaw config set logging.level info
```

### ログフォローとフィルタリング

```bash
# リアルタイム監視
openclaw gateway logs --follow

# キーワードで絞込
openclaw gateway logs --follow | grep -i error

# 特定の時間範囲
openclaw gateway logs --after "2026-09-01T10:00:00Z" --before "2026-09-01T11:00:00Z"
```

---

## 🎯 月次チェックリスト

- [ ] `openclaw status` で全体状態確認
- [ ] セッション肥大チェック（>80k トークン）
- [ ] Cron 実行履歴確認（エラー率）
- [ ] ログサイズ確認・古いログ削除
- [ ] Slack 接続確認（メッセージ送受信テスト）
- [ ] モデル認証確認（haiku で試験実行）
- [ ] ストレージ容量確認
- [ ] `MEMORY.md` 更新
- [ ] バージョン確認
- [ ] セキュリティパッチ確認（`npm audit`）

---

## 📞 トラブル時の連絡先・リソース

| リソース | URL |
|---------|-----|
| 公式ドキュメント | <https://docs.openclaw.ai/> |
| GitHub Issues | <https://github.com/openclaw/openclaw/issues> |
| Discord コミュニティ | <https://discord.gg/openclaw> |
| OpenClaw Setup ログ | <https://github.com/masauehr/openclaw_setup> |

---

## 📖 関連ドキュメント

- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) — トラブル診断フロー
- [CONFIG-REFERENCE.md](CONFIG-REFERENCE.md) — 設定変更
- [AUTOMATION.md](AUTOMATION.md) — cron 管理
- [SECRETS-MANAGEMENT.md](SECRETS-MANAGEMENT.md) — トークン管理

**最後に更新**: 2026-09-01
