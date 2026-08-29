# セットアップとモデル切り替えの試行記録

## 経緯

- 気象庁 MCP を Claude.ai のリモート版（jma-mcp-remote）と間違えていた
- ローカル版（jma-mcp）は Ollama 経由で接続済み・認証不要
- 気象庁の JSON API も直接叩ける

## ローカルモデル

- 現在: `ollama/ornith-1.5:35b`
- 今後試す:
  - `nemotron-3.5-lightning:30b`
  - `qwen3.6:27b-mlx`

## 変更方法

```bash
ollama list          # インストール済みモデル
ollama pull <name>   # モデル追加
```

OpenClaw のモデル切り替えは `/model` コマンドまたは設定で対応。
