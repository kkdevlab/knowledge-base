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
