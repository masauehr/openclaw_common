# トラブルシューティングガイド

OpenClaw で発生する一般的な問題と解決方法。

**最新更新**: 2026-09-01  
**バージョン**: OpenClaw `2026.7.1-2`

---

## 🔍 診断フロー

```
問題発生
   ↓
[ ステップ 1 ] Gateway が起動しているか？
   ├─ NO  → Gateway 起動トラブル
   └─ YES → ステップ 2
   
[ ステップ 2 ] Slack は online か？
   ├─ NO  → チャネル接続トラブル
   └─ YES → ステップ 3

[ ステップ 3 ] メッセージは返ってくるか？
   ├─ NO  → モデル・ツールトラブル
   └─ YES → パフォーマンストラブル
```

---

## 🚀 Gateway 起動トラブル

### 症状: `openclaw status` が接続失敗

```
Error: Could not connect to Gateway at localhost:18789
```

### 診断

```bash
# 1. launchd サービス状態確認
launchctl list | grep openclaw
# 出力がなければ未登録

# 2. ログ確認
cat ~/Library/Logs/openclaw/gateway.log | tail -50

# 3. ポート確認
lsof -i :18789
# 何も出なければポートが空いている
```

### 解決策

#### A) サービスが登録されていない

```bash
# onboard を再実行
openclaw onboard --auth-choice skip

# or 手動登録
openclaw gateway install  # launchd 登録
launchctl start ai.openclaw.gateway
```

#### B) サービスは登録されているが起動していない

```bash
# 起動
launchctl start ai.openclaw.gateway

# 状態確認
launchctl list | grep openclaw

# ログ確認
openclaw gateway logs --follow
```

#### C) ポート競合

別プロセスが 18789 を使用:

```bash
# 使用中のプロセスを確認
lsof -i :18789

# 競合プロセスを終了 or ポート変更
# (ポート変更の場合)
openclaw config set gateway.port 18790
launchctl restart ai.openclaw.gateway
```

#### D) 設定エラー

```bash
# 設定をバリデート
openclaw config validate

# エラーが出たら修復
openclaw doctor --fix
```

### チェックリスト

- [ ] `launchctl list | grep openclaw` で表示されるか？
- [ ] `~/Library/Logs/openclaw/gateway.log` にエラーがないか？
- [ ] ポート 18789 が空いているか？（`lsof -i :18789`）
- [ ] `~/.openclaw/openclaw.json` が有効な JSON か？（`jq empty`）

---

## 💬 Slack 接続トラブル

### 症状: Bot が offline / SETUP 状態

```bash
openclaw status
# Slack: status SETUP (not healthy)
```

### 診断

```bash
# 1. ログ確認
openclaw gateway logs --follow | grep -i slack

# 2. トークン確認（内容は見ない）
grep "appToken\|botToken" ~/.openclaw/openclaw.json | head -c 50

# 3. Slack App の状態確認
# https://api.slack.com/apps にアクセス
# → Socket Mode は有効か？
# → Scopes は設定されているか？
```

### 解決策

#### A) Socket Mode トークン期限切れ

```bash
# https://api.slack.com/apps/YOUR_APP_ID/socket-mode
# から新しい xapp-* トークンを生成

openclaw config set channels.slack.appToken "xapp-新しい値"
openclaw gateway restart
```

#### B) Bot Token 無効

```bash
# https://api.slack.com/apps/YOUR_APP_ID/oauth
# から xoxb-* トークンを再生成

openclaw config set channels.slack.botToken "xoxb-新しい値"
openclaw gateway restart
```

#### C) Scopes 不足

```bash
# https://api.slack.com/apps/YOUR_APP_ID/oauth
# Socket Mode Scopes に以下を追加:
# - chat:write
# - channels:read
# - groups:read
# - im:read
# - message.channels
# - message.groups
# - message.im

# 追加後、トークンを再生成
# OpenClaw に設定し直す
```

#### D) ワークスペース削除 / App 削除

```bash
# Slack App が削除されていないか確認
# https://api.slack.com/apps

# 削除されていれば再作成
# [CHANNELS.md](CHANNELS.md) を参照
```

### チェックリスト

