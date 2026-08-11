# Python SQLite Tips

## `sqlite3.Connection`のcontext managerは接続を閉じない

`with sqlite3.connect(...) as connection:`は、正常終了時のcommitまたは例外時のrollbackを管理するが、接続自体は閉じない。

Windowsで接続中のDBを`os.replace()`しようとすると、次のような共有違反になることがある。

`[WinError 32] プロセスはファイルにアクセスできません。別のプロセスが使用中です。`

原子的な置換を行う前に接続を確実に閉じるには、`contextlib.closing()`を併用する。

## 読み取り専用ディレクトリ内のWALデータベース

WAL方式のDBを読み取り専用で開く場合でも、SQLiteが`-shm`等の補助ファイルを作ろうとして`unable to open database file`になることがある。

バックアップでは次の順序を採用する。

1. SQLite Backup APIによるオンラインバックアップを試す
2. 開けない場合はDB本体と`-wal`を書き込み可能な一時領域へコピーする
3. コピー前後のサイズ・更新日時を比較し、変更中なら再試行する
4. 一時領域のDBを開き、Backup APIで最終DBを作る
5. `PRAGMA quick_check`に成功してから原子的に置換する

単純なDB本体だけのコピーは、未チェックポイントのWAL内容を失う可能性がある。
