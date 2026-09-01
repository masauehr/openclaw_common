# 設定リファレンス（`openclaw.json`）

`~/.openclaw/openclaw.json` の完全な設定ガイド。

**最新バージョン**: OpenClaw `2026.7.1-2`  
**公式ドキュメント**: <https://docs.openclaw.ai/gateway/configuration-reference>

---

## 📋 ファイル場所・形式

| 項目 | 値 |
|------|-----|
| **パス** | `~/.openclaw/openclaw.json` |
| **形式** | JSON5（コメント・末尾カンマ対応） |
| **バックアップ** | `~/.openclaw/openclaw.json.bak`（自動生成） |
| **ホットリロード** | 有効（ファイル保存時に即反映） |
| **バリデーション** | 厳密（スキーマ違反で起動失敗） |

---

## 🔑 基本構造

```json5
{
  // Gateway（基盤サーバ）
  "gateway": {
    "mode": "local",              // local / loopback / lan
    "bind": "loopback",           // loopback / lan / tailnet / custom
    "port": 18789,                // WebSocket ポート
    "auth": {
      "mode": "token",            // none / token / password / trusted-proxy
      "token": "***"              // トークン（平文。gitignore推奨）
    }
  },

  // エージェント（AI 実行環境）
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "model": {
        "primary": "anthropic/claude-haiku-4-5",
        "fallbacks": ["anthropic/claude-sonnet-5"]
      }
    },
    "list": [
      { "id": "main", "default": true }
    ]
  },

  // チャンネル（メッセージング連携）
  "channels": {
    "slack": {
      "enabled": true,
      "appToken": "xapp-***",
      "botToken": "xoxb-***"
    }
  },

  // セッション・ストレージ
  "session": {
    "dmScope": "per-channel-peer"
  }
}
```

---

## 📡 Gateway 設定

### mode（ゲートウェイモード）

```json
"gateway": {
  "mode": "local"  // 選択肢: local / loopback / lan
}
```

| 値 | 説明 |
|-----|------|
| **local** | localhost のみ（ローカル専用） |
| **loopback** | 127.0.0.1 のみ（最も安全） |
| **lan** | ローカルネットワーク公開（⚠️ セキュリティ要注意） |

**推奨**: `local` or `loopback`（セキュリティ重視）

### port（ポート番号）

```json
"gateway": {
  "port": 18789  // デフォルト: 18789
}
```

### auth（認証方式）

```json
"gateway": {
  "auth": {
    "mode": "token",           // none / token / password / trusted-proxy
    "token": "sk-1234567890"   // token mode 時に必須
  }
}
```

**推奨**: `token` mode（自動生成可）

トークンローテーション:

```bash
openclaw config set gateway.auth.token "新しいトークン"
openclaw gateway restart
```

### tailscale（Tailscale 連携・リモートアクセス）

```json
"gateway": {
  "tailscale": {
    "mode": "off",              // off / serve / funnel
    "resetOnExit": false
  }
}
```

| 値 | 説明 |
|-----|------|
| **off** | Tailscale 未使用（ローカルのみ） |
| **serve** | Tailscale を経由して LAN 公開 |
| **funnel** | 公開インターネット経由で公開（⚠️ 要注意） |

---

## 🤖 Agent 設定（`agents`）

### defaults（全エージェント共通設定）

```json
"agents": {
  "defaults": {
    "workspace": "~/.openclaw/workspace",
    
    // モデル・プロバイダー
    "model": {
      "primary": "anthropic/claude-haiku-4-5",
      "fallbacks": ["anthropic/claude-sonnet-5"]
    },
    
    // モデルカタログ（許可リスト）
    "models": {
      "anthropic/claude-haiku-4-5": {},
      "anthropic/claude-sonnet-5": {},
      "ollama/qwen3.6:35b-mlx": {}
    },
    
    // 対話設定
    "heartbeat": {
      "every": "30m",           // 定期チェックイン間隔
      "target": "last"          // 最後のチャネル
    },
    
    // メモリ・コンパクション
    "compaction": {
      "mode": "safeguard",      // off / safeguard / aggressive
      "keepRecentTokens": 10000
    },
    
    // サンドボックス（実行環境隔離）
    "sandbox": {
      "mode": "off",            // off / non-main / all
      "scope": "agent"          // session / agent / shared
    }
  }
}
```

### list（個別エージェント）

複数エージェント時:

```json
"agents": {
  "list": [
    {
      "id": "main",
      "default": true,
      "workspace": "~/.openclaw/workspace"
    },
    {
      "id": "work",
      "workspace": "~/.openclaw/workspace-work"
    }
  ]
}
```

---

## 💬 Channel 設定（`channels`）

### Slack