- [ ] `openclaw config get channels.slack.appToken` は `xapp-` で始まるか？
- [ ] `openclaw config get channels.slack.botToken` は `xoxb-` で始まるか？
- [ ] Socket Mode は Slack App で有効か？
- [ ] Bot の Scopes は最低限設定されているか？
- [ ] ワークスペースは存在するか？（`openclaw-test`）

---

## 📨 メッセージ返信がない

### 症状

```
user: 今日の天気は？
(5分待っても返信なし)
```

### 診断

```bash
# 1. セッション確認
openclaw sessions list
# セッションが作成されているか？

# 2. モデル確認
openclaw models list
# anthropic/claude-haiku-4-5 は存在するか？

# 3. Anthropic 認証確認
openclaw models list --provider anthropic

# 4. ログ確認
openclaw gateway logs --follow
# エラーメッセージがないか？
```

### 解決策

#### A) セッション肥大（遅延）

トークン数 > 80,000:

```bash
openclaw sessions get <session-key> --tokens
# 出力が大きければ肥大

# 圧縮
openclaw sessions compact <session-key> --max-lines 500
```

再度テスト。

#### B) モデルが見つからない

```bash
openclaw models list --provider anthropic
# 空の場合は認証が必要

openclaw models auth login --provider anthropic
# Anthropic setup-token を選択
```

#### C) Anthropic API エラー

```bash
# ログから詳細を確認
openclaw gateway logs --follow | grep -i anthropic

# 可能性:
# - API キー無効
# - レート制限
# - クォータ切れ
```

**解決**:

- API キー再生成: `claude setup-token`
- クォータ切れ: <https://console.anthropic.com/account/billing/overview> を確認

#### D) Tool 実行エラー

```bash
# ツール（web_search / exec）のエラー

# SearXNG が起動しているか
curl http://127.0.0.1:18899

# Python が実行可能か
python3 --version

# 補助スクリプトが存在するか
ls -la ~/.openclaw/workspace/bin/jma_rank.py
```

### チェックリスト

- [ ] `openclaw sessions list` にセッションがあるか？
- [ ] セッショントークン数は 80k 未満か？
- [ ] `openclaw models list --provider anthropic` に claude-haiku が出ているか？
- [ ] `openclaw gateway logs --follow` にエラーがないか？

---

## ⏱️ レスポンスが遅い（3分超）

### 診断

```bash
# 1. セッション肥大確認
openclaw sessions get <session-key> --tokens
# 50k+ = 肥大の可能性

# 2. モデル確認
openclaw config get agents.defaults.model.primary
# sonnet/opus だと遅い可能性

# 3. SearXNG の遅延確認
time curl -s http://127.0.0.1:18899 > /dev/null
# 5秒超 = SearXNG が遅い
```

### 解決策

#### A) セッション圧縮

```bash
openclaw sessions compact <session-key> --max-lines 300
```

#### B) モデルを haiku に変更

```bash
openclaw config set agents.defaults.model.primary anthropic/claude-haiku-4-5
```

#### C) Web 検索を自動から手動に

```bash
# ツール profile を制限
openclaw config set tools.profile basic
# or
openclaw config set tools.web.search.enabled false
```

#### D) SearXNG が遅い場合

```bash
# 再起動
launchctl restart local.searxng

# or web 検索そのものを無効化
openclaw config set tools.web.search.enabled false
```

---

## ⚙️ Cron ジョブ トラブル

### 症状: ジョブが実行されない

```bash
openclaw cron list
# 時刻が来ているのに実行されない
```

### 診断

```bash
# 1. cron が有効か
openclaw config get cron.enabled

# 2. ジョブスケジュール確認
openclaw cron get <job_id>

# 3. 実行ログ確認
openclaw cron runs <job_id> --recent 3

# 4. Gateway ログ確認
openclaw gateway logs | grep cron
```

### 解決策

#### A) cron が無効

```bash
openclaw config set cron.enabled true
openclaw gateway restart
```

#### B) スケジュール間違い

```bash
# 例: JST で 09:10 のつもりが UTC で解釈されている
# cron 式を確認

openclaw cron get <job_id> | grep schedule

# JST で指定されているか確認
openclaw cron get <job_id> | grep tz
```

修正:

```bash
openclaw cron update <job_id> --tz Asia/Tokyo
```

#### C) モデルが存在しない

