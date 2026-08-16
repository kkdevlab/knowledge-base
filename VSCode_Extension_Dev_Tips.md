## 2026-08-01: 手動フォルダコピーではVS Code拡張機能が認識されない

- **エラー内容**: `%USERPROFILE%\.vscode\extensions\` に拡張機能フォルダ（package.json + extension.js）を手動配置し、ウィンドウをリロードしても、右クリックメニュー等の拡張機能のcontributionが一切表示されない。`code --list-extensions` にも出てこない。
- **原因**: 近年のVS Code（確認バージョン: 1.131.0）は起動時に拡張機能フォルダを動的スキャンするのではなく、`extensions/extensions.json` という管理台帳に登録された拡張機能のみを読み込む。フォルダを直接コピーしただけではこの台帳に登録されないため、内容が正しくても一切認識されない。
- **解決方法**:
  1. `npm install -g @vscode/vsce` は不要。`npx --yes @vscode/vsce package` を拡張機能フォルダ内で実行し `.vsix` を生成
  2. `code --install-extension <vsixパス>` で正式インストール（`extensions.json` に自動登録される）
  3. 上書き時は `code --install-extension --force <vsixパス>`
  4. 動作確認だけしたい場合は、拡張機能フォルダを開いて `F5`（拡張機能開発ホスト）を使う方法もある（インストール不要）
- **備考**: `.vscodeignore` や `repository`/`LICENSE` フィールドが無いと `vsce package` は警告を出すが、パッケージ自体は生成される（個人用途では無視して問題ない）。

---

## 2026-08-16: `code --install-extension`でインストールしても拡張機能が有効化されないことがある

- **エラー内容**: `code --install-extension openai.chatgpt`でインストールが「successfully installed」と表示されるが、拡張機能ビューでは無効(Disabled)状態のままで機能しない
- **原因**: VS Codeの内部データベース(`%APPDATA%\Code\User\globalStorage\state.vscdb`、SQLite)の`extensionsIdentifiers/disabled`キーに、その拡張機能IDが登録されたままになっていることがある。過去に一度その拡張機能を無効化・アンインストールした履歴がある場合に残ることがある
- **解決方法**: VS Codeの拡張機能ビュー(`Ctrl+Shift+X`)を開き、対象拡張機能の「有効にする(Enable)」ボタンを手動でクリックする。`state.vscdb`はVS Code起動中に直接書き換えると競合・破損リスクがあるため、CLIやスクリプトからの直接編集は避ける
- **確認方法**: Pythonの`sqlite3`モジュールで`SELECT value FROM ItemTable WHERE key = 'extensionsIdentifiers/disabled'`を実行し、対象拡張機能IDが含まれているか確認できる（読み取り専用の確認であれば起動中でも安全）
