## 2026-07-13: Sort-Object で複数プロパティを昇順/降順混在で指定する

- **エラー内容**: `Sort-Object date -Descending, title` のように書くと、`-Descending`スイッチの後にカンマ区切りで複数プロパティを渡そうとしてもパースエラーにはならないが、意図通りに動作しない（構文として誤り）
- **原因**: `-Descending`はスイッチパラメータであり、値を取らない。カンマ区切りの複数プロパティを渡す書き方ではない
- **解決方法**: ハッシュテーブル形式で個別に降順/昇順を指定する
  ```powershell
  Sort-Object -Property @{Expression='date'; Descending=$true}, @{Expression='title'; Descending=$false}
  ```

---

## 2026-07-13: ConvertTo-Json で配列が1件のとき配列でなくオブジェクトになる

- **エラー内容**: `$array | ConvertTo-Json` で `$array` の要素が1件だけの場合、出力が `[{...}]` ではなく `{...}` になる
- **原因**: PowerShellの `ConvertTo-Json` はパイプライン入力を1つずつ処理し、要素数が1つだと単一オブジェクトとして扱う仕様
- **解決方法**: `-AsArray` オプションを付ける（PowerShell 6.2+）
  ```powershell
  $array | ConvertTo-Json -AsArray
  ```

---

## 2026-07-13: パイプラインの0件結果を変数へ代入すると$nullに潰れる

- **エラー内容**: `$x = $collection | Where-Object {...}` で条件に一致する要素が0件のとき、`$x`は空配列ではなく`$null`になる。後続で`$x.Count`を参照すると「nullのメソッド/プロパティにアクセスできない」エラーになる
- **原因**: PowerShellのパイプライン代入は、出力が0件または1件のとき配列をアンラップする仕様がある
- **解決方法**: 単項カンマ演算子で強制的に配列として返す/受け取る
  ```powershell
  function Get-Filtered {
      $filtered = @($collection | Where-Object {...})
      return ,$filtered  # 0件・1件でも呼び出し元は必ず配列として受け取れる
  }
  ```

---

## 2026-07-13: ConvertTo-Json で配列が0件のとき、-AsArrayを付けても$nullになる

- **エラー内容**: `@() | ConvertTo-Json -AsArray` のように要素数0の配列を渡しても、出力が空配列`[]`ではなく`$null`になる
- **原因**: `ConvertTo-Json`はパイプラインに何も渡されないと本体処理が実行されず、`-AsArray`を付けても救済されない（「1件→オブジェクト化」問題とは別の挙動）
- **解決方法**: 0件のときは明示的に`'[]'`を代入し、ConvertTo-Jsonを呼ばない

  ```powershell
  if ($array.Count -eq 0) { $json = '[]' } else { $json = $array | ConvertTo-Json -AsArray }
  ```

- **備考**: 「1件→オブジェクト化」エントリとセットで扱うこと。`-AsArray`は1件のケースは救うが0件のケースは救わない

---

## 2026-07-13: 名前付きMutexは同一スレッド内で再入可能なため、同一プロセス内の多重起動テストは偽陽性になる

- **エラー内容**: 同一PowerShellセッション内で先に`Mutex.WaitOne(0)`を呼んでから`& script.ps1`を実行すると、スクリプト内の`WaitOne(0)`が成功してしまい多重起動が検知されない
- **原因**: named Mutexは「スレッド単位」で所有権を持つ。同一スレッドは別のMutexオブジェクトのインスタンス経由でも再取得に成功する（reentrant lock）。`&`はスクリプトを新しいプロセスではなく同一スレッド上で実行する
- **解決方法**: 多重起動テストは必ず`Start-Process`等で別プロセスにMutexを保持させ、テスト対象は元のプロセスから呼び出す

---

## 2026-07-13: System.Diagnostics.Process の StandardInput は既定でConsole.InputEncoding依存になる

- **エラー内容**: 子プロセス（例: `codex exec`）にプロンプト文字列を`$process.StandardInput.Write($Prompt)`で渡したところ、対話ターミナルから手動実行すると成功するのに、スケジューラ/外部トリガー（Tasker/Join等）経由の非対話起動だと子プロセス側で「invalid byte at offset 0」のような文字化けエラーになる
- **原因**: `ProcessStartInfo.StandardOutputEncoding`/`StandardErrorEncoding`をUTF-8に指定していても、`StandardInputEncoding`を明示しないと.NETは`Console.InputEncoding`（実行コンテキスト依存のコンソールコードページ）でエンコードする。対話ターミナルではUTF-8相当のことが多いが、非対話バックグラウンド起動ではOS既定コードページ（日本語Windowsだとcp932等）にフォールバックすることがあり、非ASCII文字を送ると子プロセス側でUTF-8として不正なバイト列になる
- **解決方法**: 標準入力にも明示的にエンコーディングを指定する

  ```powershell
  $psi.StandardInputEncoding = [System.Text.Encoding]::UTF8
  ```