```bash
openclaw models list --provider anthropic

# 指定モデルが出ているか確認
openclaw cron get <job_id> | grep model
```

#### D) タイムアウト

```bash
# ジョブの実行時間が長い
openclaw cron runs <job_id> --recent 1 --detailed | grep duration

# タイムアウトを延長
openclaw cron update <job_id> --timeout-seconds 600
```

---

## 🔧 モデル・認証トラブル

### 症状: 認証エラー

```
Error: anthropic/claude-haiku-4-5 not found
Error: Anthropic API key invalid
```

### 解決策

```bash
# 1. 認証再実行
openclaw models auth logout --provider anthropic
openclaw models auth login --provider anthropic

# 2. API キー再生成
claude setup-token

# 3. 状態確認
openclaw models list --provider anthropic
```

### 症状: Fallback が機能しない

```bash
# モデルが見つからない → fallback を試す → でも失敗
```

### 解決策

```bash
# fallback が正しく配列か確認
openclaw config get agents.defaults.model.fallbacks

# 修正例
openclaw config set agents.defaults.model.fallbacks '["anthropic/claude-sonnet-5"]'
```

---

## 📊 SQLite / DB トラブル

### 症状: DB がロック・破損

```
Error: database is locked
Error: database disk image is malformed
```

### 診断

```bash
# DB ファイルサイズ確認
ls -lh ~/.openclaw/state/openclaw.sqlite*

# DB チェック
sqlite3 ~/.openclaw/state/openclaw.sqlite "PRAGMA integrity_check;"
```

### 解決策

```bash
# Gateway を停止
launchctl stop ai.openclaw.gateway

# DB ファイルを退避
mv ~/.openclaw/state/openclaw.sqlite ~/.openclaw/state/openclaw.sqlite.bak

# Gateway を再起動（新 DB が自動生成）
launchctl start ai.openclaw.gateway

# 古いデータが必要な場合は復元
# (通常は必要ない)
```

---

## 📈 性能トラブル

### 症状: メモリ使用量が増加

```bash
# Gateway プロセスのメモリ確認
ps aux | grep "[o]penclaw.*gateway" | awk '{print $6}'  # KB 単位
```

**対策**:

```bash
# セッション肥大の確認
openclaw sessions list --detailed | head -20

# 不要なセッション削除
openclaw sessions remove <old-session-key>

# Gateway 再起動
openclaw gateway restart
```

### 症状: ディスク容量が足りない

```bash
du -sh ~/.openclaw/
du -sh ~/Library/Logs/openclaw/
du -sh ~/searxng/
```

**対策**:

```bash
# ログ削除
rm ~/Library/Logs/openclaw/*.log.*

# 古いセッションデータ削除
openclaw sessions list | grep "7 days ago" | while read key; do
  openclaw sessions remove "$key"
done

# SQLite 最適化
sqlite3 ~/.openclaw/state/openclaw.sqlite "VACUUM;"
```

---

## 🆘 最終手段

### 1. Gateway 完全リセット（全セッション削除）

```bash
# ⚠️ すべてのセッション履歴が削除される

launchctl stop ai.openclaw.gateway
rm -rf ~/.openclaw/agents/main/sessions/*
launchctl start ai.openclaw.gateway
```

### 2. OpenClaw 再インストール

```bash
# 完全アンインストール
npm uninstall -g openclaw

# 再インストール
npm install -g openclaw@2026.7.1-2

# onboard 再実行
openclaw onboard
```

### 3. サポート・ログ出力

```bash
# 診断情報をファイルに出力
openclaw doctor > ~/openclaw-doctor.txt

# ログを zip で出力
tar -czf ~/openclaw-logs.tar.gz ~/Library/Logs/openclaw/

# GitHub Issue を報告
# https://github.com/openclaw/openclaw/issues
```

---

## 📖 関連ドキュメント

- [OPERATIONAL-GUIDE.md](OPERATIONAL-GUIDE.md) — 日常運用
- [CONFIG-REFERENCE.md](CONFIG-REFERENCE.md) — 設定変更
- [SECRETS-MANAGEMENT.md](SECRETS-MANAGEMENT.md) — シークレット管理
- 公式: <https://docs.openclaw.ai/troubleshooting>

---

**最後に更新**: 2026-09-01  
**バージョン**: OpenClaw `2026.7.1-2`
