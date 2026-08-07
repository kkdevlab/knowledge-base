# Chrome DevTools MCP Tips

`chrome-devtools-mcp`（Model Context Protocol経由でChromeを自動操作するツール）を使う際の汎用的な注意点。

---

## 2026-08-07: 既定では、ユーザーの実ブラウザとは別の一時プロファイルで新規Chromeインスタンスを起動する

- **症状**: `.mcp.json`に`--browserUrl`等の接続先指定なしで`chrome-devtools-mcp`を設定すると、`navigate_page`で`chrome://extensions/`を開いても検索してもインストール済み拡張機能が1件も見つからない。拡張機能もログインセッションも入っていない、空のブラウザに繋がっている
- **原因**: `chrome-devtools-mcp`は、既に起動しているブラウザに接続する設定（`--browserUrl`でCDPエンドポイントを指定する等）をしない限り、**呼び出しのたびに独立した一時プロファイルで新規Chromeインスタンスを起動する**仕様。ユーザーが普段使っているブラウザ（拡張機能・ログインセッションが入ったプロファイル）とは別物になる
- **対処法**:
  - ユーザーの実際のブラウザ拡張機能の中身（ソースコード等）を調べたい場合は、DevTools経由の操作ではなく、拡張機能のインストール先を**PC上のファイルとして直接読む**方が確実で高速
    - Windows: `%LOCALAPPDATA%\Google\Chrome\User Data\<Profile>\Extensions\<拡張ID>\<version>\`
    - 拡張機能IDは`chrome://extensions/`の詳細画面（ユーザーにスクリーンショットで見せてもらう）や、拡張機能のDevTools URL（`chrome-extension://<ID>/...`）から取得できる
  - どうしても実ブラウザをchrome-devtools-mcpで操作したい場合は、ユーザーに既存Chromeを`--remote-debugging-port=9222`付きで起動し直してもらい、`--browserUrl=http://localhost:9222`を指定して接続する（既存ウィンドウをそのまま乗っ取ることはできず、再起動が必要）
- **実例**: Join（joaoapps）Chrome拡張機能のpush中継処理（`gcm.js`の`execute()`関数）の実装を調査する際、DevTools経由の全ファイル検索（Ctrl+Shift+F）が効かなかったため、拡張機能のインストールディレクトリを直接grep/readして原因を特定した
