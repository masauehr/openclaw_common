# チャットアプリ連携ガイド

Slack / Telegram / Discord など、メッセージングプラットフォームの設定・運用。

**現在の運用**: Slack（`openclaw-test` ワークスペース）

---

## 📲 チャンネル一覧

OpenClaw が対応するメッセージングプラットフォーム:

| プラットフォーム | サポート | 認証方式 | DM | グループ |
|---------|---------|--------|-----|---------|
| **Slack** | ✅ | Socket Mode | ✅ | ✅ |
| Telegram | ✅ | Bot Token | ✅ | ✅ |
| Discord | ✅ | Bot Token | ✅ | ✅ |
| WhatsApp | ✅ | QR コード | ✅ | ✅ |
| Signal | ✅ | Phone auth | ✅ | ❌ |
| iMessage | ✅ | Device auth | ✅ | ✅（グループ） |
| Google Chat | ✅ | OAuth | ❌ | ✅ |
| Mattermost | ✅ | Token | ✅ | ✅ |
| Matrix | ✅ | Token | ✅ | ✅ |

詳細: <https://docs.openclaw.ai/channels>

---

## 🔌 Slack セットアップ（現在の構成）

### 前提

- Slack ワークスペース作成済み（`openclaw-test`）
- ワークスペース管理者アクセス

### ステップ 1: App 作成

1. <https://api.slack.com/apps> にアクセス
2. **「Create New App」** → **「From scratch」**
3. App name: `OpenClaw`
4. Workspace: `openclaw-test` を選択
5. **「Create App」**

### ステップ 2: Socket Mode 有効化

左メニュー → **「Socket Mode」** → **「Enable Socket Mode」**

トークンが自動生成される:
- **App-Level Token**: `xapp-1-...`（Socket Mode 用）

### ステップ 3: Permissions 設定

左メニュー → **「OAuth & Permissions」**

#### Scopes（権限）

**Bot Token Scopes**:
```
chat:write              # メッセージ送信
groups:read             # プライベートチャネル読み込み
channels:read           # 公開チャネル読み込み
im:read                 # DM 読み込み
users:read              # ユーザー情報読み込み
app_mentions:read       # メンション検出
message.channels        # チャネルメッセージ読み込み
message.groups          # グループメッセージ読み込み
message.im              # DM メッセージ読み込み
message.mpim            # マルチユーザー DM
```

#### OAuth Tokens

- **Bot User OAuth Token**: `xoxb-...`（OpenClaw で使用）
- **App-Level Token**: `xapp-1-...`（Socket Mode 用）

### ステップ 4: OpenClaw に設定

```bash
openclaw config set channels.slack.enabled true
openclaw config set channels.slack.appToken "xapp-1-xxx..."
openclaw config set channels.slack.botToken "xoxb-xxx..."
```

または `~/.openclaw/openclaw.json` を直接編集:

```json
{
  "channels": {
    "slack": {
      "enabled": true,
      "appToken": "xapp-1-xxx...",
      "botToken": "xoxb-xxx..."
    }
  }
}
```

### ステップ 5: 動作確認

```bash
openclaw gateway restart
openclaw status
```

Slack で Bot が online になったか確認。

---

## 🔐 DM・グループアクセス制御

### DM ポリシー設定

```json
{
  "channels": {
    "slack": {
      "dmPolicy": "allowlist",
      "allowFrom": ["slack:U0XXXXXXXXX"]
    }
  }
}
```

| dmPolicy | 説明 |
|----------|------|
| **pairing** | 初回ペアリングコード要求（デフォルト） |
| **allowlist** | `allowFrom` に登録されたユーザーのみ |
| **open** | すべての DM 受け入れ |
| **disabled** | DM 受け付けない |

**このMacの設定**: `allowlist`（自分のみ）

### ユーザーID 確認

```bash
# Slack で自分の user_id を確認
# プロフィール → member_id 確認
# or CLI から
slack auth list
```

設定例（自分のみ許可）:

```bash
openclaw config set channels.slack.dmPolicy allowlist
openclaw config set channels.slack.allowFrom '["slack:U0XXXXXXXXX"]'
```

### グループチャット設定

```json
{
  "channels": {
    "slack": {
      "groupPolicy": "allowlist",
      "groupAllowFrom": ["slack:CXXXXXXXX"],
      "groups": {
        "*": {
          "requireMention": true
        },
        "CXXXXXXXX": {
          "requireMention": false
        }
      }
    }
  }
}
```

| 設定 | 説明 |
|------|------|
| **groupPolicy** | グループ受け入れポリシー |
| **requireMention** | mention がないと無視するか |

---

## 💬 Mention パターン設定

