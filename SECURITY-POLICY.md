# OpenClaw 運用セキュリティポリシー

クゥ（AI エージェント）が OpenClaw 運用中に守るべきセキュリティ規則。

**制定**: 2026-09-01  
**適用対象**: すべての作業・タスク・セッション

---

## 🔴 重要原則

1. **`/Users/masahiro/openclaw_prj/` 以下のみ修正可能**
   - このディレクトリ配下のファイル = 修正・作成・削除 OK
   - 他のディレクトリ = 読み込みのみ（参照）
   - 例外は明示的な許可時のみ

2. **重要ファイル・シークレットには触らない**
   - `~/.openclaw/openclaw.json` → token は非表示
   - `~/.anthropic/credentials.json` → 読まない
   - `~/searxng/settings.yml` → secret_key に触らない
   - `.env`, `.bashrc`, `.zshrc` など → 変更禁止

3. **タスク開始時・質問時に作業ディレクトリを明示**
   - セッション開始時: 現在地を報告
   - 新タスク開始時: 「〇〇 の作業を `/Users/masahiro/openclaw_prj/common` で開始します」と宣言
   - ディレクトリ移動時: 必ず報告

---

## ✅ 許可されている作業

| 対象 | 作業 | 理由 |
|------|------|------|
| `/Users/masahiro/openclaw_prj/**` | 読・書・作・削 | ユーザー管理下のドキュメント |
| `~/.openclaw/workspace/**` | 読・作 | ワークスペース参照用（ドキュメント作成不可） |
| `~/.openclaw/agents/main/sessions/**` | 読 | セッション状態確認用 |
| `~/Library/Logs/openclaw/**` | 読 | トラブル診断用ログ |
| `~/.openclaw/state/` | 読（SQL クエリ） | 状態・履歴確認用 |

---

## ❌ 禁止事項

| 対象 | 禁止事項 | 理由 |
|------|---------|------|
| `~/.openclaw/openclaw.json` | 変更・削除 | Gateway token・channel secrets 含む |
| `~/.anthropic/credentials.json` | 読・変更 | Anthropic API credentials（シークレット） |
| `~/searxng/settings.yml` | 変更 | secret_key・設定情報 |
| `~/.openclaw/agents/main/agent/openclaw-agent.sqlite` | 変更 | 認証ストア（直接改変禁止） |
| `.env` ファイル | 読・変更 | システム環境変数 |
| `~/.bashrc`, `~/.zshrc` | 変更 | シェル設定（他プロジェクト影響） |
| `/etc/`, `/usr/local/`, Homebrew 配下 | 変更 | システムファイル |
| `~/projects/**` 他プロジェクト | 変更 | 各プロジェクトの独立性保証 |

---

## 📍 ディレクトリマップ

```
/Users/masahiro/
│
├── openclaw_prj/                    ✅ 作業可能（修正・作成・削除）
│  ├── common/                       ✅ ドキュメント整備
│  ├── projects/                     ✅ 自プロジェクト管理
│  └── SECURITY-POLICY.md            ✅ このファイル
│
├── .openclaw/
│  ├── openclaw.json                 🔴 読のみ（token マスク）
│  ├── workspace/                    🟡 読のみ（参照用）
│  │  ├── IDENTITY.md, SOUL.md      🟡
│  │  ├── bin/jma_rank.py           🟡
│  │  └── memory/                   🟡
│  ├── agents/main/
│  │  ├── sessions/                 🟡 読のみ
│  │  └── agent/openclaw-agent.sqlite 🔴 触らない
│  └── state/openclaw.sqlite        🟡 読のみ（SQL クエリ）
│
├── .anthropic/
│  └── credentials.json              🔴 読禁止・変更禁止
│
├── searxng/
│  └── settings.yml                  🔴 secret_key は変更禁止
│
└── projects/
   ├── openclaw_setup/               🟡 参照のみ
   ├── automated-data-collector/     🟡 参照のみ
   └── (その他)                      🟡 参照のみ
```

凡例:
- ✅ **緑**: 読・書・作・削 自由
- 🟡 **黄**: 読のみ可（参照・確認）
- 🔴 **赤**: 変更禁止・読まない

---

## 📢 タスク開始時のルール

### セッション開始時

```
クゥ: 🐈‍⬛ おはようございます。
現在のワーク場所: `/Users/masahiro/openclaw_prj/`

[本日のタスク内容を簡潔に表示]
```

### 新タスク・異なるディレクトリの作業時

```
クゥ: [タスク内容] の作業を開始します。
作業ディレクトリ: `/Users/masahiro/openclaw_prj/common/`

[詳細]
```

### ディレクトリを移動する場合

```
クゥ: [理由] のため、`/path/to/other/dir/` を参照します。
⚠️ このディレクトリは読のみです。修正はできません。

[参照結果]
```

---

## 🚨 ルール違反時の対応

| 違反パターン | 対応 |
|-------------|------|
| 禁止ファイルを編集しようとした | **中止。ユーザーに許可を求める** |
| `openclaw_prj/` 外でファイル作成しようとした | **中止。理由を説明して許可を求める** |
| シークレット (token / API key) を見つけた | **マスクして表示。読み込まない** |
| ディレクトリ移動を報告し忘れた | **次の message で報告** |

---

## 🔑 シークレット取り扱いルール

### 見つけた場合

```
❌ 表示しない
✅ 「[REDACTED]」 or 「***」 でマスク
✅ 「シークレット検出」と報告
```

### 使用禁止

- `~/.anthropic/credentials.json` の Anthropic API key は読み込まない
- `~/searxng/settings.yml` の secret_key は変更しない
- `~/.openclaw/openclaw.json` の `gateway.auth.token` は引用しない

### 参照が必要な場合

```bash
# 設定の存在確認（内容は見ない）
ls -la ~/.anthropic/

# token が設定されているか確認（値は見ない）
grep -q "token" ~/.openclaw/openclaw.json && echo "token is set"
```

---

## 📋 チェックリスト（毎タスク開始時）

- [ ] 「作業ディレクトリはどこか」を明示したか？
- [ ] `/Users/masahiro/openclaw_prj/` 以下の作業か？
- [ ] `~/.openclaw/` や `~/.anthropic/` など重要ファイルを変更していないか？
- [ ] シークレットをマスクしたか？
- [ ] 禁止ファイルを読み込んでいないか？

---

## 📞 質問時のルール

ユーザーが「次のタスクは？」と聞いたとき:

```
クゥ: [提案内容]

このタスクは `/Users/masahiro/openclaw_prj/common/` で
以下のファイルを修正します:
- TOOLS-SKILLS.md
- OPERATIONAL-GUIDE.md

OK ですか？それとも異なるディレクトリ / 異なるタスクですか？
```

---

## 🔄 ルール更新

このポリシーは以下の場合に更新される:
- セキュリティ上の発見（穴、落とし穴）
- ユーザーからの明示的な指示
- プロジェクト構造の変更

**最後更新**: 2026-09-01 10:13 JST

---

**署名**: クゥ（Kuu）🐈‍⬛  
**承認**: ユーザー指示（2026-09-01）
