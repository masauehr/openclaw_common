# シークレット管理・セキュリティガイド

OpenClaw 運用時のトークン・API キー・パスワード管理と セキュリティベストプラクティス。

**最新更新**: 2026-09-01

---

## 🔐 シークレット一覧

このMac の OpenClaw 環境で取り扱うシークレット:

| シークレット | 用途 | 保存場所 | 形式 | リスク |
|-------------|------|---------|------|--------|
| **Gateway Token** | Gateway 認証 | `~/.openclaw/openclaw.json` | トークン（ランダム文字列） | 🔴 高（全アクセス） |
| **Anthropic API Key** | Claude API 呼び出し | `~/.anthropic/credentials.json` | setup-token | 🔴 高（課金対象） |
| **Slack App Token** | Slack Socket Mode 接続 | `~/.openclaw/openclaw.json` | `xapp-*` | 🟠 中（App レベル） |
| **Slack Bot Token** | Slack メッセージ送受信 | `~/.openclaw/openclaw.json` | `xoxb-*` | 🟠 中（Bot レベル） |
| **SearXNG Secret Key** | SearXNG 検索エンジン設定 | `~/searxng/settings.yml` | ランダム文字列 | 🟡 低（ローカルのみ） |

---

## 🛡️ セキュリティベストプラクティス

### 1. ファイルアクセス制限

```bash
# 重要ファイルのパーミッション設定
chmod 600 ~/.openclaw/openclaw.json       # owner read/write only
chmod 600 ~/.anthropic/credentials.json   # owner read/write only
chmod 600 ~/searxng/settings.yml          # owner read/write only
```

### 2. Git から除外

```bash
# ~/.gitignore に追加
~/.openclaw/openclaw.json
~/.anthropic/credentials.json
~/searxng/settings.yml
```

**確認**:

```bash
git config --global core.excludesfile ~/.gitignore
git check-ignore ~/.openclaw/openclaw.json  # nothing output = OK
```

### 3. バックアップ暗号化（推奨）

```bash
# バックアップ作成時は暗号化
tar czf - ~/.openclaw/ | openssl enc -aes-256-cbc -out backup-openclaw.tar.gz.enc

# 復元時は復号
openssl enc -aes-256-cbc -in backup-openclaw.tar.gz.enc | tar xz
```

### 4. 環境変数での管理（将来の推奨）

```bash
# 現在: 平文ファイル
# 将来: 環境変数参照

# ~/.openclaw/openclaw.json
{
  "gateway": {
    "auth": {
      "token": "${OPENCLAW_GATEWAY_TOKEN}"
    }
  }
}

# ~/.bashrc / ~/.zshrc
export OPENCLAW_GATEWAY_TOKEN="***"
```

---

## 🔄 トークン・キー ローテーション

### Gateway Token ローテーション

**頻度**: 月 1 回推奨（or 定期イベント後）

```bash
# 新トークン生成
NEW_TOKEN=$(openssl rand -hex 32)

# 設定変更
openclaw config set gateway.auth.token "$NEW_TOKEN"

# Gateway 再起動
openclaw gateway restart

# 確認（マスク表示）
openclaw config get gateway.auth.token | head -c 20
```

### Anthropic API キー ローテーション

**頻度**: 3-6ヶ月 or 漏洩検知時

```bash
# Claude CLI で新キー生成
claude setup-token

# OpenClaw に登録
openclaw models auth logout --provider anthropic
openclaw models auth login --provider anthropic

# 確認
openclaw models list --provider anthropic
```

### Slack Token ローテーション

**頻度**: 年 1 回 or 権限変更後

#### App Token (`xapp-*`)

```bash
# https://api.slack.com/apps/YOUR_APP_ID/socket-mode
# 「Generate Token」を新規生成
# 新しい xapp-* をコピー

openclaw config set channels.slack.appToken "xapp-新しい値"
openclaw gateway restart
```

