# Android adb 実行時 Tips

## 2026-07-21: Git Bash経由でadb shellに渡す絶対パスがMSYSに変換されて壊れる

- **エラー内容**: `adb shell run-as ... cat /storage/emulated/0/...`のようなコマンドで、パスが`C:/Program Files/Git/storage/...`のようなWindowsパスに化けてコマンドが失敗する
- **原因**: Git Bash(MSYS2)は`/`で始まる引数を自動的にWindowsパスへ変換する。adb shellへ渡すデバイス側の絶対パスもこの変換対象になってしまう
- **解決方法**: コマンド実行前に`export MSYS_NO_PATHCONV=1`を設定してから`adb shell`を実行する
- **備考**: それでもrun-as経由のcatでは別の権限エラーが起きることがある（下記参照）。ログ等単純なファイル取得は`adb pull`を直接使う方が確実

## 2026-07-21: run-asでは自分自身のアプリの外部ストレージ(Android/data)ファイルが読めないことがある

- **エラー内容**: `adb shell run-as <pkg> cat /storage/emulated/0/Android/data/<pkg>/files/...`で"Permission denied"。同じrun-asで`id`を実行するとそのアプリのUIDになっていることは確認できる（run-as自体は機能している）
- **原因**: 不明（Android11+のスコープドストレージ・SELinux制約の可能性）。run-as経由でのcat/lsは外部ストレージ(Android/data)配下のファイルに対して拒否されるケースがある
- **解決方法**: run-asを使わず`adb pull <デバイス側パス> <ローカルパス>`を直接実行する。自分のアプリの外部ストレージファイルでも(root化・run-as不要で)pullできた
- **備考**: 同じパスに対し`adb shell ls`(run-asなし)は一覧表示だけ成功するなど挙動が一貫しない。確実な取得は`adb pull`を使う

## 2026-07-24: 実機にインストール済みのAPKが最新コードを反映しておらず、修正の動作確認が正しく行えない

- **エラー内容**: コード修正済みのはずの挙動（例: 特定条件でBroadcastを拒否する）が実機で再現せず、修正前と同じ挙動のままだった
- **原因**: 実機に数日前ビルドの古いAPKが入ったままで、当該セッションでの修正が反映されていなかった。`adb install`し直さない限り、ソースコードの変更は実機に自動反映されない
- **解決方法**: `adb shell dumpsys package <パッケージ名> | grep -E "lastUpdateTime|versionCode"`でインストール済みAPKのビルド日時を確認する。セッション中の修正より古ければ、`./gradlew assembleDebug`→`adb install -r app/build/outputs/apk/debug/app-debug.apk`（`-r`でアプリ内データは保持）で最新化してから再検証する
- **備考**: 実機での動作確認を行う前に、まずインストール済みAPKの`lastUpdateTime`を確認する習慣をつけると無駄な切り分け作業を避けられる
