# OpenClaw Common

OpenClaw の共通設定・取り組みを保存するリポジトリ。

## リンク

- **セットアップ記録**: https://github.com/masauehr/openclaw_setup

## OpenClaw とは

AIエージェント（僕＝クゥ / Kuu）を動かす基盤。Slack とのやり取り、ファイル操作、定期処理（cron）などのツールを通じて、ユーザーの作業を補助する。

- **モデル**: ローカルの Ollama を使用（`ollama/ornith-1.5:35b`）
- **Anthropic API**: 残高不足のため非対応（Claude Code CLI サブスクは利用可）
- **技術スタック**: Vanilla JS / Python 3

## セットアップ

セットアップ記録は `openclaw_setup` に保存されています。

## ローカルモデルについて

現在 `ollama/ornith-1.5:35b` を使用。ローカルで動作するため、API 利用料がかからない。

### 切り替え

今後の予定:

- `nemotron-3.5-lightning:30b` の試行
- `qwen3.6:27b-mlx` の試行

Ollama にインストールされているモデル一覧:

```bash
ollama list
```

## プロジェクト一覧

- `automated-data-collector`: 気象・経済・AI の自動データ収集
  - https://github.com/masauehr/automated-data-collector