```json
"channels": {
  "slack": {
    "enabled": true,
    "appToken": "xapp-1-xxxx",
    "botToken": "xoxb-xxxx",
    
    // DM アクセス制御
    "dmPolicy": "allowlist",    // pairing / allowlist / open / disabled
    "allowFrom": ["slack:U0XXXXXXXXX"],
    
    // グループチャット設定
    "groupPolicy": "allowlist",
    "groupAllowFrom": [],
    
    // Mention requirement
    "groups": {
      "*": {
        "requireMention": true
      }
    }
  }
}
```

| dmPolicy | 説明 |
|----------|------|
| **pairing** | 初回ペアリングコード要求（デフォルト） |
| **allowlist** | `allowFrom` に登録された sender のみ |
| **open** | すべての DM 受け入れ |
| **disabled** | DM 受け付けない |

詳細: [CHANNELS.md](CHANNELS.md)

---

## 🔄 Session 設定（`session`）

### dmScope（セッション分離単位）

```json
"session": {
  "dmScope": "per-channel-peer"  // main / per-peer / per-channel-peer / per-account-channel-peer
}
```

| 値 | セッション分離 |
|----|--------------|
| **main** | 全員で 1 セッション共有（⚠️ プライバシー注意） |
| **per-peer** | ユーザーごと（チャネル横断） |
| **per-channel-peer** | チャネル × ユーザーごと（推奨） |
| **per-account-channel-peer** | アカウント × チャネル × ユーザー |

**推奨**: `per-channel-peer`（プライバシーとシンプルさのバランス）

### threadBindings（スレッド・トピック分離）

```json
"session": {
  "threadBindings": {
    "enabled": true,
    "idleHours": 24,            // 非アクティブ時の保持期間
    "maxAgeHours": 0            // 最大保持期間（0 = 無制限）
  }
}
```

### reset（セッションリセット）

```json
"session": {
  "reset": {
    "mode": "daily",            // off / daily / weekly / on-demand
    "atHour": 4,                // 毎日何時にリセットするか
    "idleMinutes": 120          // 非アクティブ何分でリセット
  }
}
```

---

## 🛠️ Tools 設定（`tools`）

### profile（ツール許可セット）

```json
"tools": {
  "profile": "coding"           // basic / coding / full / custom
}
```

| profile | 許可ツール |
|---------|----------|
| **basic** | read / write / web_search のみ |
| **coding** | basic + exec / process / git など |
| **full** | すべて（browser / canvas など含む） |

### web（Web 検索）

```json
"tools": {
  "web": {
    "search": {
      "enabled": true,
      "provider": "searxng"     // brave / duckduckgo / searxng など
    },
    "searxng": {
      "baseUrl": "http://127.0.0.1:18899"
    }
  }
}
```

---

## 🔐 Model プロバイダー設定（`models`）

### Anthropic（Claude）

```json
"models": {
  "providers": {
    "anthropic": {
      "apiKey": {
        "source": "env",
        "id": "ANTHROPIC_API_KEY"
      }
    }
  }
}
```

### Ollama（ローカル）

```json
"models": {
  "providers": {
    "ollama": {
      "baseUrl": "http://localhost:11434"
    }
  }
}
```

### OpenRouter（プロキシ）

```json
"models": {
  "providers": {
    "openrouter": {
      "apiKey": "sk-or-xxx",
      "baseUrl": "https://openrouter.ai/api/v1"
    }
  }
}
```

---

## ⏰ Cron 設定（`cron`）

定期実行ジョブの全体設定:

```json
"cron": {
  "enabled": true,
  "maxConcurrentRuns": 8,
  "sessionRetention": "24h",
  "runLog": {
    "maxBytes": "2mb",
    "keepLines": 2000
  }
}
```

| 項目 | 説明 |
|------|------|
| **enabled** | cron 機能有効化 |
| **maxConcurrentRuns** | 同時実行ジョブ数上限 |
| **sessionRetention** | ジョブセッション保持期間 |
| **runLog.maxBytes** | ジョブ実行ログの保持容量 |
| **runLog.keepLines** | ジョブ実行ログの保持行数 |

詳細: [AUTOMATION.md](AUTOMATION.md)

---

## 🪝 Webhook 設定（`hooks`）

外部システムからの HTTP コールバック:

```json
"hooks": {
  "enabled": true,
  "token": "secret-token-12345",
  "path": "/hooks",
  "defaultSessionKey": "hook:ingress",
  "allowRequestSessionKey": false,
  "allowedSessionKeyPrefixes": ["hook:"],
  
  "mappings": [
    {
      "match": { "path": "gmail" },
      "action": "agent",
      "agentId": "main",
      "deliver": true
    }
  ]
}
```

