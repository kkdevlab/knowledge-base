# Windows セットアップ Tips

## Windows 11 OOBEでMicrosoftアカウントをスキップする手段が使えないケース（2026年8月時点）

- **症状**: セットアップ中の「ネットワークに接続しましょう」等の画面で`Shift+F10`を押してもコマンドプロンプトが開かない
- **原因**: Windows 11の一部ビルドでは、ローカルアカウント作成を回避させる`oobe\bypassnro`コマンド自体、あるいは`Shift+F10`によるコマンドプロンプト起動が無効化されている（Microsoftが年々この種の回避策を塞ぐ方向で仕様変更している）
- **代替策**: 回避策にこだわらず、一旦Microsoftアカウントでサインインしてセットアップを完了させ、デスクトップ到達後に「設定→アカウント→ユーザーの情報→ローカルアカウントでのサインインに切り替える」で変換する（公式にサポートされた経路）。OneDriveのフォルダ自動バックアップが有効化されていないか事後確認すること
- **備考**: HP OmniBook 7 Aero 13-bg1000AU（Windows 11 Home、2026年8月時点のプレインストール状態）で実際に発生を確認

---

## winget install の `--override` でVisual Studioのワークロード指定が効かないことがある

- **症状**: `winget install --id Microsoft.VisualStudio.2022.Community --override "--add Microsoft.VisualStudio.Workload.ManagedDesktop --includeRecommended --passive"`を実行し正常終了(exit code 0)するが、実際にはVS本体のみインストールされ、指定したワークロードが一切入らない（`vswhere -packages`で確認すると空）
- **原因**: 不明（wingetのoverride引数の受け渡しに問題がある可能性）。少なくとも2026年8月時点のwinget/VS 2022 17.14系で再現
- **解決方法**: VS Installer本体を直接使ってワークロードを追加(modify)する
  ```powershell
  & "${env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\vs_installer.exe" modify --installPath "C:\Program Files\Microsoft Visual Studio\2022\Community" --add Microsoft.VisualStudio.Workload.ManagedDesktop --includeRecommended --passive --norestart --wait
  ```
- **注意**: `vs_installer.exe modify`は`--wait`を付けても、実際のパッケージダウンロード・インストールは別プロセス(`setup.exe`)にハンドオフされ、コマンド自体はハンドオフ完了時点で終了扱いになることがある。`Get-Process -Name setup`で実プロセスの終了を待ち、`vswhere -format json`の`isComplete`/`isLaunchable`で真の完了を確認すること

---

## schtasks /run はタスクが既に実行中だと新プロセスを起動しない

- **症状**: `schtasks /create /f`でタスク定義（実行ファイルパス等）を更新した直後に`schtasks /run`で動作確認しても、更新前の設定で起動した既存プロセスがそのまま応答し、変更が反映されているように見えない
- **原因**: `schtasks /run`は対象タスクが既に実行中の場合、新しいプロセスを起動せず「currently running」を返すのみ。ログオン時トリガー等で既に起動済みのプロセスは、タスク定義を後から更新しても自動では再起動されない
- **解決方法**: 定義変更後の動作確認では、対象プロセスを`Stop-Process`等で明示的に終了させてから`schtasks /run`を実行する

---

## OneDriveの「フォルダのバックアップ」が意図せず再有効化され、リネーム中に重複フォルダができる

- **症状**: OneDriveの「バックアップの管理」でDocumentsのバックアップを一度オフにしてフォルダをリネームしても、後で（再起動・再サインイン等のタイミングで）勝手にオンへ戻ることがある。有効化されると`C:\Users\<user>\Documents`（本来のローカル実体）の中身が空になり、`OneDrive\Documents`（または日本語環境なら`OneDrive\ドキュメント`）へのショートカット(.lnk)だけが残る。この状態でフォルダ名を変更しようとすると、新旧2つの名前のフォルダが併存してしまうことがある
- **原因**: OneDriveのKnown Folder Move(KFM)は、Windows/OneDriveからの「バックアップしましょう」通知への誤承諾や、OneDriveクライアントの再起動・再同期のタイミングで再度有効化されることがある
- **対処**: リネーム作業中はOneDriveの「バックアップの管理」で都度Documentsのバックアップ状態を確認する。重複してできた空フォルダ（実データが無く`desktop.ini`とショートカットのみ）は、OneDriveが裏側で自動的に整理・削除する場合がある。手動削除しようとする前に少し待って（数分〜PC再起動後）再確認するとよい
- **備考**: フォルダリネーム自体もOneDriveの同期プロセスと衝突しやすい。対象フォルダを開いているアプリ（VS Code等）を閉じるだけでは不十分で、PCの再起動が必要になる場合がある