- **備考**: `StandardOutputEncoding`/`StandardErrorEncoding`だけ設定して満足しがちだが、stdinへの書き込みがある場合は3つとも揃える必要がある

---

## 2026-07-13: 子プロセスのカレントディレクトリは既定で親プロセスから引き継がれる（ProcessStartInfo.WorkingDirectory未指定時）

- **エラー内容**: `codex exec`をSystem.Diagnostics.Processで起動する際、`WorkingDirectory`を指定しなかったところ、手動でgitリポジトリ内から実行したときは成功するが、Tasker/Join等の外部トリガー経由で起動すると「Not inside a trusted directory and --skip-git-repo-check was not specified.」で失敗する
- **原因**: `ProcessStartInfo.WorkingDirectory`を指定しないと、子プロセスは親プロセス（PowerShell）のカレントディレクトリをそのまま引き継ぐ。外部トリガー経由だと親プロセスのカレントディレクトリがgit作業ツリー外になりがちで、git作業ツリー内であることを前提とするCLI（codex等）のチェックに失敗する
- **解決方法**: 呼び出し元のカレントディレクトリに依存させず、明示的に固定する

  ```powershell
  $psi.WorkingDirectory = $PSScriptRoot  # または既知の固定パス
  ```

- **備考**: 呼び出し元（スケジューラ・外部トリガー）側で`cd`させる対処もあるが、呼び出し元が増えるたびに同じ対応が必要になるため、子プロセスを起動する側で固定する方が再発防止になる

---

## 2026-07-14: Windows PowerShell 5.1でBOMなしUTF-8スクリプトが文字化けし構文エラーになる

- **エラー内容**: Windowsタスクスケジューラで`powershell.exe`（Windows PowerShell 5.1）を実行エンジンに指定すると、日本語の文字列リテラルを含むスクリプトが構文エラーになり、実行前に失敗する（タスクの`LastTaskResult=1`、スクリプト側のログ出力すら残らない）
- **原因**: スクリプトファイルがBOMなしUTF-8で保存されており、かつ実行環境の既定コードページが日本語（Shift-JIS）の場合、Windows PowerShell 5.1はBOMなしスクリプトをUTF-8ではなくシステムの既定コードページとして読み込む。マルチバイト文字（日本語）がShift-JISとして誤読され、文字列リテラルの閉じクォート位置がずれて構文エラーになる
- **解決方法**: スクリプトの実行には`pwsh.exe`（PowerShell 7系）を明示的に使う。PowerShell 7はBOMの有無に関わらずスクリプトファイルをUTF-8として読み込むため、この問題が起きない

  ```powershell
  # NG: Windows PowerShell 5.1（コードページ依存）
  powershell.exe -File "C:\path\to\script.ps1"

  # OK: PowerShell 7系（常にUTF-8として読み込む）
  "C:\Program Files\PowerShell\7\pwsh.exe" -NoProfile -ExecutionPolicy Bypass -File "C:\path\to\script.ps1"
  ```

- **備考**: Windowsタスクスケジューラでスクリプトを登録する際、`Execute`欄が既定で`powershell.exe`になっているケースがあるため要注意。日本語コメント・文字列を含むBOMなしUTF-8スクリプトを外部トリガー（タスクスケジューラ・他言語からの呼び出し等）から実行する場合は、実行エンジンが5.1系でないか必ず確認する

---

## 2026-07-15: タスクスケジューラーでpwshのコンソールウィンドウを非表示にする

- **内容**: Windowsタスクスケジューラーのアクションで`pwsh.exe`（PowerShell 7）を直接呼び出すと、実行時にコンソールウィンドウが表示される
- **解決方法**: Argumentsの先頭に`-WindowStyle Hidden`（短縮形`-w Hidden`）を追加する

  ```text
  -WindowStyle Hidden -NoProfile -ExecutionPolicy Bypass -File "C:\path\to\script.ps1"
  ```

