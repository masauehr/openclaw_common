# OpenClaw プロジェクト — 統合管理リポジトリ

OpenClaw（セルフホスト型 AI アシスタント・ゲートウェイ）の設定・ドキュメント・プロジェクト管理。

**最新更新**: 2026-09-01 | **バージョン**: OpenClaw `2026.7.1-2`

---

## 🚀 クイックスタート

### このMacでの OpenClaw 運用を始める

```bash
# 1. ドキュメントを読む（10分）
open common/README.md

# 2. 現在の構成を確認
openclaw status

# 3. Gateway を起動（自動起動の場合はスキップ）
openclaw gateway start

# 4. Slack で試す
# Slack DM でクゥ (@openclaw) に話しかける
```

詳細: **[common/README.md](common/README.md)**

---

## 📂 ディレクトリ構成

```
openclaw_prj/
│
├── README.md                    ← このファイル（概要・誘導）
├── CHANGELOG.md                 ← 変更履歴（重要な更新）
├── SECURITY-POLICY.md           ← AI エージェント作業規則
│
├── common/                      ⭐ OpenClaw 共通ドキュメント
│   ├── README.md                ⭐ 【最初はここから】ドキュメント目次
│   ├── ARCHITECTURE.md          システム図・構成・データフロー
│   ├── INSTALL.md               インストール・セットアップ
│   ├── MODELS-SETUP.md          モデル認証・切り替え
│   ├── CONFIG-REFERENCE.md      openclaw.json 完全リファレンス
│   ├── CHANNELS.md              Slack / Telegram など チャネル連携
│   ├── AUTOMATION.md            Cron ジョブ・定期実行設定
│   ├── TOOLS-SKILLS.md          スキル・ツール（web_search / exec など）
│   ├── OPERATIONAL-GUIDE.md     日常運用・メンテナンス・チェック
│   ├── TROUBLESHOOTING.md       トラブル診断・解決フロー
│   ├── SECRETS-MANAGEMENT.md    トークン管理・セキュリティ
│   └── MODELS.json              モデル一覧（JSON リファレンス）
│
└── projects/                    プロジェクト管理
    └── automated-data-collector/ 気象・経済・AI の自動データ収集
        ├── README.md
        ├── SETUP-NOTES.md
        └── .gitignore
```

---

## 📖 ドキュメント案内

### 🎯 目的別ガイド

#### ✨ 「初めて OpenClaw を使う」

1. **[ARCHITECTURE.md](common/ARCHITECTURE.md)** (5分) — 全体図を理解
2. **[INSTALL.md](common/INSTALL.md)** (20分) — インストール・セットアップ
3. **[CHANNELS.md](common/CHANNELS.md)** (15分) — Slack 接続

#### 🔧 「設定を変更したい」

→ **[CONFIG-REFERENCE.md](common/CONFIG-REFERENCE.md)** (30分)  
`openclaw.json` のすべてのオプションを詳細解説

#### 🤖 「モデルを切り替えたい」

→ **[MODELS-SETUP.md](common/MODELS-SETUP.md)** (20分)  
Anthropic / Ollama / OpenRouter の認証・切り替え

#### ⏰ 「定期実行（cron）を設定したい」

→ **[AUTOMATION.md](common/AUTOMATION.md)** (25分)  
ジョブ作成・スケジュール・管理コマンド

#### 🛠️ 「ツール・スキルを使いたい」

→ **[TOOLS-SKILLS.md](common/TOOLS-SKILLS.md)** (15分)  
web_search / exec / JMA スクリプト などの設定

#### 📊 「日常のメンテナンスは？」

→ **[OPERATIONAL-GUIDE.md](common/OPERATIONAL-GUIDE.md)** (30分)  
日次・週次・月次チェック項目・コマンド集

#### 🚨 「トラブルが発生した」

→ **[TROUBLESHOOTING.md](common/TROUBLESHOOTING.md)** (診断フロー)  
問題別の診断手順・解決策

#### 🔐 「トークン・シークレット管理」

→ **[SECRETS-MANAGEMENT.md](common/SECRETS-MANAGEMENT.md)** (25分)  
API キー・トークン・パスワードの安全な管理

#### 📋 「このMacでのモデル構成を知りたい」

→ **[MODELS.json](common/MODELS.json)** (JSON リファレンス)  
サポートモデル一覧・能力比較・推奨構成

---

## 🔄 変更履歴

### 📌 2026-09-01: ドキュメント整備 & Sonnet 運用開始

**新規**:
- ✅ `CHANGELOG.md` — プロジェクト変更履歴
- ✅ `SECURITY-POLICY.md` — AI エージェント作業規則
- ✅ 12 の包括的ドキュメント (`common/`)

