# Tasker XML / Description フォーマット対応表

Tasker の「Task Description」（公式エクスポート機能で出力される疑似コード）と XML バックアップの対応関係をまとめたリファレンス。

Tasker でタスクの修正提案を行う際、Description フォーマットで疑似コードを記述するために使用する。

---

## 参考: アクションコード・演算子一覧（外部リファレンス）

XML の `<code>` タグやConditionList の `<op>` コードの対応表は、以下の公開リソースを参照。

- **Tasker XML Codes**: https://github.com/Taskomater/Tasker-XML-Info/blob/master/Tasker_XML_Codes.md
- **Tasker XML Info（リポジトリ）**: https://github.com/Taskomater/Tasker-XML-Info

本ドキュメントでは、上記に載っていない **Description フォーマットの書き方ルール** と **XML arg ↔ Description パラメータの対応表** に焦点を当てる。

---

## Description フォーマットの書き方ルール

### 1. タスクヘッダ

先頭行にタスク名を記載する。

```
Task: タスク名
```

### 2. ラベル（セクション区切り）

Anchor（code 300）にラベルを付けてセクション区切りとして使う。

**タスク名ラベル（cyan）** — 先頭に配置し、タスクの開始点を示す:

```
<<font color=cyan>タスク名</font>>
A1: Anchor
```

**セクションラベル（green）** — 処理ブロックの区切りや Goto のジャンプ先:

```
<<font color=green>SECTION_NAME</font>>
A10: Anchor
```

**アクションに付くコメントラベル** — Anchor 以外のアクションに付けるコメント:

```
<※ ここに到達 = 異常系>
A21: Variable Set [
      Name: %result
      To: error ]
```

### 3. アクション番号

`A1:` から連番で振る。途中追加しても連番を振り直す。

```
A1: Anchor
A2: Variable Set [...]
A3: If [...]
```

### 4. パラメータの記載

`[` `]` 内に各パラメータを改行・インデントして記載する。デフォルト値のみのパラメータは省略可。

```
A3: Variable Set [
     Name: %var_name
     To: value
     Structure Output (JSON, etc): On ]
```

### 5. 無効化アクション

`[X]` プレフィクスを付ける。XML では `<on>false</on>` に対応。

```
A9: [X] Variable Set [
     Name: %gps_on
     To: 0 ]
```

### 6. 条件付きアクション

パラメータの `]` の後、インデントして `If [ 条件 ]` を記載。

```
A4: Variable Set [
     Name: %body
     To: %par2 ]
    If  [ %par2 Set & %par2 !~ ^% ]
```

### 7. If / For ブロック内のインデント

ブロック内のアクションはインデントを1段下げる。

```
A7: If [ %method ~ query ]

    A8: Variable Set [
         Name: %url
         To: https://example.com/api ]

A9: End If
```

### 8. 複数条件

`&`（AND）、`|`（OR）で接続する。

```
A2: If [ %LOC_HOME = 0 & %DEBUG_MODE = 0 ]
A38: If [ %http_response_code !Set | %http_response_code ~ ^% ]
```

### 9. 条件演算子の Description 表記

| Description 表記 | 意味 |
|-----------------|------|
| `~` | パターンマッチ |
| `!~` | パターン不一致 |
| `<` | 数値: 未満 |
| `>` | 数値: より大きい |
| `=` | 数値: 等しい |
| `!=` | 数値: 等しくない |
| `Set` | 変数がセット済み |
| `!Set` | 変数が未セット |

※ XML の op コードとの対応は Tasker-XML-Info を参照。

---

## XML arg ↔ Description パラメータ対応表

### Variable Set（code 547）

```
A3: Variable Set [
     Name: %var_name
     To: value
     Do Maths: On
     Structure Output (JSON, etc): On ]
```

| XML arg | パラメータ名 | 値の意味 |
|---------|-------------|---------|
| arg0 | Name | 変数名 |
| arg1 | To | セットする値 |
| arg3 | Do Maths | 1=On（数式として評価） |
| arg6 | Structure Output | 1=On |

### Variable Split（code 590）

```
A2: Variable Split [
     Name: %var_name
     Splitter: , ]
```

| XML arg | パラメータ名 |
|---------|-------------|
| arg0 | Name |
| arg1 | Splitter |

