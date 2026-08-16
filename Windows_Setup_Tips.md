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

---

## OneDriveの「重要なフォルダーのバックアップ」の対象範囲とデスクトップの表示の仕組み

- **仕様**: OneDriveの「重要なフォルダーのバックアップ」(KFM)はDesktop/Documents/Picturesの3つのみが対象。`%LocalAppData%`・`%APPDATA%`・`%UserProfile%`直下は対象外
- **確認方法**: レジストリ `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders` の `Desktop`値でOneDrive配下にリダイレクトされているか確認できる
- **デスクトップの見え方の罠**: エクスプローラーのデスクトップ画面には、個人の`OneDrive\Desktop`と`C:\Users\Public\Desktop`(全ユーザー共通)の中身が重ねて表示される。「デスクトップがパブリックになっている」ように見えても、実際は2つのフォルダの表示が重畳しているだけで、個人のOneDriveデスクトップは正しく機能していることが多い
- **アプリのインストール時の分岐**: インストーラーで「全ユーザー用」を選ぶ(または管理者権限で実行)と、ショートカットはPublicデスクトップ・共通スタートメニュー(`C:\ProgramData\...`)に作成され、インストール先も`Program Files`直下になる。「自分のみ」を選ぶと個人デスクトップ・`AppData\Local`配下になる
- **注意**: アプリの設定・データ(`%LocalAppData%`等)自体もこのバックアップ対象外のため、個人データの保護が必要なアプリは別途対応が必要（kklabではgit root直下`config/`・`logs/`への保存に統一する方針。詳細: `kklab-rules/csharp/summary.md` 4.1章）

---

## `python3`コマンドがWindows Store実行エイリアスに捕まり「見つかりません」と出る

- **症状**: `python3 --version`を実行すると`Python was not found; run without arguments to install from the Microsoft Store...`と表示される。Pythonは実際にはインストール済み（`python --version`は正常に動く）
- **原因**: Windows標準の「アプリ実行エイリアス」機能が`python3.exe`という名前のスタブを`WindowsApps`配下に用意しており、実体のPythonが`python.exe`としてしかPATHに登録されていない環境ではこのスタブが先に呼ばれてしまう
- **対処**: このマシンでは`python3`ではなく`python`を使う

---

## OneDriveとNTFSジャンクション: 方向によって挙動が真逆になる

- **OneDrive内→外方向は不可**: OneDriveフォルダ「内」にjunctionを置き外部の実フォルダを参照する構成は、作成直後は正常に見えても、Windowsの`FindFirstChangeNotification`APIの制約でOneDriveがjunction先の変更を検知できなくなり、後日（実例では数日後）静かに通常フォルダへ差し戻される
- **OneDrive外→内方向は可**: OneDriveフォルダの「外」にjunctionを置き、OneDrive「内」の実フォルダを参照する構成は問題なく機能する。OneDrive自身のツリー内にreparse pointが一切現れないため、上記の不具合が構造的に発生しない。junction経由の書き込みは即座にOneDrive側の実体に反映され、OneDriveも数秒でCloud Filter API経由の新規ファイルとして認識する（複数マシンでの運用実績あり）
- **Files On-Demand関連**: `attrib +p /s <path>`でフォルダを再帰的に「常にこのデバイスに保持する」（ピン留め）状態にできる。ピン留め済みファイルも`ReparsePoint`属性を持つのは正常（OneDriveのCloud Filter APIによる管理マーク。クラウド専用＝未ダウンロードを意味しない）

---

## Gitリポジトリ配下の個別ファイルへのNTFSハードリンクは`git checkout`/`stash`/`rebase`で切れる

