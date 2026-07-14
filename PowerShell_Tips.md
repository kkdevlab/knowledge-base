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