- **確認方法**: `pwsh.exe -?` で `-WindowStyle | -w` オプションの存在を確認できる
- **備考**: `.ps1`ファイル自体の変更は不要。既存タスクは`Set-ScheduledTaskAction`でActionのArgumentsを更新すればよい

---

## 2026-07-17: ProcessStartInfo.ArgumentListは.NET Framework版PowerShell(5.1)では使えない

- **内容**: `System.Diagnostics.ProcessStartInfo`の`ArgumentList`プロパティ（引数をエスケープ不要で個別に追加できるコレクション）は.NET Core/.NET 5+ベースのAPIで、Windows標準の`powershell.exe`（.NET Framework 4.x・PowerShell 5.1）では利用できない
- **対処**: 外部プロセスをArgumentList経由で起動するスクリプトは、`powershell.exe`ではなく`"C:\Program Files\PowerShell\7\pwsh.exe"`を明示的に指定して実行する
- **注意**: Taskerなどのオートメーションから`powershell -File ...`のような旧来のコマンドをそのまま流用すると、この非互換に気づかず実行時エラーになる（起動元のコマンド文字列も含めて`pwsh.exe`に揃える必要がある）

---

## 2026-07-17: タスクスケジューラのCommand/Argumentsを正しく分離しないとFILE_NOT_FOUNDになる

- **エラー内容**: `New-ScheduledTaskAction`や`schtasks`でスペースを含む実行パス（例: `C:\Program Files\PowerShell\7\pwsh.exe -WindowStyle Hidden ...`）をまとめて1つの文字列として渡すと、登録後のタスクXMLで`<Command>C:\Program</Command>`・`<Arguments>Files\PowerShell\7\pwsh.exe -WindowStyle Hidden ...</Arguments>`のように最初のスペースで分割されてしまう。実行するたびに`LastTaskResult: 2147942402`（0x80070002 = ERROR_FILE_NOT_FOUND）で即失敗する
- **原因**: タスクスケジューラのAction（Exec）は`Command`と`Arguments`が別フィールドであり、実行パス全体を`Command`側に丸ごと渡すと最初の空白で機械的に分割されてしまう
- **解決方法**: `New-ScheduledTaskAction -Execute '<実行ファイルのフルパス>' -Argument '<オプション文字列>'`のように、実行ファイルパスとオプションを最初から別々の引数として渡す
  ```powershell
  $action = New-ScheduledTaskAction -Execute 'C:\Program Files\PowerShell\7\pwsh.exe' -Argument '-WindowStyle Hidden -NoProfile -File "C:\path\to\script.ps1"'
  ```
- **注意**: `schtasks /query /tn "<タスク名>" /xml`でタスクの実際の`<Command>`/`<Arguments>`を確認できる。似た症状（`0x80070002`）でもmemory/troubleshooting.mdの2026-04-23エントリ（実行ファイル名のみ指定でPATH解決に失敗）とは原因が異なるので、まずXMLで分割位置を確認してから切り分けること

---

## 2026-07-17: New-ScheduledTaskSettingsSetのパラメータ名は反転したブーリアン命名になっている

- **内容**: `New-ScheduledTaskSettingsSet`には`-DisallowStartIfOnBatteries`や`-StopIfGoingOnBatteries`という直感的な名前のパラメータは存在しない。実際は`-AllowStartIfOnBatteries`（指定しなければ既定でdisallow=true）・`-DontStopIfGoingOnBatteries`（指定しなければ既定でstop=true）という反転した命名になっている
- **確認方法**: `Get-Command New-ScheduledTaskSettingsSet -Syntax`でパラメータ一覧を確認できる
- **対処**: 既定のバッテリー挙動（バッテリー駆動中は開始しない・バッテリー駆動に切り替わったら停止する）を維持したい場合は、これらのパラメータを何も指定しなければ良い（既定値がそのまま望ましい動作になっている）

---

## 2026-07-17: -WindowStyle Hiddenは既定ターミナルがWindows Terminalの環境では効かないことがある

