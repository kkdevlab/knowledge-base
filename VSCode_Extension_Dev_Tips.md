## 2026-08-01: 手動フォルダコピーではVS Code拡張機能が認識されない

- **エラー内容**: `%USERPROFILE%\.vscode\extensions\` に拡張機能フォルダ（package.json + extension.js）を手動配置し、ウィンドウをリロードしても、右クリックメニュー等の拡張機能のcontributionが一切表示されない。`code --list-extensions` にも出てこない。
- **原因**: 近年のVS Code（確認バージョン: 1.131.0）は起動時に拡張機能フォルダを動的スキャンするのではなく、`extensions/extensions.json` という管理台帳に登録された拡張機能のみを読み込む。フォルダを直接コピーしただけではこの台帳に登録されないため、内容が正しくても一切認識されない。
- **解決方法**:
  1. `npm install -g @vscode/vsce` は不要。`npx --yes @vscode/vsce package` を拡張機能フォルダ内で実行し `.vsix` を生成
  2. `code --install-extension <vsixパス>` で正式インストール（`extensions.json` に自動登録される）
  3. 上書き時は `code --install-extension --force <vsixパス>`
  4. 動作確認だけしたい場合は、拡張機能フォルダを開いて `F5`（拡張機能開発ホスト）を使う方法もある（インストール不要）
- **備考**: `.vscodeignore` や `repository`/`LICENSE` フィールドが無いと `vsce package` は警告を出すが、パッケージ自体は生成される（個人用途では無視して問題ない）。