エージェントに mention していない場合の処理:

```json
{
  "messages": {
    "groupChat": {
      "visibleReplies": "automatic",     // automatic / message_tool
      "unmentionedInbound": "room_event" // room_event / quiet
    }
  },
  "agents": {
    "list": [
      {
        "id": "main",
        "groupChat": {
          "mentionPatterns": [
            "@クゥ",
            "クゥ",
            "@openclaw"
          ]
        }
      }
    ]
  }
}
```

| visibleReplies | 説明 |
|---------|------|
| **automatic** | 自動送信（デフォルト） |
| **message_tool** | `message(action=send)` で明示的送信要求 |

---

## 📝 実装例（このMac）

```json5
// ~/.openclaw/openclaw.json
{
  "channels": {
    "slack": {
      "enabled": true,
      "appToken": "xapp-1-...",
      "botToken": "xoxb-...",
      
      // DM: 自分のみ許可
      "dmPolicy": "allowlist",
      "allowFrom": ["slack:U0BTCCXRPJP"],
      
      // グループ: mention 必須
      "groupPolicy": "disabled",  // グループは許可しない（この場合）
      
      // Health check
      "healthMonitor": {
        "enabled": true
      }
    }
  },
  
  // 全体の message ポリシー
  "messages": {
    "visibleReplies": "automatic"
  }
}
```

---

## 🔧 トラブルシューティング

### Bot が online にならない

```bash
# ログ確認
openclaw gateway logs --follow

# Socket Mode 接続エラー
# → appToken / botToken 確認
openclaw config get channels.slack
```

### メッセージが来ない

```bash
# Bot の Scopes を再確認
# https://api.slack.com/apps/APP_ID/oauth

# DM ポリシー確認
openclaw config get channels.slack.dmPolicy
openclaw config get channels.slack.allowFrom

# 自分の User ID が正しいか確認
slack auth list
```

### Socket Mode ハンドシェイク失敗

```
Error: Socket Mode handshake timeout
```

原因と対策:
- `appToken` が正しいか確認
- Slack App に「Socket Mode」が有効か確認
- インターネット接続確認

### メッセージが遅延・タイムアウト

セッションが肥大している可能性:

```bash
openclaw sessions list --detailed
# トークン数が大きいセッションを確認

openclaw sessions compact <session-key> --max-lines 500
```

---

## 🎯 Slack での使用例

### 1. DM で質問

```
user: 今日の天気は？

クゥ: 気象庁データをお調べします...
   東京: 晴れ 32℃
   大阪: 曇り 30℃
```

### 2. ツール呼び出し

```
user: openclaw_setup の README の行数を数えて

クゥ: ファイルを読み込みます...
   README.md: 54 行
```

### 3. コード実行

```
user: 現在の気温ランキング上位5を出して

クゥ: (exec: python3 ~/.openclaw/workspace/bin/jma_rank.py temp 5)
   1. 須佐  33.8℃
   2. 西海  33.8℃
   3. 小松  33.0℃
   4. 豊岡  32.9℃
   5. 秋ヶ島 32.9℃
```

---

## 🔐 セキュリティベストプラクティス

| 対策 | 理由 |
|------|------|
| **Socket Mode 使用** | ポーリングより安全（Bot token が疎通中に必要ない） |
| **Token は環境変数** | 平文コミット防止 |
| **DM ポリシー: allowlist** | 無関係ユーザーからのアクセス防止 |
| **Gateway は localhost:18789** | ローカルアクセスのみ |
| **Slack Org OAuth** | 複数ワークスペース対応時は必須 |

詳細: [SECRETS-MANAGEMENT.md](SECRETS-MANAGEMENT.md)

---

## 📡 他のプラットフォーム（参考）

### Telegram

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "123456:ABCdef...",
      "dmPolicy": "allowlist",
      "allowFrom": ["telegram:123456789"]
    }
  }
}
```

### Discord

```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "botToken": "YOUR_DISCORD_BOT_TOKEN",
      "dmPolicy": "pairing",
      "serverAllowFrom": ["GUILD_ID_1", "GUILD_ID_2"]
    }
  }
}
```

詳細は各プラットフォームの公式ドキュメント参照。

---

## 📖 関連ドキュメント

- [OPERATIONAL-GUIDE.md](OPERATIONAL-GUIDE.md) — 日常運用
- [SECRETS-MANAGEMENT.md](SECRETS-MANAGEMENT.md) — トークン管理
- [CONFIG-REFERENCE.md](CONFIG-REFERENCE.md) — 設定詳細
- 公式: <https://docs.openclaw.ai/channels/slack>

**最後に更新**: 2026-09-01