- **内容**: `pwsh.exe -WindowStyle Hidden -File script.ps1`をタスクスケジューラから実行しても、Windows 11で既定ターミナルアプリがWindows Terminal（`wt.exe`）に設定されている環境では、コンソールウィンドウ（タイトルに実行コマンドラインがそのまま表示される）が表示され続けることがある
- **原因**: Windows Terminalへのターミナル委任（Default Terminal Application）が有効な環境では、`-WindowStyle`によるウィンドウ非表示リクエストが確実に反映されない
- **対処**: `conhost.exe --headless`で回避できる場合があるが、環境によっては`conhost.exe`自体が存在しない（本環境で確認）。確実な代替として、VBScriptの`WScript.Shell.Run`（第2引数に`0`=`SW_HIDE`を指定）でWin32レベルの完全非表示起動を行う
  ```vbscript
  Set objShell = CreateObject("WScript.Shell")
  objShell.Run """C:\Program Files\PowerShell\7\pwsh.exe"" -NoProfile -File ""C:\path\to\script.ps1""", 0, True
  ```
  タスクスケジューラのActionは`wscript.exe "<このvbsのパス>" <引数>`のように、直接ではなくvbs経由で起動する
- **注意**: 2026-07-15に「`-WindowStyle Hidden`で解消」と記録したが、それは本環境（Windows Terminalが既定ターミナル）では不十分だったことが今回判明した。`-WindowStyle Hidden`は環境によっては効くこともあるため、まず試してよいが、それでも表示される場合はVBScript方式に切り替える

---

## 2026-07-17: Invoke-RestMethodのcatchブロックで$_.ErrorDetailsがnullになりStrictMode下で例外が握りつぶされる

- **エラー内容**: `Invoke-RestMethod`が失敗した際の`catch`ブロックで`$_.ErrorDetails.Message`にアクセスしたところ、`PropertyNotFoundException: The property 'Message' cannot be found on this object`が発生し、本来報告したかったエラー内容（実際の原因）が完全に隠れてしまった
- **原因**: `$_.ErrorDetails`はHTTPエラーレスポンス（本文付き）がある場合のみ設定され、タイムアウトや接続エラーなどレスポンス本文が存在しない失敗では`$null`になる。`Set-StrictMode -Version Latest`環境では、`$null`に対するプロパティアクセス（`$null.Message`）がエラーとして扱われる
- **解決方法**: `$_.ErrorDetails`が存在するかを先にチェックする
  ```powershell
  $errorDetail = if ($_.ErrorDetails) { $_.ErrorDetails.Message } else { $null }
  if (-not $errorDetail) { $errorDetail = $_.Exception.Message }
  ```
- **注意**: `$_.ErrorDetails.Message`を直接使っているコードはStrictMode環境では同じ危険がある。HTTPレスポンス以外の失敗（タイムアウト等）が起きうるすべての`Invoke-RestMethod`/`Invoke-WebRequest`のcatchで同様のガードを入れること

---

## 2026-08-03: Set-StrictMode下でParser::ParseFileに未宣言変数へ[ref]を渡すとエラーになる

- **エラー内容**: `[ref] cannot be applied to a variable that does not exist.`
- **原因**: `Set-StrictMode -Version Latest`が有効な状態で、`[System.Management.Automation.Language.Parser]::ParseFile($path, [ref]$tokens, [ref]$errors)`のように、事前に`$errors`変数を宣言せず`[ref]`へ直接渡すと、StrictModeが「存在しない変数への参照」としてエラーにする
- **解決方法**: 呼び出し前に`$errors = $null`等で変数を明示的に初期化してから`[ref]$errors`を渡す
- **備考**: 構文解析のみを目的とした一時スクリプトでも発生するため、Parserクラスを使う一時検証コードでは特に注意

---

## 2026-08-03: PowerShellで[int]$nullは例外を投げず0になる（オプション項目の欠如を無音で握りつぶす）

- **エラー内容**: なし（エラーは出ないが、意図しない値になる）
- **原因**: ハッシュテーブル・YAML等から読んだ任意項目のキーが存在しない場合、`$hashtable['key']`は`$null`を返す。`Set-StrictMode`は未宣言"変数"は検知するが、ハッシュテーブルの存在しないキー参照は検知しない。その`$null`を`[int]`へキャストすると例外を投げず`0`になる
- **解決方法**: 任意項目を数値として使う場合、`if ($hashtable.ContainsKey('key') -and $hashtable['key']) { [int]$hashtable['key'] } else { <既定値> }`のように、キャスト前に存在確認と既定値へのフォールバックを明示する
- **備考**: `PromptRunner`の`timeoutSec`省略時に「即タイムアウトする」不具合として実際に発見。タイムアウト秒数のように「0だと即座に破綻する」値を扱う箇所は特に注意