- **症状**: `New-Item -ItemType HardLink`で作成した個別ファイルへのハードリンク（例: Gitリポジトリ内のファイルと、リポジトリ外の別パスを同一実体化する構成）が、作成直後は`fsutil hardlink list`で双方向のパスを正しく表示するのに、`git checkout`（ブランチ切り替え）・`git stash pop`・`git rebase`のいずれを実行した後には片方のパスしか表示されなくなる（`fsutil hardlink list`で1行しか出ない＝実体が分離した状態）
- **原因は2つ複合**:
  1. `core.autocrlf=true`の場合、gitがcheckout時にLF→CRLF変換のためファイル内容を書き直す。内容の実質差分はなくても、書き込みによって新しいファイル実体が作られハードリンクが切れる
  2. `git rebase`はコミットを一度リセットして再生成するため、そのコミットで追加(`A`)されたファイルは「新規作成」として書き込まれる。この場合`.gitattributes`で`text eol=lf`を指定して改行コード変換を止めても、新規作成自体は避けられずハードリンクは切れる
- **対処**: 改行コード変換対策として対象パスに`.gitattributes`で`* text eol=lf`を指定するのは無駄ではないが、rebase等によるファイル再生成には無力。個別ファイルの同一実体化が目的なら、**ハードリンク/シンボリックリンクではなく片方向コピー**（コミット前に都度`Copy-Item`等で上書き）を使う方が確実
- **junctionとの違いに注意**: ディレクトリ単位のNTFS junctionは「OneDrive外→OneDrive内」方向なら安定して機能する（上記参照）。これは対象がディレクトリであり、git操作がその中の個別ファイルを書き換えてもjunctionという入れ物自体は影響を受けないため。個別ファイルへのハードリンクとは別物として扱うこと

---

## icaclsは孤立SID（削除済みユーザー/グループ）への`/remove`を、管理者権限でも無言で失敗させる

- **内容**: `icacls <path> /remove:g <SID文字列>`を、既に削除されたローカルユーザー/グループのSIDに対して実行すると、エラーメッセージなしに"Successfully processed 1 files"等の成功風メッセージを返すが、実際には対象ACEは削除されない。PowerShellの`Set-Acl`も`SeSecurityPrivilege`権限不足で失敗する。管理者権限PowerShellでも同様に失敗する
- **正解**: 確実な削除方法はエクスプローラーGUI（対象フォルダのプロパティ→セキュリティ→詳細設定）からの手動削除のみ
- **注意**: 孤立SIDかどうかは`Get-Acl`で`IdentityReference.Value`がSID文字列のまま返ることで判別できる（名前解決できていれば`ドメイン\ユーザー名`形式になる）
- **確認済みバージョン**: Windows 11（2026-08-16時点）

---

## Claude Code終了後も独立して動くテストスクリプトはWindowsタスクスケジューラー経由で実行する

- **内容**: Claude Code（やその他CLIツール）がBashツール等で起動するバックグラウンドプロセスは、そのセッションのプロセスツリーに紐づいており、セッション終了時に道連れで終了してしまう。「アプリを完全に終了させた状態」を独立して検証したい場合、この方法では監視プロセスも一緒に消えてしまう
- **正解**: `schtasks /Create /TN <name> /TR "powershell.exe -File <script.ps1>" /SC ONCE /ST <dummy-time> /F`で管理者権限不要のワンタイムタスクを作成し、`schtasks /Run /TN <name>`で起動する。タスクスケジューラーが実行するプロセスは呼び出し元セッションから独立しているため、Claude Code終了後も動き続ける

---

## DENY(DC)フラグ付きACEがあっても、対象への明示的なALLOWフルコントロールを持つユーザーには実害がないことがある

- **内容**: `Everyone:(CI)(DENY)(DC)`（FILE_DELETE_CHILD拒否）が親フォルダに付与されていても、ユーザー自身が対象ファイル・親フォルダに明示的な`(F)`Allowエントリを持っていれば、そのユーザーの各種アプリ（PowerShell・エクスプローラー等）経由の削除は成功する
- **注意**: DCフラグは「Everyoneグループとしての親経由削除」のみを拒否するものであり、個別ユーザーの直接DELETE権限までは阻害しない。「DENY ACEがある＝削除できない」と早合点せず、実際に削除を試して実害の有無を確認すること
