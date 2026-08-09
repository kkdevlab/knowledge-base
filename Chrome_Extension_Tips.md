# Chrome Extension Tips

Chrome拡張機能（Manifest V3）開発で遭遇した挙動・回避策のまとめ。

---

## 2026-08-09: File System Access APIのrequestPermission()はユーザー操作中でないとSecurityErrorになる

- **エラー内容**: `Uncaught (in promise) SecurityError: Failed to execute 'requestPermission' on 'FileSystemHandle': User activation is required to request permissions.`
- **原因**: `FileSystemHandle.requestPermission()`は仕様上「ユーザー操作(トランジェントアクティベーション)」中でないと呼び出せない。拡張機能のポップアップが開いた直後の自動初期化処理内などで呼ぶとブロックされる。`queryPermission()`はユーザー操作不要
- **解決方法**: ポップアップ起動時は`queryPermission()`のみ、または保存済みハンドルの`.name`表示（権限不要で取得可能）にとどめ、`requestPermission()`はボタンクリック等のユーザー操作ハンドラ内でのみ呼ぶ
- **備考**: PageToMarkdown拡張機能（File System Access API + IndexedDBでフォルダハンドルを永続化）の再実装時に発見