### Goto（code 135）

```
A11: Goto [
      Type: Action Label
      Label: <font color=green>SECTION_NAME</font> ]
```

| XML arg | パラメータ名 |
|---------|-------------|
| arg0 | Type（1=Action Label） |
| arg2 | Label |

### Return（code 126）

```
A50: Return [
      Value: %result
      Stop: On ]
```

| XML arg | パラメータ名 |
|---------|-------------|
| arg0 | Value |
| arg1 | Stop（1=On） |

### HTTP Request（code 339）

```
A15: HTTP Request [
      Method: GET
      URL: https://example.com/api
      Headers: %header
      Body: %body
      Timeout (Seconds): 30
      Structure Output (JSON, etc): On
      Continue Task After Error:On ]
```

| XML arg | パラメータ名 | 備考 |
|---------|-------------|------|
| arg1 | Method | 0=GET, 1=POST, 4=PATCH |
| arg2 | URL | |
| arg3 | Headers | |
| arg5 | Body | |
| arg8 | Timeout (Seconds) | |
| arg12 | Structure Output | 1=On |
| `<se>` | Continue Task After Error | false=On |

### Perform Task（code 130）

```
A33: Perform Task [
      Name: Sub_TaskName
      Priority: %priority+1
      Parameter 1 (%par1): value1
      Parameter 2 (%par2): %body
      Return Value Variable: %result
      Structure Output (JSON, etc): On ]
```

| XML arg | パラメータ名 |
|---------|-------------|
| arg0 | Name |
| arg1 | Priority |
| arg2 | Parameter 1 (%par1) |
| arg3 | Parameter 2 (%par2) |
| arg4 | Return Value Variable |
| arg10 | Structure Output（1=On） |

### JavaScriptlet（code 129）

```
A47: JavaScriptlet [
      Code: var data = JSON.parse(http_data);
     var count = data.results.length;
      Auto Exit: On
      Timeout (Seconds): 45 ]
```

| XML arg | パラメータ名 |
|---------|-------------|
| arg0 | Code |
| arg2 | Auto Exit（1=On） |
| arg3 | Timeout (Seconds) |

### Flash（code 548）

```
A21: Flash [
      Text: メッセージ (%count/3)
      Continue Task Immediately: On
      Dismiss On Click: On ]
```

| XML arg | パラメータ名 |
|---------|-------------|
| arg0 | Text |
| arg9 | Continue Task Immediately（1=On） |
| arg11 | Dismiss On Click（1=On） |

### Wait（code 30）

```
A14: Wait [
      MS: 0
      Seconds: 3
      Minutes: 0
      Hours: 0
      Days: 0 ]
```

| XML arg | パラメータ名 |
|---------|-------------|
| arg0 | MS |
| arg1 | Seconds |
| arg2 | Minutes |
| arg3 | Hours |
| arg4 | Days |

### For（code 39）

```
A45: For [
      Variable: %count
      Items: 1:%list(#) ]
```

| XML arg | パラメータ名 |
|---------|-------------|
| arg0 | Variable |
| arg1 | Items |

### Profile Status（code 159）

```
A47: Profile Status [
      Name: %profile_name
      Set: On ]
```

| XML arg | パラメータ名 |
|---------|-------------|
| arg0 | Name |
| arg1 | Set（1=On, 0=Off） |

---

## プラグインアクション

AutoTools、AutoLocation 等のプラグインは長いコード番号を持つ。

Description では `Configuration:` にプラグインの設定概要が表示される。
XML では `com.twofortyfouram.locale.intent.extra.BLURB` フィールドに同じ内容が格納されている。

```
A24: AutoTools Dialog [
      Configuration: Dialog Type: 2 Choices
     Title: ネットワーク未接続
     Choice One: OK
     Choice Two: Cancel
      Timeout (Seconds): 60
      Structure Output (JSON, etc): On ]
```

### 主なプラグインコード例

| コード | プラグイン / アクション |
|--------|----------------------|
| 1304982781 | AutoTools Dialog |
| 725543659 | AutoLocation Location |
| 1879487834 | AutoLocation Geofences |
| 1469986059 | AutoLocation Activities |
| 1520257414 | AutoNotification Event |
| 1447159672 | AutoTools Text |
