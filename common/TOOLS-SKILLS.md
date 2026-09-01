# スキル・ツール設定ガイド

OpenClaw で利用可能なスキル（拡張機能）とツール（実行機能）の設定・運用。

**このMacでの構成**: web_search（SearXNG） + exec + JMA スクリプト

---

## 🛠️ ツール概要

OpenClaw の「ツール」= エージェントが呼び出して実行できる機能

| ツール | 説明 | このMac |
|--------|------|--------|
| **web_search** | Web 検索（SearXNG） | ✅ 有効 |
| **web_fetch** | URL 内容取得 | ✅ 有効 |
| **exec** | シェルコマンド実行 | ✅ 有効 |
| **read / write / edit** | ファイル操作 | ✅ 有効 |
| **git** | Git 操作 | ✅ 有効 |
| **browser** | ブラウザ自動化 | ❌ 未使用 |
| **canvas** | HTML/画像レンダリング | ❌ 未使用 |

---

## 📍 Web 検索（SearXNG）

### 概要

**SearXNG** = 自己ホスト型メタサーチエンジン
- DuckDuckGo のボット判定をバイパス
- API キー不要・上限なし
- `localhost:18899` で稼働

### セットアップ

```bash
# インストール（初回のみ）
git clone https://github.com/searxng/searxng.git ~/searxng
cd ~/searxng
pip install -r requirements.txt

# 起動確認
curl http://127.0.0.1:18899

# launchd 常駐化（別途 plist ファイル作成）
# ~/Library/LaunchAgents/local.searxng.plist
```

### 設定（OpenClaw）

```json
{
  "tools": {
    "web": {
      "search": {
        "enabled": true,
        "provider": "searxng"
      },
      "searxng": {
        "baseUrl": "http://127.0.0.1:18899"
      }
    }
  }
}
```

### 使用例

エージェントが自動的に呼び出し:

```bash
# Slack で
user: 今日の日経平均は？

# → 内部で web_search 実行
# → SearXNG が DuckDuckGo / Google を統合
# → 結果を返す
```

### トラブルシューティング

```bash
# SearXNG が起動しているか確認
curl -s http://127.0.0.1:18899 | head -20

# 起動していない場合
launchctl start local.searxng

# エラーの場合
cat ~/Library/Logs/searxng.log
```

---

## 🔧 Exec（シェルコマンド実行）

### 概要

エージェントが `exec` ツールを呼び出してシェルコマンドを実行。

### 設定

```json
{
  "tools": {
    "profile": "coding"  // "basic" / "coding" / "full"
  }
}
```

| profile | exec 許可 |
|---------|---------|
| **basic** | ❌ 不可 |
| **coding** | ✅ 可能（このMac） |
| **full** | ✅ 可能 |

### セキュリティ

```json
{
  "tools": {
    "exec": {
      "sandboxed": true,                    // サンドボックス有効
      "timeoutSeconds": 30,                 // タイムアウト
      "allowedCommands": [                  // ホワイトリスト
        "python3",
        "grep",
        "find"
      ]
    }
  }
}
```

### 使用例（このMac）

#### 気象庁ランキング取得

```bash
python3 ~/.openclaw/workspace/bin/jma_rank.py temp 10
```

出力:

```
1. 須佐  33.8℃
2. 西海  33.8℃
...
```

#### ファイル処理

```bash
grep -r "openclaw" ~/projects/ | head -10
```

---

## 📊 スキル（拡張機能）

**スキル** = 複数ツールを組み合わせた再利用可能な手順

### インストール済みスキル

```bash
openclaw skills list
```

**このMacでは主に**:
- `github` — GitHub CLI（PR / Issue 操作）
- `slack` — Slack メッセージング
- `weather` — 気象情報（web_search 統合）

### スキル有効化・無効化

```bash
# 有効化
openclaw skills enable github

# 無効化
openclaw skills disable github

# 確認
openclaw skills list --enabled
```

### スキル制限（agents.defaults.skills）

```json
{
  "agents": {
    "defaults": {
      "skills": ["github", "slack"]  // この 2 つのみ
    }
  }
}
```

---

## 🌡️ 補助スクリプト（JMA）

### 概要

気象庁（JMA）アメダス観測値のランキング抽出。

**パス**: `~/.openclaw/workspace/bin/jma_rank.py`

### 用法

```bash
python3 ~/.openclaw/workspace/bin/jma_rank.py <element> [top]
```

### パラメータ

