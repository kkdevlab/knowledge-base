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

---

## 2026-07-22: Windows版はターミナル/VS Code拡張機能/Desktopアプリ(Windows+Linux VM)で4系統に分裂しうる

- **内容**: Windows上でClaude Code CLIは以下の4系統のバイナリとして独立に存在しうる
  - ターミナル版(PATH登録): `%USERPROFILE%\.local\bin\claude.exe`(ネイティブインストーラー、`installMethod:"native"`)
  - VS Code拡張機能(`anthropic.claude-code`)同梱版: `<拡張機能フォルダ>\resources\native-binary\claude.exe`
  - Claude Desktopアプリ同梱版(Windows): `%APPDATA%\Claude\claude-code\<version>\claude.exe`(バージョン別サブフォルダで管理)
  - Claude Desktopアプリ内Linux VM/サンドボックス用: `%APPDATA%\Claude\claude-code-vm\<version>\claude`(**Linux ELFバイナリ**。バックグラウンドエージェント機能用と見られる)
  - 実機確認時点(2026-07-22)でそれぞれ2.1.212 / 2.1.217 / 2.1.209 / 2.1.197 と全部異なるバージョンで共存していた
- **VS Code拡張機能の挙動**: デフォルト(`claudeCode.useTerminal: false`)ではPATH上の`claude`コマンドを使わず、同梱バイナリを直接起動する。設定`claudeCode.claudeProcessWrapper`(machine scope、実行ファイルパス直接指定)または`claudeCode.useTerminal: true`(VS Code統合ターミナル経由=PATH版を使う)で変更可能
- **確認方法**: 各拡張機能/アプリのフォルダ配下を`claude.exe`または`claude`でファイル名検索。`<binary> --version`または`<binary> doctor`で個別にバージョン確認
- **npmグローバルインストールは別枠**: `npm list -g`に出てこない。`%LOCALAPPDATA%\claude-cli-nodejs`は紛らわしいがCLI本体ではなくMCPログのキャッシュ置き場
- **対処/方針**: ターミナル版以外(VS Code拡張機能・Desktopアプリ同梱版)はそれぞれの親アプリの更新に完全に追従するため、手動で書き換える必要はない
- **確認済みバージョン**: 2026-07-22時点、上記4バージョン

---

## 2026-07-22: `~/.claude.json`の`autoUpdates:false`だけでは自動更新の有効/無効を判断できない

- **内容**: `~/.claude.json`に`"autoUpdates": false`と書かれていても、同時に`"autoUpdatesProtectedForNative": true`がセットされている場合は自動更新は実際には**有効**。バイナリ内の実装ロジックは`if (autoUpdates !== false || autoUpdatesProtectedForNative === true) 更新有効`という判定になっており、`autoUpdatesProtectedForNative`があると`autoUpdates:false`は無視される
- **原因**: ネイティブインストーラーが旧方式(npm等)からの移行を検出すると、`installMethod:"native", autoUpdates:false, autoUpdatesProtectedForNative:true`をセットで書き込む仕様。ログ文言は`Native installer: Set installMethod to "native" and disabled legacy auto-updater for protection`で、これは「旧世代の自動更新ロジックを無効化する」ためだけの処置であり、ユーザーが更新を止めたい意思とは無関係
- **確認方法**: 設定ファイルの値を直接読んで判断せず、`claude doctor`を実行して`Auto-updates: enabled/disabled`の行を見るのが確実(実際の反映結果を表示してくれる)
- **真に自動更新を止めたい場合**: 環境変数`DISABLE_AUTOUPDATER`を設定する(設定ファイルの`autoUpdates:false`だけでは不十分なケースがある)
- **確認済みバージョン**: 2.1.217 (2026-07-22)