#### Bot Token (`xoxb-*`)

```bash
# https://api.slack.com/apps/YOUR_APP_ID/oauth
# 「Reinstall to Workspace」で再生成
# 新しい xoxb-* をコピー

openclaw config set channels.slack.botToken "xoxb-新しい値"
openclaw gateway restart
```

---

## 🚨 漏洩検知・対応

### 症状: Gateway Token が公開された（GitHub など）

```bash
# 1. 即座に新トークン生成・設定
NEW_TOKEN=$(openssl rand -hex 32)
openclaw config set gateway.auth.token "$NEW_TOKEN"
openclaw gateway restart

# 2. 古いトークンが使用されていないか確認
openclaw gateway logs | grep "auth failed" | wc -l

# 3. Git 履歴から削除（公開前なら不要）
# github から削除リクエスト
```

### 症状: API キー がログに出現した

```bash
# 1. ログをチェック
grep -r "sk-ant" ~/Library/Logs/openclaw/ | wc -l

# 2. API キーをローテーション
claude setup-token
openclaw models auth login --provider anthropic

# 3. ログから痕跡削除
rm ~/Library/Logs/openclaw/gateway.log*
```

### 症状: Slack Token が漏洩

```bash
# 1. 新トークン生成（Slack Web から）
# https://api.slack.com/apps/YOUR_APP_ID

# 2. 古いトークンをリボーク
# (Slack は自動的に無効化される傾向)

# 3. OpenClaw に設定
openclaw config set channels.slack.botToken "xoxb-新"
openclaw gateway restart
```

---

## 📋 設定ファイル詳細

### `~/.openclaw/openclaw.json`

```json5
{
  "gateway": {
    "auth": {
      "mode": "token",
      "token": "***"  // ← 平文保存（⚠️ 注意）
    }
  },
  "channels": {
    "slack": {
      "appToken": "xapp-***",   // ← 平文保存
      "botToken": "xoxb-***"    // ← 平文保存
    }
  }
}
```

**取り扱い**:
- ✅ 権限: `600`（owner read/write only）
- ✅ 共有禁止（GitHub コミット禁止）
- ✅ バックアップ時は暗号化
- ✅ `.gitignore` に追加

### `~/.anthropic/credentials.json`

```json
{
  "default": {
    "api_key": "sk-ant-***"  // ← 平文保存
  }
}
```

**取り扱い**:
- ✅ 権限: `600`
- ✅ 読み込み禁止（OpenClaw が内部で読む）
- ✅ ローカルマシンのみ保存

### `~/searxng/settings.yml`

```yaml
server:
  secret_key: "***"  # ← 平文保存
```

**取り扱い**:
- ✅ 権限: `644`（通常）
- ✅ ローカルのみ（リモートアクセスなし）
- ✅ 変更は慎重に

---

## 🔒 アクセス制限

### ローカルマシンのセキュリティ

| 対策 | 実装状況 |
|------|--------|
| **ユーザーアカウント** | macOS ログインユーザーのみアクセス可 |
| **ファイルパーミッション** | `600` で owner read/write only |
| **Firewall** | Gateway は localhost のみバインド |
| **SSH** | 外部 SSH アクセスなし |
| **Git** | クレデンシャル非コミット |

### ネットワークセキュリティ

| コンポーネント | バインド | アクセス |
|-------------|---------|--------|
| **Gateway** | `localhost:18789` | 🟢 ローカルのみ |
| **SearXNG** | `localhost:18899` | 🟢 ローカルのみ |
| **Slack** | インターネット | 🟠 公開（Slack 経由） |
| **Anthropic API** | インターネット | 🟠 公開（API 経由） |

**推奨**:
- Gateway をリモート公開する場合は **Tailscale** 等の VPN 経由
- Slack / Anthropic はインターネット接続必須（VPN 不要）

---

## 🔍 シークレット検出

### 設定ファイル内の平文確認

