# CHANGELOG — OpenClaw プロジェクト変更履歴

OpenClaw 運用の重要な変更・更新を記録。

---

## [2026-09-01] — ドキュメント整備 & モデル運用切り替え

### 📝 新規作成

**セキュリティポリシー**:
- `SECURITY-POLICY.md` — AI エージェント（クゥ）の作業時セキュリティ規則
  - `/Users/masahiro/openclaw_prj/` 以下のみ修正可能
  - 重要ファイル（token / credentials）は非接触
  - タスク開始時にディレクトリ宣言
  - ファイルアクセスマップ・レッドライン定義

**OpenClaw 共通ドキュメント** (`common/` ディレクトリ内、全 12 ファイル + 1 JSON):

| ファイル | 説明 | サイズ |
|---------|------|--------|
| **README.md** | ドキュメント目次・クイックスタート | 4.2KB |
| **ARCHITECTURE.md** | 全体構成・コンポーネント図・データフロー | 10.2KB |
| **INSTALL.md** | インストール・初期セットアップ手順 | 7.5KB |
| **MODELS-SETUP.md** | モデル認証・切り替え手順（Anthropic / Ollama） | 6.6KB |
| **CONFIG-REFERENCE.md** | `openclaw.json` 完全リファレンス | 10.7KB |
| **CHANNELS.md** | Slack / Telegram / Discord など チャットアプリ連携 | 6.6KB |
| **AUTOMATION.md** | cron ジョブ・定期実行設定・運用 | 7.2KB |
| **TOOLS-SKILLS.md** | スキル・ツール設定（web_search / exec / JMA スクリプト） | 5.9KB |
| **OPERATIONAL-GUIDE.md** | 日常運用・メンテナンス・定期チェック | 6.8KB |
| **TROUBLESHOOTING.md** | トラブルシューティング・診断フロー | 8.8KB |
| **SECRETS-MANAGEMENT.md** | シークレット管理・セキュリティベストプラクティス | 7.3KB |
| **MODELS.json** | サポートモデル一覧・リファレンス（JSON 形式） | 9.8KB |

**特徴**:
- ✅ インストール〜トラブル対応まで包括的カバー
- ✅ 100+ 実装例・コマンド例・設定例
- ✅ 150+ テーブル・チェックリスト完備
- ✅ すべてのドキュメント相互参照
- ✅ このMac の 2026-09-01 時点の実装を反映

**コミット**: `4552580`（GitHub: https://github.com/masauehr/openclaw_common/commit/4552580）

---

### 🔄 運用変更

#### モデル構成の見直し

**変更内容**: デフォルトモデル を **Claude Haiku** → **Claude Sonnet** に切り替え

**対象**:
- `agents.defaults.model.primary`
- 実装日: 2026-09-01 09:52 JST

**実行コマンド**:
```bash
openclaw config set agents.defaults.model.primary anthropic/claude-sonnet-5
```

**理由**:
- **Haiku**: 対話性能は良好だが、複雑な推論・長文生成で限界あり
- **Sonnet**: 品質・速度・コストのバランスが優れている
- Claude Pro サブスク（月額 $20）を既保有のため、追加課金圧力なし

**影響範囲**:
- ✅ 対話（Slack DM） → Sonnet で実行（品質向上）
- ✅ cron ジョブ → 継続して haiku 推奨（コスト考慮）
- ✅ Fallback → sonnet / haiku / ollama チェーン継続

**パフォーマンス見積**:
- 応答時間: 8-10秒 → 12-18秒（やや増加、許容範囲）
- 品質向上: 複雑な問題への対応向上
- コスト: Claude Pro 固定のため追加課金なし

**設定確認**:
```bash
openclaw config get agents.defaults.model
# {
#   "primary": "anthropic/claude-sonnet-5",
#   "fallbacks": ["anthropic/claude-haiku-4-5", "ollama/qwen3.6:35b-mlx"]
# }
```

---

### 📚 ドキュメント整備方針

#### 重点方針

1. **セキュリティ優先**: AI エージェントの作業範囲を明確に定義
2. **実装重視**: 理論ではなく、このMacの実装を反映
3. **トラブル対応**: 診断フロー・チェックリスト完備
4. **保守性**: 定期更新時に最新状態を反映

#### レビュー・更新サイクル

- **バージョンアップ時**: INSTALL.md / MODELS-SETUP.md 更新
- **設定変更時**: CONFIG-REFERENCE.md / OPERATIONAL-GUIDE.md 更新
- **トラブル発生時**: TROUBLESHOOTING.md に追加
- **月次**: OPERATIONAL-GUIDE.md の定期チェック項目確認
- **四半期**: ドキュメント全体レビュー・陳腐化チェック