**運用変更**:
- 🔄 デフォルトモデル: **Haiku → Sonnet** に切り替え
  - 理由: 品質・推論能力の向上（Claude Pro サブスク範囲内）
  - 影響: 対話の応答時間やや増加（8-10s → 12-18s）、品質向上
  - Cron: haiku 推奨（コスト考慮）

**詳細**: [CHANGELOG.md](CHANGELOG.md)

---

### 📊 このMacでの現在の運用構成

| 項目 | 値 | 状態 |
|------|-----|------|
| **バージョン** | OpenClaw `2026.7.1-2` | ✅ 最新 |
| **Primary Model** | `anthropic/claude-sonnet-5` | ✅ 対話用 |
| **Fallback** | `anthropic/claude-haiku-4-5` | ✅ cron 推奨 |
| **Fallback 2** | `ollama/qwen3.6:35b-mlx` | ⚠️ 課金枠切時 |
| **Web 検索** | SearXNG (localhost:18899) | ✅ 稼働中 |
| **Slack** | Socket Mode (WS) | ✅ online |
| **Cron ジョブ** | 3 本（気象・経済・AI週次） | ✅ 09:10-09:16 |
| **Gateway** | localhost:18789 (launchd) | ✅ 自動起動 |

詳細: [ARCHITECTURE.md](common/ARCHITECTURE.md)

---

## 🔐 セキュリティ

### AI エージェント作業規則

AI アシスタント（クゥ）がこのプロジェクトで守るべきルール:

**✅ 許可**:
- `/Users/masahiro/openclaw_prj/` 以下の読・書・作・削

**🔴 禁止**:
- `~/.openclaw/openclaw.json` の変更（token 含む）
- `~/.anthropic/credentials.json` の読み込み
- 他のプロジェクト・システムファイルの変更

**📢 報告義務**:
- セッション開始時に作業ディレクトリを宣言
- 禁止ファイルへのアクセス試行時は許可を求める

詳細: [SECURITY-POLICY.md](SECURITY-POLICY.md)

---

## 🔗 関連リポジトリ

| リポジトリ | 用途 | URL |
|-----------|------|-----|
| **openclaw_common** | 本プロジェクト（このMacの OpenClaw ドキュメント） | <https://github.com/masauehr/openclaw_common> |
| **openclaw_setup** | 導入・試行の詳細ログ | <https://github.com/masauehr/openclaw_setup> |
| **automated-data-collector** | 気象・経済・AI データ収集スクリプト | <https://github.com/masauehr/automated-data-collector> |

---

## 📚 ドキュメント統計

| 項目 | 数値 |
|------|------|
| **ドキュメントファイル** | 12 個 |
| **総ボリューム** | 約 91KB |
| **実装例** | 100+ |
| **テーブル・リスト** | 150+ |
| **相互参照** | すべてのドキュメント相互リンク完備 |
| **最新性** | 2026-09-01 現在のこのMacの実装を反映 |

---

## 🎯 次のステップ

### 1️⃣ 今すぐ読むべき

- **[common/README.md](common/README.md)** (5分) — ドキュメント目次・全体像
- **[CHANGELOG.md](CHANGELOG.md)** (5分) — 本日の変更点・Sonnet 移行記録

### 2️⃣ 用途別に読む

上記「ドキュメント案内」セクション参照

### 3️⃣ 運用で活用

**日次**:
```bash
openclaw status
```

**週次**:
```bash
# [OPERATIONAL-GUIDE.md](common/OPERATIONAL-GUIDE.md) の「週次チェック」を実行
openclaw sessions list --detailed
openclaw cron runs <job_id> --recent 7
```

**トラブル時**:
→ **[TROUBLESHOOTING.md](common/TROUBLESHOOTING.md)** で診断

---

## 📞 サポート・リソース

### このプロジェクト

- **Issues**: GitHub Issues で報告・提案
- **Wiki**: README & CHANGELOG で履歴管理

### OpenClaw 本体

- **公式ドキュメント**: <https://docs.openclaw.ai/>
- **GitHub**: <https://github.com/openclaw/openclaw>
- **コミュニティ**: Discord / GitHub Discussions

---

## 🏢 プロジェクト情報

**管理者**: Msashiro UEHARA  
**エージェント**: クゥ (Kuu) 🐈‍⬛（黒猫）  
**OS**: macOS 26.6.2 (arm64)  
**Node.js**: v25.9.0  
**npm**: 11.12.1  

**作成日**: 2026-08-29  
**最終更新**: 2026-09-01 10:33 JST

---

## 📝 ライセンス

このドキュメント・プロジェクト: MIT  
OpenClaw 本体: MIT (https://github.com/openclaw/openclaw)

---

**📖 本プロジェクトを開く**: [common/README.md](common/README.md)  
**📋 変更履歴を見る**: [CHANGELOG.md](CHANGELOG.md)  
**🔐 セキュリティルール**: [SECURITY-POLICY.md](SECURITY-POLICY.md)