```bash
# Token が含まれているファイルを検出
grep -r "xapp-\|xoxb-\|sk-ant-" ~/.openclaw/

# 出力がない = OK（マスクされている）
# 出力がある = リスク（アクセス制限確認）
```

### ログにシークレットが出力されないか確認

```bash
# ログスキャン
grep -r "sk-ant-\|xapp-\|xoxb-" ~/Library/Logs/openclaw/

# 見つかった場合
rm ~/Library/Logs/openclaw/gateway.log*  # ログ削除
openclaw gateway restart  # 再起動して新ログ作成
```

---

## 🛠️ SecretRef（将来対応）

OpenClaw の将来バージョンで **SecretRef** が導入される予定:

```json
{
  "gateway": {
    "auth": {
      "token": {
        "source": "env",
        "id": "OPENCLAW_GATEWAY_TOKEN"
      }
    }
  }
}
```

**利点**:
- ✅ ファイルに平文保存しない
- ✅ 環境変数から読み込み
- ✅ CI/CD パイプライン対応

**現在**: 手動設定で対応（上記の "環境変数での管理" 参照）

---

## 📝 監査・ログ

### シークレットアクセスログ

```bash
# Gateway ログでトークン検証を確認
openclaw gateway logs | grep -i "auth\|token"

# 正常なアクセス
[INFO] Socket Mode authenticated
[INFO] Anthropic API connected
```

### 定期監査

**毎月末**:

```bash
# 1. ファイルパーミッション確認
stat -f "%OLp" ~/.openclaw/openclaw.json  # 600 か確認

# 2. Git 履歴をスキャン
git log -p | grep -i "sk-ant\|xapp" | wc -l  # 0 であること

# 3. ログをチェック
grep -c "sk-ant\|token.*:" ~/Library/Logs/openclaw/gateway.log

# 4. アクセスを確認
launchctl list | grep openclaw  # プロセス稼働確認
```

---

## 🚫 禁止事項

| 禁止 | 理由 |
|------|------|
| Token をチャットに貼る | 第三者が読める |
| Token を git commit | 公開リポジトリに永久保存 |
| Token を平文メールで送信 | 通信経路で盗聴可能 |
| ローカルバックアップを暗号化なし | 盗難時に全アクセス奪われる |
| 同じ Token を複数環境で使用 | 1 つ漏洩したら全滅 |

---

## ✅ チェックリスト（セットアップ時）

- [ ] `~/.openclaw/openclaw.json` のパーミッション = `600`
- [ ] `~/.anthropic/credentials.json` のパーミッション = `600`
- [ ] `.gitignore` に `openclaw.json` / `credentials.json` を追加
- [ ] Git 履歴にトークンが含まれていないか確認
- [ ] Gateway ログでトークン平文が出力されていないか確認
- [ ] Slack Bot が online か確認
- [ ] Anthropic API が接続できるか確認

---

## 📞 セキュリティ問題報告

OpenClaw セキュリティ脆弱性を発見した場合:

- **OpenClaw 本体**: <https://github.com/openclaw/openclaw/security>
- **このプロジェクト**: GitHub Issues（Private モード）
- **Slack**: ワークスペース管理者に報告

**漏洩トークン対応**:
- Anthropic: API キーを再生成（<https://console.anthropic.com>）
- Slack: トークンをリボーク（<https://api.slack.com/apps>）
- Gateway: `openclaw config set gateway.auth.token` で新トークン生成

---

## 📖 関連ドキュメント

- [CONFIG-REFERENCE.md](CONFIG-REFERENCE.md) — 設定ファイル詳細
- [OPERATIONAL-GUIDE.md](OPERATIONAL-GUIDE.md) — 日常運用
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) — トラブル診断
- 公式: <https://docs.openclaw.ai/security>

**最後に更新**: 2026-09-01  
**バージョン**: OpenClaw `2026.7.1-2`