詳細: [Webhooks @ 公式docs](https://docs.openclaw.ai/gateway/webhooks)

---

## 📝 UI・ログ設定（`ui` / `logging`）

### UI

```json
"ui": {
  "darkMode": "auto",           // auto / light / dark
  "language": "ja"              // ja / en など
}
```

### Logging

```json
"logging": {
  "level": "info",              // trace / debug / info / warn / error
  "format": "json",             // json / text
  "maxFiles": 10,
  "maxSize": "10mb"
}
```

---

## 🎫 Identity（`identity`）

エージェントの公開情報:

```json
"identity": {
  "name": "クゥ",
  "emoji": "🐈‍⬛",
  "avatar": "https://example.com/avatar.png"
}
```

**注**: より詳しい人格設定は `~/.openclaw/workspace/IDENTITY.md` で管理。

---

## 📚 Environment Variables（`env`）

環境変数の設定:

```json
"env": {
  "vars": {
    "ANTHROPIC_API_KEY": "sk-ant-xxx",
    "SEARXNG_SECRET_KEY": "..."
  },
  "shellEnv": {
    "enabled": true,
    "timeoutMs": 15000
  }
}
```

値中で変数参照可:

```json
"tools": {
  "web": {
    "searxng": {
      "baseUrl": "${SEARXNG_BASE_URL}"
    }
  }
}
```

---

## 🎯 実装例

### 例 1: ローカルのみ（オフライン構成）

```json5
{
  "gateway": {
    "mode": "local",
    "port": 18789,
    "auth": { "mode": "token", "token": "***" }
  },
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "model": {
        "primary": "ollama/qwen3.6:35b-mlx",
        "fallbacks": []
      }
    }
  },
  "channels": {
    "slack": { "enabled": false }
  },
  "tools": {
    "web": { "search": { "enabled": false } }
  }
}
```

### 例 2: Anthropic + Slack フル構成

```json5
{
  "gateway": {
    "mode": "local",
    "port": 18789,
    "auth": { "mode": "token", "token": "***" }
  },
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "model": {
        "primary": "anthropic/claude-haiku-4-5",
        "fallbacks": ["anthropic/claude-sonnet-5"]
      },
      "models": {
        "anthropic/claude-haiku-4-5": {},
        "anthropic/claude-sonnet-5": {}
      }
    }
  },
  "channels": {
    "slack": {
      "enabled": true,
      "appToken": "xapp-***",
      "botToken": "xoxb-***",
      "dmPolicy": "allowlist",
      "allowFrom": ["slack:U0XXXXXXXXX"]
    }
  },
  "session": {
    "dmScope": "per-channel-peer"
  },
  "tools": {
    "profile": "coding",
    "web": {
      "search": {
        "enabled": true,
        "provider": "searxng"
      },
      "searxng": {
        "baseUrl": "http://127.0.0.1:18899"
      }
    }
  },
  "cron": {
    "enabled": true,
    "maxConcurrentRuns": 8
  }
}
```

---

## 🔍 検証・デバッグ

### 設定検証

```bash
openclaw config validate
```

### 設定確認（特定フィールド）

```bash
openclaw config get agents.defaults.model
openclaw config get channels.slack
```

### 設定パッチ（部分更新）

```bash
openclaw config patch --stdin << 'EOF'
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-5"
      }
    }
  }
}
EOF
```

### ホットリロード状態確認

```bash
openclaw gateway logs --follow
# "config reloaded" を確認
```

---

## ⚠️ よくある間違い

| 間違い | 症状 | 解決 |
|--------|------|------|
| `gateway.auth.token` が空 | 起動失敗 | `onboard` で再生成 |
| `agents.defaults.model` が文字列 | 起動失敗 | `{ "primary": "..." }` にする |
| チャネルの `allowFrom` が配列でない | 起動失敗 | `["slack:U0XXXXXXXXX"]` に修正 |
| `gateway.mode: "loopback"` + `dmPolicy: "open"` | セキュリティ穴 | `loopback` ならば良好 |
| cron 有効化忘れ | ジョブ実行されない | `cron.enabled: true` に |

---

## 📖 関連ドキュメント

- [ARCHITECTURE.md](ARCHITECTURE.md) — 全体図・コンポーネント
- [INSTALL.md](INSTALL.md) — インストール手順
- [CHANNELS.md](CHANNELS.md) — チャネル詳細設定
- [AUTOMATION.md](AUTOMATION.md) — cron 設定
- [SECRETS-MANAGEMENT.md](SECRETS-MANAGEMENT.md) — シークレット管理
- 公式: <https://docs.openclaw.ai/gateway/configuration-reference>

**最後に更新**: 2026-09-01