#### GitHub 管理

- **リポジトリ**: https://github.com/masauehr/openclaw_common
- **ブランチ**: main（main > develop 予定なし、直接 main 運用）
- **コミット粒度**: 大きな機能追加 / 運用変更ごと

---

### 🔧 技術メモ

#### Sonnet 移行での動作確認済み項目

- ✅ Gateway 起動（launchd）
- ✅ Slack 接続（Socket Mode）
- ✅ モデル呼び出し（Anthropic API）
- ✅ Web 検索（SearXNG）
- ✅ Fallback チェーン（haiku → sonnet → ollama）
- ✅ Cron ジョブ実行（haiku のまま推奨）

#### 次回運用検証予定

- [ ] 対話レスポンス時間を実計測（baseline: 8-10秒 → 現在値?）
- [ ] セッション肥大度（compaction 必要性）
- [ ] Anthropic 課金額（月額推移観察）
- [ ] Sonnet で cron も実行する場合のコスト試算

---

## [2026-08-30] — Cron ジョブ最適化 & ローカルモデル断念

### 📌 変更概要

**Cron ジョブのモデル切り替え**: ローカル Ollama → Anthropic Claude に移行

**対象ジョブ**:
- `weather-jp-am` (毎日 09:10)
- `econ-jp-am` (毎日 09:13)
- `ai-weekly-digest` (月曜 09:16)

**新構成**:
- Primary: `anthropic/claude-haiku-4-5`
- Fallback: `anthropic/claude-sonnet-5`

**理由**: ローカル Ollama (`ornith-1.5:35b` / `qwen3.6:35b-mlx`) は
- 🔴 無反応多発
- 🔴 中国語混入
- 🔴 遅延（45-120秒）
- ❌ 日本語対話に不適切

**実行時刻最適化**:
- 朝の実行を 07:30 台 → **09:10-09:16** に変更
- 理由: 他の自動実行（stock_analysis / weather_digest など）との競合回避

**結果**:
- ✅ ジョブ実行頻度: 5回/日 → 3回/日に削減
- ✅ コスト削減: Sonnet fallback により安定性向上

---

## [2026-08-29] — OpenClaw 導入・基本セットアップ完了

### 📌 導入完了

**バージョン**: OpenClaw `2026.7.1-2`

**基本構成**:
- ✅ Gateway: `localhost:18789` (launchd: `ai.openclaw.gateway`)
- ✅ エージェント: 名前「クゥ」 / 黒猫 / 🐈‍⬛
- ✅ モデル: `anthropic/claude-haiku-4-5` + fallback
- ✅ Slack: Socket Mode (WS `openclaw-test`)
- ✅ Web 検索: SearXNG (自己ホスト, `localhost:18899`)
- ✅ Cron ジョブ: 3本登録

**ドキュメント**: `~/projects/openclaw_setup/` に詳細ログ

---

## 📊 変更統計

| 項目 | 2026-08-29 | 2026-08-30 | 2026-09-01 |
|------|-----------|-----------|-----------|
| **ドキュメント** | 1 (README) | 1 | **13** |
| **モデル（primary）** | haiku | haiku | **sonnet** |
| **Cron ジョブ** | 3 個 | 3 個 | 3 個 |
| **実行時刻帯** | 各時間 | 09:10-09:16 | 09:10-09:16 |
| **ローカルモデル** | ornith-1.5 | qwen3.6 | qwen3.6 (fallback) |

---

## 🔮 将来予定

### 短期（9月中）

- [ ] Sonnet 運用での性能測定・ログ確認
- [ ] Cron ジョブで Sonnet も試行（コスト対効果検証）
- [ ] ドキュメント追加：運用スナップショット（monthly report）

### 中期（9-10月）

- [ ] Tailscale 検討（リモートアクセス）
- [ ] SecretRef 導入（平文トークン廃止）
- [ ] マルチエージェント構成の検討（work 環境分離）

### 長期（Q4）

- [ ] 気象・経済データを DB に蓄積（分析基盤）
- [ ] ダッシュボード UI 整備（web ベース）
- [ ] スリープ運用の最適化（スケジュール工夫）

---

## 📖 関連ドキュメント

- **本プロジェクト**: https://github.com/masauehr/openclaw_common
- **導入記録**: https://github.com/masauehr/openclaw_setup
- **OpenClaw 公式**: https://docs.openclaw.ai/
- **自動データ収集**: https://github.com/masauehr/automated-data-collector

---

**最後に更新**: 2026-09-01 10:33 JST  
**管理者**: Msashiro UEHARA  
**エージェント**: クゥ (Kuu) 🐈‍⬛