| element | 説明 |
|---------|------|
| `temp` | 気温（℃） |
| `wind` | 風速（m/s） |
| `gust` | 突風（m/s） |
| `precip1h` | 1時間降水量（mm） |
| `precip3h` | 3時間降水量（mm） |
| `precip24h` | 24時間降水量（mm） |
| `humidity` | 湿度（%） |
| `sun1h` | 日射量（MJ/㎡） |

| 引数 | 説明 |
|------|------|
| `top` | 上位件数（デフォルト 10） |

### 使用例

```bash
# 気温ランキング上位 10
python3 ~/.openclaw/workspace/bin/jma_rank.py temp 10

# 24時間降水量上位 5
python3 ~/.openclaw/workspace/bin/jma_rank.py precip24h 5

# 風速ランキング（デフォルト 10）
python3 ~/.openclaw/workspace/bin/jma_rank.py wind
```

### 出力例

```
1. 須佐  33.8℃  (81011)
2. 西海  33.8℃  (84306)
3. 小松  33.0℃  (56276)
...
```

### データソース

- **JSON API**: 気象庁公開 JSON
- **ファイル**: `amedas/data/map/{timestamp}.json`
- **テーブル**: `amedas/const/amedastable.json`（観測地点情報）

**重要**: HTML スクレイピング禁止（JS 描画のため）。JSON API 直叩きのみ。

### トラブルシューティング

```bash
# スクリプト確認
ls -la ~/.openclaw/workspace/bin/jma_rank.py

# 実行権限確認
chmod +x ~/.openclaw/workspace/bin/jma_rank.py

# テスト実行
python3 ~/.openclaw/workspace/bin/jma_rank.py temp 5

# Python パス確認
which python3  # /opt/homebrew/bin/python3 推奨
python3 --version  # 3.12 以上

# JSON データ直接確認
curl -s https://www.jma.go.jp/bosai/amedas/data/latest_time.txt
```

---

## 🎯 実装例（このMac）

### Web 検索 + exec 統合

気象ダイジェスト生成時:

```
[エージェント実行]
1. exec: python3 ~/.openclaw/workspace/bin/jma_rank.py temp 10
   → 気温ランキング取得
2. web_search: "日本 警報 今日"
   → 最新警報情報を検索
3. 統合 → ダイジェスト作成
4. Slack DM に配信
```

### トークンバジェット管理

実行ツールはトークン消費するため:

```json
{
  "agents": {
    "defaults": {
      "compaction": {
        "mode": "safeguard",
        "keepRecentTokens": 10000
      }
    }
  }
}
```

---

## 📈 パフォーマンス最適化

### Web 検索の最小化

不必要な検索を避ける:

```json
{
  "agents": {
    "defaults": {
      "tools": {
        "web": {
          "search": {
            "enabled": true,
            "autoSearch": false  // 手動トリガーのみ
          }
        }
      }
    }
  }
}
```

### Exec タイムアウト

```json
{
  "tools": {
    "exec": {
      "timeoutSeconds": 30  // デフォルト
    }
  }
}
```

気象スクリプトが時間掛かる場合は 60 秒に延長:

```bash
openclaw config set tools.exec.timeoutSeconds 60
```

### キャッシング

```json
{
  "tools": {
    "web": {
      "cache": {
        "enabled": true,
        "ttlSeconds": 3600  // 1時間キャッシュ
      }
    }
  }
}
```

---

## 🚨 ツール呼び出しエラー時

### web_search が失敗

```bash
# SearXNG が起動しているか
curl http://127.0.0.1:18899

# Gateway ログ確認
openclaw gateway logs --follow | grep searxng
```

### exec がタイムアウト

```bash
# タイムアウト延長
openclaw config set tools.exec.timeoutSeconds 120

# 実行時間が長いコマンドを確認
time python3 ~/.openclaw/workspace/bin/jma_rank.py temp 10
```

### ファイル操作（read/write）のパーミッション

```bash
# ファイルが読み込める状態か
ls -la ~/.openclaw/workspace/bin/jma_rank.py

# 必要なら権限変更
chmod 644 ~/.openclaw/workspace/bin/jma_rank.py
```

---

## 📖 関連ドキュメント

- [CONFIG-REFERENCE.md](CONFIG-REFERENCE.md) — ツール設定の詳細
- [AUTOMATION.md](AUTOMATION.md) — cron で使用するツール
- [OPERATIONAL-GUIDE.md](OPERATIONAL-GUIDE.md) — 日常運用
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) — トラブル対応
- 公式: <https://docs.openclaw.ai/tools>

**最後に更新**: 2026-09-01
