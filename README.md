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

## 🆕 OpenClaw 2.0（v2026.8.1）アップグレード情報（2026-09-02 調査）

Slack で「OpenClaw2が公開された。DB化してテキストデータが読めなくなるらしい」との質問を受けて調べた内容と、
クゥ（本エージェント）の回答の要約。一次情報: <https://docs.openclaw.ai/releases/2026.8.1>、
GitHub Releases、二次検証記事 <https://cellcog.ai/blog/openclaw-2-0/>。

### 概要

- バージョンタグは `v2026.8.1`（2026-08-31 03:30 UTC 公開）。開発チームが「2.0」と呼んでいるのは
  規模の大きさゆえ（プロジェクト全体の全マージPRのうち約半分に相当する16,000+件、933人の
  コントリビューターが関わった）で、日付ベースのバージョニング自体は維持。前リリースから
  約7週間空いての大型リリース。
- 目玉機能:
  - オンボーディング簡略化（既存のChatGPT/Claudeサブスク・APIキー・ローカルモデルを自動検出し、
    選んだモデルが実際に応答するか検証してから確定）
  - Shared cloud sessions（複数人が同じセッションに参加・文脈ごと作業を引き継げる「マルチプレイヤー」機能。
    OpenClaw開発チーム自身もこれで OpenClaw を開発しているとのこと）
  - メモリ機構の統合（built-in Memory が検索・想起の中核に。QMDアドオンは廃止。バックグラウンドで
    「出所が確かな」記憶を長期メモリへ昇格＝Dream Diary、自己学習で強い教訓をスキル化提案）
  - Skill Workshop（スキル作成・検証・カタログ化・適用履歴を一本化したワークフロー）
  - セキュリティモデルの構造強化（承認がリクエスト・コマンド・セッション・人に厳密紐付け、
    セッションごとに読み取り専用/ガード付き/workspace/full の権限選択、チーム共有クレデンシャルストア）

### 「自己学習によるスキル自動生成」は Hermes Agent に寄せてきた？（ユーザー指摘・2026-09-02）

- この機能（強い教訓を検知して自動でスキル化提案）は、Hermes Agent（Nous Research製）の
  目玉機能「自己改善ループ」（タスク完了後に再利用可能なスキルを自動生成し、使用中に自律的に
  改良していく）と概念的にかなり近い。
- OpenClaw 公式は Hermes を意識したとは明言していないが、Hermes Agent の GitHub README には
  `hermes claw migrate`（OpenClaw からの設定・メモリ・スキル・APIキー移行コマンド）が既に存在し、
  両陣営が互いを競合として意識し合っている状況証拠はある。
- 違いは抑制の度合い: Hermes の自己改善ループは比較的積極的（基本ループ自体はデフォルト寄り）なのに
  対し、OpenClaw 2.0 の自己学習は「強い教訓のみ」「提案どまり」で、Skill Workshop の承認フロー
  （apply/reject/quarantine）に必ず乗る抑制的設計。クゥ（本エージェント）が使っている `skill_workshop`
  ツールは、まさにこの「提案 → 人間が明示的に適用/却下」という設計思想の実装そのもの。

### 「テキストデータが読み込めない」の正体（セッション・DB移行）

- 公式 warning 原文: "This release changes how sessions and transcripts are stored by moving
  them into SQLite... sessions created after the migration will not appear in older releases"
  （ダウングレード時）。
- 実態: **セッション・会話トランスクリプトの保存先がテキストファイルから SQLite へ移行**。
  アップグレード時は自動変換されるため、通常利用で「読めなくなる」ことはない。
- 問題が起きるのは**ダウングレード（ロールバック）方向のみ**: SQLite 移行後に新規作成された
  セッションは、古いテキストベース版に戻すと見えなくなる（一方向の移行）。
- `AGENTS.md`/`SOUL.md`/`USER.md`/`MEMORY.md` 等のワークスペーステキストファイルや
  `openclaw.json` 設定は対象外（sessions/transcripts のみが対象）。

### リリース初日（8/31）のバグ（GitHub Issue Tracker 確認済み。翌日 8.2 で一部修正）

- `doctor --fix` が TTY 無し（SSH・自動化スクリプト経由）で実行されると**移行処理を黙って
  スキップ**するのに、エラーメッセージは相変わらず「doctor --fix を実行しろ」と表示し続ける
  （P1、Gateway クラッシュループ・exec 承認ブロックの実害報告あり。既に修正版公開済み）。
- Gemini 埋め込みでのメモリ同期がバッチサイズ超過で中断し、メモリが古いまま止まる（P1、未修正）。
- 旧プラグインの consent（権限同意）が引き継がれず、`plugins update --all --accept-capabilities`
  が成功したように見えて実は古いプラグインがブロックされたまま（P2、未修正）。
- 翌日の `v2026.8.2` で「legacy v17 agent database の修復」「セッション移行がまだブロック中なら
  明確に失敗させる」といった追加の安全策が投入されている＝8.1 単体はまだ移行が荒かった。

### 破壊的変更（3点＋1点、`openclaw doctor --fix` で対応）

| 変更 | 対応 |
|---|---|
| セッション/トランスクリプトの SQLite 移行 | 事前に検証済みバックアップ必須。ダウングレード注意 |
| OpenProse プラグイン・`/prose` 廃止 | `openclaw doctor --fix` ＋アップストリームの Agent Skill 移行 |
| `codex/`・`openai-codex/` モデル参照 | `openclaw doctor --fix` が `openai/*` へ自動移行 |
| Plugin SDK 非推奨ゲート | 2026-09-01 発効（既に到来）。外部プラグインは要確認 |

### このMacでの判断（2026-09-02時点）

- 現状 `openclaw@2026.7.1-2`。個人 Slack DM＋cron3本＋SearXNG＋Ollama ローカルモデルの構成では、
  2.0 の目玉（Shared cloud sessions 等）の恩恵はほぼ無い。
- リリース生後2〜3日で day-1 バグが複数あり（上記）、収束を見てから上げる方が安全。
- **判断: 即時アップグレードは見送り、1〜2週間様子見**（v2026.8.2 以降のバグ収束状況を見て再検討）。

### 今後の予定（TODO）

- **2026-09-中旬以降を目安に、v2026.8.x 系の安定版へアップグレード予定**。
- 実施前に必ず: (1) 検証済みバックアップ、(2) 対話的 TTY から `openclaw doctor --fix` を実行
  （SSH・非対話実行だと移行が黙ってスキップされるバグが 8.1 にあったため）、
  (3) アップグレード後に `openclaw cron list` / `openclaw channels status --deep` / `openclaw doctor`
  で疎通確認。
- アップグレード実施時は、結果をこの README と `common/TROUBLESHOOTING.md` に追記すること。

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
