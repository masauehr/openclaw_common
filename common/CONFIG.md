# OpenClaw 共通設定

このフォルダは OpenClaw の共通設定・取り組みを保存します。

## 基本情報

- **ベースディレクトリ**: `/Users/masahiro/openclaw_prj/`
- **モデル**: `ollama/ornith-1.5:35b`（ローカル）
- **気象庁データ源**:
  1. ローカル JMA MCP（`~/projects/jma_mcp/`）— `get_daily_ranking` 等
  2. JSON API 直叩き（`amedas/data/map/{hhmmss}.json`）

## 自動データ収集プロジェクト

- `projects/automated-data-collector/`
- 気象（毎日9時・18時）、経済（毎日9時05分・18時05分）、AI（毎週月曜9時10分）
