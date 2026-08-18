# Git（Windows環境）Tips

Windows環境でGitを使う際に遭遇した、環境固有の挙動・落とし穴をまとめる。

---

## 2026-08-18: `core.autocrlf=true`環境で`git diff --cached --check`が全行を"trailing whitespace"と誤検出する

- **エラー内容**: 新規追加した行のほぼ全てに対し、`git diff --cached --check`が`trailing whitespace.`という警告を出す（exit code 2）
- **原因**: `core.autocrlf=true`が設定されたWindowsリポジトリでは、作業ツリー上のファイルはCRLF改行になっている。`git diff --check`は改行前の`\r`をtrailing whitespaceとして検出するため、CRLF環境では追加した行すべてに警告が出やすい
- **解決方法**: 警告が出ても即座に異常と判断せず、`git show :<path>`（stageされたblobそのもの）を確認する。`cat -A`等で行末に`^M`（`\r`）が含まれていなければ、staged blob自体はLFへ正しく正規化されており、コミットしても問題ない（`core.autocrlf=true`はcommit時にCRLF→LFへ自動変換するため）
- **備考**: `git diff --cached --check`の警告は、実際にstageされる内容ではなく、コミット前の作業ツリー表示上の差分に対して評価される。CRLFリポジトリではこの警告を「真の異常（実際に不要な空白が混入している）」と区別するために、必ずstaged blob側を確認する一手間を挟む
