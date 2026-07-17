# Claude Code CLI Tips

`claude -p`（非対話モード）を外部スクリプトから呼び出す際の知見。

---

## 2026-07-17: `--effort <level>` でreasoning effortを指定できる

- **内容**: `claude -p "..." --effort <level>` でreasoning effortを指定できる（`claude --help`で確認）
- **有効な値**: `low` / `medium` / `high` / `xhigh` / `max`
- **デフォルト値の決まり方**: `--effort`を指定しない場合、`~/.claude/settings.json`の`"effortLevel"`キーの値が使われる（設定されていなければCLI自体の既定値）。Codex CLIの`~/.codex/config.toml`の`model_reasoning_effort`と同じ「ローカル設定ファイルがCLI実行時にそのまま読まれる」構造

## `--output-format json`の応答から実際に使われたモデル名を取得する

- **内容**: `claude -p "..." --output-format json`のJSON応答にはトップレベルの`"model"`フィールドが存在しない
- **取得方法**: `modelUsage`オブジェクトのキーが実際に使われたモデル名になっている
  ```json
  {"modelUsage":{"claude-sonnet-5":{"inputTokens":2,"outputTokens":3, ...}}}
  ```
- **注意**: サブエージェント等で複数モデルが使われた場合は`modelUsage`に複数キーが入る可能性がある（未検証）
