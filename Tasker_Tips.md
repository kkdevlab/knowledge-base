# Tasker Tips

Tasker の仕様・ノウハウをまとめたリファレンス。

---

## 変数の型と比較

### 変数に型はない

Tasker の変数は厳密な型（Integer や String など）を持たない。
値が `10` の場合、使用する演算子によって「数値の10」にも「文字列の"10"」にもなる。

- **数値として比較**: `%count` `Maths: Greater Than` `5` → 数値比較
- **文字列として比較**: `%count` `Matches` `10` → パターンマッチ（文字列比較）

### 演算子による挙動の違い

| 種類 | 演算子例 | 解説 |
|------|---------|------|
| **数値比較** | Maths: Equals (=), Maths: Greater Than (>) | 両方の値を数値として評価。`01` と `1` は等しい |
| **パターンマッチ** | Matches (~), Doesn't Match (!~) | 文字列として比較。ワイルドカード `*` が使える。`01` と `1` は一致しない |
| **正規表現** | Regex Matches (~R) | 高度な文字列検索 |

### 条件演算子一覧

| 演算子 | 略称 |
|--------|------|
| Equals | EQ |
| Doesn't Equal | NEQ |
| Matches | ~ |
| Doesn't Match | !~ |
| Matches Regex | ~R |
| Doesn't Match Regex | !~R |
| Maths: Less Than | < |
| Maths: Greater Than | > |
| Maths: Equals | = |
| Maths: Isn't Equal To | != |
| Maths: Is Even | EVEN |
| Maths: Is Odd | ODD |
| Is Set | SET |
| Isn't Set | !SET |

### 注意事項

- **If 内で直接計算はできない**: `If [ %value + 1 > 10 ]` は不可
- **解決策**: 先に Variable Set（Do Maths チェック）で計算し、結果を比較に使う
- **未定義変数の数値比較**: 変数が空の状態で数値比較するとエラーになる場合がある。事前に初期値を設定すること

### ベストプラクティス

- 数値比較（「〜より大きい」「〜以下」など）→ **Maths: ...** 系を使う
- 特定の文字と一致するか → **Matches** を使う

---

## JavaScriptlet の制限事項

### Java クラスにアクセス不可

Tasker の JavaScriptlet（JavaScript/JavaScriptlet アクション）は完全にサンドボックス化されており、
Java・Android クラスへのアクセスは **一切できない**。

| 試みた記述 | エラー |
| --- | --- |
| `java.util.UUID.randomUUID()` | `ReferenceError: java is not defined` |
| `Packages.javax.crypto.Mac` | `ReferenceError: Packages is not defined` |
| `Java.type("javax.crypto.Mac")` | `ReferenceError: Java is not defined` |
| `android.util.Base64.encodeToString(...)` | `ReferenceError: android is not defined` |

### Java Code アクション（BeanShell）を使う

Tasker v6.6 以降の **Java Code** アクション（`+` → Code → Java Code）は BeanShell で動作し、
本物の Java コードが書ける。HMAC-SHA256 などの Crypto API も直接使用可能。

#### 変数 API

| メソッド | 用途 |
| --- | --- |
| `tasker.getVariable("VAR_NAME")` | 変数取得（大文字名 = グローバル、小文字名 = ローカル） |
| `tasker.setVariable("var_name", value)` | 変数セット（大文字名 = グローバル、小文字名 = ローカル） |

#### Java Code アクションの「Return」フィールドに注意

Java Code アクション UI には「Return」フィールドがある。
**このフィールドにはスクリプトの最終評価値（BeanShell の戻り値）が格納される。**

- `tasker.setVariable("auth_json", ...)` は `void` を返す
- Return フィールドに `%auth_json` を設定すると、`void`（= 空）で上書きされてしまう

**正しい構成：**

1. Java Code アクション → Return フィールドは **空欄**
2. その後に別の **Return アクション**（Task カテゴリ → Return）を追加して値を返す

```text
// アクション 1: Java Code（Return フィールド = 空欄）
tasker.setVariable("auth_json", ...);

// アクション 2: 別の Return アクション（Task → Return）
Return: %auth_json
```

#### 最小構成例

```java
import java.util.UUID;
String nonce = UUID.randomUUID().toString();
tasker.setVariable("nonce", nonce);
```

---

## Variable Split + JavaScriptlet の連携

Variable Split で分割した変数を JavaScriptlet で使う場合の注意点。

### NG パターン

Variable Split で `%result` を分割後、JavaScriptlet 内で：

- `result2` → 存在しない変数（エラー: `result2 is not defined`）
- `result[1]` → 文字列の2文字目を返す（配列要素ではない）

### OK パターン

Split 後に明示的に別変数へコピーしてから JavaScriptlet で参照する：

1. `Variable Set %http_data = %result(2)`
2. JavaScriptlet 内で `JSON.parse(http_data)` を使用

### なぜこうなるか

Variable Split 後の `%result` は Tasker 配列 `%result()` になる。
JavaScriptlet への注入時、配列はカンマ区切り文字列として渡されるため、
JavaScript の `result` は文字列であり、`result[N]` は N+1 番目の**文字**を返す。

---

## Tasker 配列変数を JavaScriptlet で参照する方法

### 症状

For ループなどで構築した Tasker 配列変数（`%var(n)`）を JavaScriptlet の `local()` で取得しようとしても空文字列が返る。

```javascript
// NG: local('status_entries') → 空文字列
var entries = local('status_entries').split(',');
```

### なぜ起きるか

`local('varname')` は Tasker 配列変数（`%var(n)`）を返せない。

### 正しいアクセス方法

インデックス付きの名前で個別にアクセスする。

```javascript
// OK: local('status_entries1') → %status_entries(1) の値
var count = parseInt(local('idx')) || 0;
var statusMap = {};
for (var i = 1; i <= count; i++) {
    var entry = local('status_entries' + i);
    if (entry) {
        var kv = entry.split('|');
        if (kv.length === 2) statusMap[kv[0]] = kv[1];
    }
}
```

`local('varname' + i)` が Tasker の `%varname(i)` に対応する。

### 注意

配列の要素数をループカウンター変数（例: `%idx`）として別途保持しておくこと。

---

## Tasker ドット記法による JSON オブジェクトアクセス

Tasker の変数に JSON オブジェクトが入っている場合、ドット記法でフィールドに直接アクセスできる。

### 使用例

```json
%result = {"success":true,"status":200,"count":3,"data":{...},"error_msg":""}

%result.success   → true
%result.data      → {...}  （Notion レスポンス本体など）
%result.error_msg → ""
```

### Condition での使用

```text
If %result.success ~ false  → Goto ERRCODE
```

### Variable Set での使用

```text
Variable Set %json_data = %result.data
```

### メリット

- Variable Split → 個別変数コピー の手順が不要
- JavaScriptlet に渡す前に必要なフィールドだけ絞り込める
- エラーチェックが Condition 1行で書ける

### JavaScriptlet との組み合わせパターン（推奨）

```text
1. Perform Task: Sub_NotionQuery → Return Value Variable: %result
2. Goto ERRCODE  If: %result.success ~ false
3. Variable Set: %json_data = %result.data
4. JavaScriptlet: var data = JSON.parse(local('json_data')); ...
```

---

## Widget V2 レイアウト構築

Tasker 6.4以降の標準ホーム画面ウィジェット機能。

### ホーム画面への配置手順

1. ホーム画面を長押し → 「ウィジェット」
2. 「Tasker」→「Widget V2」を選択して配置
3. **ウィジェット名を入力**（例: `DateWidget`）← タスクから参照するキーになる

### 利用可能な要素（+ ボタンで追加）

| 要素 | 用途 |
|------|------|
| Text | テキスト表示 |
| Image | 画像 |
| Button / Icon Button | タップ可能なボタン |
| Column | 縦並びコンテナ |
| Row | 横並びコンテナ |
| Grid | グリッドレイアウト |
| Box | 汎用コンテナ |
| Spacer | 余白 |
| Scaffold / Title Bar | 画面フレーム |

### 2×2 グリッドの構造

```
Column（Size: Fill）
├── Row（1行目）
│   ├── Text（Is Weighted: True）
│   └── Text（Is Weighted: True）
└── Row（2行目）
    ├── Text（Is Weighted: True）
    └── Text（Is Weighted: True）
```

### 重要なプロパティ

| プロパティ | 推奨設定 | 効果 |
|-----------|---------|------|
| Column > Size | `Fill` | ウィジェット全体を埋める |
| Row > Size | 数値入力（Manual）| Columnと異なりドロップダウンなし |
| Text > Is Weighted | `True` | Row内で均等幅に配分 |
| Text > Max Lines | `1` | 折り返し防止 |
| Text > Align | `Center` | テキスト中央寄せ |
| Text > Text Size | 数値（sp） | フォントサイズ指定 |

### Size ドロップダウンの選択肢（Column等のコンテナ）

- `Fill` : 幅・高さともに親要素いっぱいに広げる
- `FillWidth` : 幅だけ広げる
- `FillHeight` : 高さだけ広げる
- `Manual` : Width / Height を数値で個別指定

### レイアウトエディタのパンくず（ナビゲーション）

```
≡ » ≡ » ||| » Tt
↑    ↑    ↑    ↑
Root Column Row  Text（現在地）
```

タップすることで上位の要素に移動できる。

### Perform Task で戻り値を受け取る

サブタスクが `Return` アクションで返す値は、呼び出し元の **Return Variable** フィールドに指定した変数に入る。省略すると失われる。

```
Perform Task: Sub_Date
  Par1: 0
  Return Variable: %result   ← 必須
```

---

## Widget V2 の変数置換における日本語文字の注意点

### 問題

Tasker は日本語（CJK文字）を変数名の一部として認識するため、変数直後に日本語を書くと置換が失敗する。

```
%month月%day日  → そのまま表示される（置換されない）
```

### 原因

Javaの `Character.isLetter('月')` は `true` を返すため、Taskerのパーサーが `月` を変数名の続きと解釈する。

### 解決策

**① 配列変数の `(n)` 記法を使う（推奨）**

```
%result(2)月%result(3)日  → 2月26日（正常置換）
```

閉じ括弧 `)` が変数名の終端として明確に機能する。

**② スペースを入れる（次善策）**

```
%month 月%day 日  → 2 月26 日（スペースが入る）
```

**③ `%(varname)` 記法は Tasker では使えない**

```
%(month)月  → そのまま表示される（非対応）
```

### 推奨パターン

ゼロ除去しつつ配列記法を維持する：

```
// Variable Split で %result(2)="02", %result(3)="25" を取得後
Variable Set: %result(2) = %result(2)  Do Maths: ON  → "2"
Variable Set: %result(3) = %result(3)  Do Maths: ON  → "25"

// Widget V2 の Text に
%result(2)月%result(3)日  → 2月26日 ✓
```

---

## Widget V2 カスタムレイアウト（JSON形式）

公式リファレンス: [Widget v2 Custom Layout](https://tasker.joaoapps.com/userguide/en/widgetv2_custom.html)

ビジュアルエディタの代わりに JSON で直接レイアウトを記述できるモード。

### 変数のJSON埋め込みルール

- **変数を文字列値として埋め込む場合**: Variable Convert → JSON Encode で特殊文字をエスケープしてから埋め込む
- **変数がJSON構造要素（数値・bool・キー・オブジェクト）の場合**: JSON Encode 不要（そのまま埋め込む）

```
Variable Convert: %text  Function: JSON Encode
→ %text の引用符や改行がエスケープされた安全な文字列になる
```

### 共通プロパティ（全要素）

#### サイズ

- `width`, `height`, `size` — 数値（dp）
- `fillMaxWidth`, `fillMaxHeight`, `fillMaxSize` — true/false

#### 余白

- `padding`, `paddingTop`, `paddingBottom`, `paddingStart`, `paddingEnd`

#### スタイル

- `backgroundColor` — カラー文字列 or Material You カラー名
- `cornerRadius` — 角丸（dp）
- `visibility` — `Visible` / `Invisible` / `Gone`
- `useMaterialYouColors` — true で Material You 配色を有効化

#### インタラクション

- `task` — タップ時に実行するタスク名
- `taskVariables` — タスクに渡す変数（例: `%par1=hello`）
- `command` — タップ時に送るコマンド文字列
- `commandPrefix` — CheckBox/Switch の状態をコマンドの先頭に付加

### コンテナ要素

- **Box** — 汎用コンテナ（重ね合わせ可）
- **Row** — 横並びコンテナ
- **Column** — 縦並びコンテナ
- **Grid** — グリッドレイアウト（固定列数 or 最小アイテムサイズで指定）

#### 整列プロパティ（Row/Column/Box）

- `horizontalAlignment`: `Start` / `Center` / `End`
- `verticalAlignment`: `Top` / `Center` / `Bottom`

#### Grid の注意点

- 子要素が10個を超えるとスクロール可能になる
- **`fixed`**（固定列数）または **`minSize`**（最小幅自動計算）で指定
  - ❌ `"columns": 2` → 無効（全アイテムが横1行に並ぶ）
  - ✅ `"fixed": 2` → 2列の縦グリッドになる

### コンテンツ要素

#### Image

- `contentScale`: `Crop` / `Fit` / `FillBounds`
- `effect`: `circle`（円形切り抜き）/ `sepia` / `grayscale` / `tint`

#### Progress

- `progress`（0〜100）を指定 → 横バー表示
- `progress` 未指定 → 円形インジケーター（無限ループ）

#### CheckBox / Switch

- タップ時の状態（true/false）が `%par1` としてタスクに渡される
- `commandPrefix: true` にするとコマンド文字列の先頭に状態が付加される
- `taskVariables` で `par1` を明示指定すると、スイッチ状態の自動渡しが上書きされるため注意

#### Image 要素

- 画像ソースのプロパティ名は **`url`**（`image` や `src` は不正解）
- Tasker 内蔵アイコンの参照形式: `"android.resource://net.dinglisch.android.taskerm/drawable/アイコン名"`
- `tint` プロパティで色を指定可能（例: `"tint": "#ffffff"` で白）

```json
{
  "type": "Image",
  "url": "android.resource://net.dinglisch.android.taskerm/drawable/mw_action_bug_report",
  "size": { "width": 20, "height": 20 },
  "tint": "#ffffff"
}
```

#### Scaffold の背景色

- Scaffold は親要素の `backgroundColor` を**引き継がない**（Jetpack Compose の仕様）
- 黒背景にしたい場合は**ネストされた全 Scaffold に個別指定が必要**
- 確実な方法: 全 Scaffold に `"backgroundColor": "#000000"` を追加
- 透明化: `"backgroundColor": "#00000000"`（ARGB 先頭2桁=アルファ）で可能な可能性あり（未検証）

### Material You カラーパレット

`backgroundColor` などに以下のカラー名を直接指定可能:

```
primary / onPrimary / primaryContainer / onPrimaryContainer
secondary / onSecondary / secondaryContainer / onSecondaryContainer
tertiary / onTertiary / tertiaryContainer / onTertiaryContainer
error / onError / errorContainer / onErrorContainer
surface / onSurface / surfaceVariant / onSurfaceVariant
inverseSurface / inverseOnSurface / inversePrimary
```

### 最小構成JSON例

```json
{
  "type": "Box",
  "fillMaxSize": true,
  "backgroundColor": "surface",
  "horizontalAlignment": "Center",
  "verticalAlignment": "Center",
  "children": [
    {
      "type": "Text",
      "text": "%var_text",
      "textColor": "onSurface"
    }
  ]
}
```

---

## Wait Until の仕様と正しい使い方

### 仕様

- `Wait Until` は**条件が成立するまで無期限に待機**する
- `Seconds` パラメータは「タイムアウト」ではなく**条件再チェックの最大間隔**（インターバル）
- 条件が永遠に成立しない場合、タスクはハングする
- タイムアウト機能は存在しない（コミュニティで Feature Request 段階）

参考: [Tasker Wait Until 公式ドキュメント](https://tasker.joaoapps.com/userguide/en/help/ah_wait_until.html)

### 適切な使用例（条件が必ず成立する場合）

```text
Wait Until: %http_done = 1    ← 非同期タスクが必ず完了する
Wait Until: %WIFI ~ on        ← 接続試行中なので必ず成立する
```

### タイムアウト付き待機が必要な場合

**ポーリングループ**を使う（For × 小さいWait + Break）：

```text
Variable Set: %wk_flag = 0
For: %wk_i, 1 to 10        ← 10回 × 0.5秒 = 最大5秒
  Wait: 0.5秒
  If: %wk_flag = 1
    Break                   ← 条件成立で即脱出
  End If
End For
```

または複合条件（タイムスタンプ比較）：

```text
Variable Set: %expire = %TIMES + 5   ← 5秒後のタイムスタンプ
Wait Until: %wk_flag = 1 OR %TIMES > %expire
```

---

## Widget V2 Row内の均等幅分割（isWeighted）

### 問題

Row内の子要素に `fillMaxWidth: true` を設定すると、1つ目の要素が全幅を占有し2つ目以降が消える。
`weight: 1` も効果なし。

### 正しい方法

```json
{
  "type": "Row",
  "size": {"fillMaxWidth": true},
  "children": [
    {"type": "Button", "text": "左", "isWeighted": true},
    {"type": "Button", "text": "右", "isWeighted": true}
  ]
}
```

- Row 自体に `"size": {"fillMaxWidth": true}` を設定
- 各子要素に `"isWeighted": true` を設定（`fillMaxWidth` / `weight` は効かない）

### 実装例

`UpdateDateWidget.tsk.xml` の2列日付表示（西暦列 + 和暦列）が参考実装。
Column 要素で `"isWeighted": true` を使い均等分割している。

---

## Widget V2 taskVariables でTasker変数（%var）は展開されない

### 問題

```json
{"task": "UpdateDateWidget", "taskVariables": {"par1": "%WK_Today"}}
```

`%WK_Today` がリテラル文字列のままタスクに渡される（変数展開されない）。

### 原因

Widget V2 は taskVariables の値に含まれる Tasker 変数参照（`%var`）を展開しない。

### 解決策：ラッパータスクパターン

```
button_config.json
  → "task": "V2_UpdateDateWidget"（ラッパー）

V2_UpdateDateWidget タスク:
  A1: Perform Task [
       Name: UpdateDateWidget
       Par1: %WK_Today      ← グローバル変数を直接参照
       Par2: %WK_Holiday
  ]
```

ラッパータスク内では通常の Tasker 変数参照が機能するため、グローバル変数を正しく渡せる。

### ルール

- Widget V2 の taskVariables → 静的な値（デバイスID等）のみ使う
- Tasker グローバル変数を渡す必要がある場合 → ラッパータスクを作成する

---

## 未設定変数の検出イディオム（~R \%）

### 問題

Tasker では、変数が未設定の場合、その変数名がそのまま展開される。

```text
%txt_time が未設定 → 値は文字列 "%txt_time"（%記号を含む）
%txt_time が設定済 → 値は "09:00" など（%記号を含まない）
```

### 検出方法

```text
If [ %txt_time ~R \% ]
```

- 演算子: **Matches Regex** (`~R`)
- パターン: `\%`（Java正規表現でリテラルの `%` にマッチ）
- 「値に `%` が含まれるか」= 「変数が未設定か」を検出する

### なぜ "Is/Isn't Set" を使わないか

`Is Set` / `Isn't Set` 演算子は変数が一度でも代入されたかを見る。
Scene V2 の `sv2_element_value()` は、対応要素が空欄の場合に展開されない変数名を返す（"Is Set" 状態でも値が `%...`）ため、 `~R \%` の方が確実。

### 空欄のコメントを許容する場合のパターン

```text
If [ %comment ~R \% ]
    Variable Set %comment = " "   ← 半角スペースで上書きして"未設定扱い"を解除
End If
```

エラーにせずデフォルト値で続行する場合はこのパターンを使う。

---

## For ループのコロン範囲記法

Tasker の For アクションの `Items` フィールドは、コンマ区切りリストのほかに **コロン範囲記法** を使える。

### 記法

| 形式 | 意味 |
| ------ | ------ |
| `1:10` | 1 から 10 まで（ステップ=1） |
| `1:%count` | 1 から変数の値まで |
| `1:%sv2_element_id(#)` | 1 から配列要素数まで |
| `1:2:10` | 1 から 10 まで、ステップ=2 |

### XML の実構造

```text
Variable: %idx
Items: 1:%sv2_element_id(#)   ← Items 1フィールドに start:end
Step: 1                        ← arg2 がステップ
```

疑似コードで「`For %idx, 1, N`（カンマ3引数）」と書きがちだが、Tasker UI では **Items 欄に `start:end` を一括指定**する形式が正しい。

---

## Scene V2 全要素スキャンパターン（sv2_element_id/value ループ）

Scene V2 のテキスト入力・選択値などを一括取得する際のパターン。
GetElement(code 483) を要素ごとに呼ぶ代わりに、For ループで全要素を走査してID比較で振り分ける。

### パターン

```text
For %idx, Items: 1:%sv2_element_id(#)

    If [ %sv2_element_id(%idx) ~ txt_time ]
        Variable Set %txt_time = %sv2_element_value(%idx)
    Else If [ %sv2_element_id(%idx) ~ comment ]
        Variable Set %comment = %sv2_element_value(%idx)
    Else If [ %sv2_element_id(%idx) ~ val_busy ]
        Variable Set %val_busy = %sv2_element_value(%idx)
    Else If [ %sv2_element_id(%idx) ~ val_temp ]
        Variable Set %val_temp = %sv2_element_value(%idx)
    End If

End For
```

### スキャンパターンのメリット

- GetElement を要素数分呼ぶ必要がない（アクション数削減）
- ID比較なので要素の追加・順序変更に強い
- `%sv2_element_id(#)` で要素数を動的に取得するため要素数変更も自動対応

### スキャンパターンの注意事項

- `%sv2_element_id(%idx) ~ "txt_time"` の `~` は Matches（シンプルパターン）
  - ワイルドカードなし = 大文字小文字を区別しない完全一致として動作
- `sv2_element_value()` が返す値は、空欄の input area は `%sv2_element_valueN` という未展開変数名になる
  → 取得後に `~R \%` でバリデーションすること

---

## 正規表現エスケープ文字の日本語環境差異（`\.` vs `¥.`）

Variable Search Replace の Search フィールドに正規表現を使う場合：

| 入力環境 | 表記 | 意味 |
| --------- | ------ | ------ |
| ASCII 環境（PC 直接編集など） | `\.` | バックスラッシュ + ドット = リテラルのドット |
| 日本語 Android キーボード | `¥.` | 円マーク + ドット = 同じくリテラルのドット |

**Taskerの正規表現エンジン（Java）は `¥`（円マーク）を `\`（バックスラッシュ）と同等に扱う。**
どちらで入力しても動作は同一。日本語環境では `¥.` と表示されることが多い。

---

## Variable Search Replace の Replace With フィールドのエスケープ解釈

### Replace With フィールドの `\n` / `\\n` の違い

| Replace With に入力 | Tasker が出力する文字列 | 用途 |
| --- | --- | --- |
| `\n` | 実際の改行文字（LF） | 改行を挿入したいとき |
| `\\n` | `\` + `n` の2文字リテラル | JSON の改行エスケープを埋め込みたいとき |

### JSON に改行付き変数を埋め込むパターン（%wk_lf パターン）

Scene V2 の `multiLine: true` TextInput などで取得した変数には実際の改行文字が含まれる。
JSON 文字列に直接埋め込むと無効な JSON になるため、事前にエスケープが必要。

```text
Variable Set  %wk_lf = [To フィールドに実際の改行を入力]
Variable Search Replace
  Variable: %comment
  Search:   %wk_lf       ← 実際の改行文字（Regex OFF でも動く）
  Replace:  \\n          ← JSON の \n エスケープ（2文字リテラル）として出力
```

### `%wk_lf` vs `¥n` の使い分け

- `%wk_lf`（変数に実際の改行を格納）: フィールドを問わず常に改行文字として振る舞う
- `¥n` はフィールドによって解釈が異なる:
  - Search フィールド（Regex ON）→ 改行文字にマッチ
  - Replace With フィールド → 実際の改行文字を出力（= `%wk_lf` と同じ結果）
  - Variable Set To フィールド → `¥n` の2文字テキストのまま（改行にならない）

→ Search に `%wk_lf` を使う方が Regex ON/OFF に依存せず安全。

確認バージョン: Tasker 6.7.3-beta

---

## Custom Setting でシステム設定を読み取る

Custom Setting アクション（code 235）は書き込みだけでなく読み取りにも使える。

### 読み取り方法

- **Value**: 空のまま
- **Read Setting To**: 読み取り先の変数名（例: `%usb_adb`）

```text
Custom Setting: Type=Global, Name="adb_enabled", Value=(空), Read Setting To=%usb_adb
```

→ `%usb_adb` に現在の値（`"0"` または `"1"`）が格納される。

### 書き込みと同時に読み取る場合

Value と Read Setting To の両方を指定すると、書き込んだ後の値が変数に入る。

---

## プロファイル/設定変更時の Monitor 再起動と State プロファイルの再発火（仕様）

### 挙動

Tasker の設定(Preferences)を抜ける、またはプロファイルを**オン/オフ・編集**すると、設定反映のため **Monitor（監視サービス）が再起動**する。このとき**その時点で「有効」だった State プロファイルのコンテキストが作り直され、「有効→無効→有効」の遷移が発生して Exit タスクと Enter タスクが両方再発火する**。実際の条件（WiFi 接続など）が変わっていなくても遷移イベントが発生する点が要注意。

- **これはバグではなく仕様**。決定論的に再現する。
- 常時有効な State プロファイル（例: 自宅 WiFi 接続）は、設定を抜けるたびに Exit→Enter を撃ち直す。
- 有効プロファイルが**無効化された瞬間も Exit が走る**（「有効だった状態から抜けた」とみなされる）。無効化は次の Enter（再有効化）を止めるだけで、Exit は止めない。

### タスク内容の編集では起きない

タスクの中身を編集した場合も Monitor は再起動するが、**プロファイルのコンテキストは保持され再サイクルされない**（Run Log に `P Inactive` が出ない）。

→ 引き金は「Monitor 再起動」そのものではなく「**プロファイル・コンテキストの再評価を伴う操作**」。Run Log に `P Inactive <ProfileID>` が出るかどうかが分かれ目。

| 操作 | Monitor 再起動 | プロファイル再サイクル(Exit→Enter) |
|---|---|---|
| タスクの中身を編集 | する | **しない**（context 保持・`P Inactive` 無し） |
| プロファイルのオン/オフ・編集、設定を抜ける | する | **する**（`P Inactive`→`P Active`） |

### なぜ普段は気づかれないか

再サイクルは起きていても、Enter/Exit タスクが**地味な処理（変数セット等）なら無害で無音**。シーン表示・着信音・通知などの**目立つ副作用**を Enter/Exit に持つ場合だけ「設定を抜けると毎回これが動く」と気づく。

### 対策の考え方（防御的設計）

仕様なので修正待ちではなく、**偽の再サイクルに強い設計**にする：

- **A. Exit タスク冒頭でディベウンス＋状態再確認**: 数秒待って実状態（WiFi 等）がまだ維持されているか再確認し、維持されていれば `Stop`。偽の再サイクルでは実状態が変わっていないので、副作用の起点（フラグセット等）を未然に止められる。Enter 側の安定確認と対称。
- **B. Monitor 再起動検知フラグ**: 「Monitor Start」を拾うプロファイルでフラグを数秒立て、副作用タスクはフラグ中スキップ。全プロファイル横断で守れる。

---

## %WIFII（WiFi 接続情報）の中身

State「WiFi Connected」等で使う組み込み変数 `%WIFII` には、接続中のとき以下のようなブロックが入る。**接続中 SSID はダブルクォートで囲まれる**。

```text
>>> CONNECTION <<<

"YourSSID-5G"

Mac: xx:xx:xx:xx:xx:xx
IP : 192.168.x.x
Sig: 9
Spd: 864Mbps
Chl: 48
```

- 切断時はこの `CONNECTION` ブロックが無い。
- SSID だけ欲しいときは `"`（ダブルクォート）で Variable Split して中身を取り出す（クォートが区切りになる）。
- **⚠️ `%WIFII` は非リアルタイム（monitored 変数）**: Prefs→Monitor の WiFi スキャン間隔（既定で分単位）でしか更新されない。**数秒スケールの「今つながっているか」判定には使えない**（古い値が返る）。
- **現在の接続中 SSID をリアルタイムに取得するには `Test Net`（code 341, Type: Wifi SSID）を使う**。結果変数（例 `%result`）に現在 SSID が入るので `%result ~R %target_ssid` で比較する（`~R` は部分一致＝SSID 変数をそのまま置ける。特殊文字を含むなら glob `~ *%target_ssid*`）。
- この「monitored 変数は非リアルタイム」は %WIFII に限らず位置・セル・センサー系の組み込み変数すべてに該当する（→ 次節）。

---

## 組み込み変数の更新間隔（monitored 変数は非リアルタイム）

WiFi・位置・セル・センサー系の組み込み（グローバル）変数は「**monitored**」型で、**リアルタイムではなく Prefs→Monitor の間隔でしか更新されない**。バックグラウンドのスキャン頻度に依存するため、タスク内で数秒スケールの「今の状態」を判定する用途には使えない（古い値が返る）。

- **該当する主な変数**: `%WIFII`（WiFi情報）／ `%LOC`・`%LOCN`（GPS・ネットワーク位置）／ `%CELLID`・`%CELLSIG`・`%CELLSRV`（セル）／ `%TETHER` ／ センサー系（`%LIGHT`/`%PRESSURE`/`%TEMP`/`%HUMIDITY`/`%HEART`/`%MFIELD`）等
- **更新間隔を決める設定**: Prefs → Monitor（例: `Wifi Scan Seconds`＝画面ON時のWiFiスキャン、`Display Off All Checks`＝画面OFF時の全チェック、各 Cell/GPS/Net の間隔）。具体値はバージョン・端末で異なる（数値より「**非リアルタイム**」という性質が重要）。
- **原則**: リアルタイムの値が必要なら、変数を読まず**能動プローブ・アクション**で都度取得する。
  - WiFi接続/SSID → `Test Net`（code 341, Type: Wifi SSID）→ 結果変数に現在SSID
  - 位置・セルも同様に対応する取得/テスト系アクションを使う
- ロジック分析・新規タスク設計でも「これらの変数は非リアルタイム」を**常に前提**にする。
- 出典: Tasker 公式 Variables / Location Without Tears（monitored 変数とスキャン間隔）

---

## プロファイル XML（バックアップ）の読み方

`.prf.xml` / `backup.xml` を解析するときの要点。

| 要素 | 意味 |
|---|---|
| `<mid0>` | **Enter タスク**の Task ID |
| `<mid1>` | **Exit タスク**の Task ID |
| `<State><code>` | State コンテキストの種別コード（例: `160`=WiFi 接続(SSID 指定), `165`=変数値, `5`=カレンダー, BT/充電/電池 系 等） |
| `<limit>true</limit>` | **そのプロファイルは無効化**されている（監視対象外＝再評価・発火しない） |
| `<App>` `<Event>` `<Time>` `<Day>` `<Loc>` `<Cell>` | アプリ/イベント/時刻/曜日/位置/セル のコンテキスト種別タグ |

- **Enter/Exit の対応は `mid0`/`mid1`（XML）を正とする**。手書きの索引ドキュメントは Enter/Exit を取り違えていることがあるので必ず XML で確認する。
- 「ある状況で偽トグルしうる State プロファイル」を洗い出すには、`<limit>` が無い（=有効）かつ その状況で active になる State コンテキストだけが候補、という観点で絞り込める。

---

## タグの XML 構造（新UI / 6.7.5-beta で確認）

> **ベータ仕様**: 6.7.5-beta（実機 export で確認、2026-06-17）。正式版で要素名・構造が変わる可能性があるためバージョン付きで記録。

新 Main Screen UI のタグは、XML 上では **「定義」と「参照」が分離**している。

### 定義（ID ↔ 名前 ↔ 色）

フルバックアップ（`backup.xml`）の root 直下に、タグ1個＝`<TaskerTag>` 1要素で集約される。

```xml
<TaskerTag sr="tags0">
    <clr>4283072457</clr>   <!-- タグの色（符号なし32bit ARGB 整数） -->
    <id>357b270b-bc8a-4b7c-966d-81ccc6ba09ae</id>   <!-- タグID（UUID） -->
    <lbl>All Action</lbl>   <!-- タグ名（ラベル） -->
</TaskerTag>
```

### 参照（各アイテム側）

タグを付けたアイテム（`<Task>` 等）は、**タグID を配列で参照するだけ**。名前は持たない。

```xml
<Task sr="task174">
    <_arrlst_tagIds0>357b270b-bc8a-4b7c-966d-81ccc6ba09ae</_arrlst_tagIds0>
    ...
```

- `_arrlst_<field><index>` は Tasker の**配列リスト**シリアライズ形式。`tagIds` 配列の 0 番目。タグを複数付けると `_arrlst_tagIds0`, `_arrlst_tagIds1`, … と増える。
- 方向は従来のプロジェクトと**逆**：プロジェクトは `<Project>` 側が `<tids>` で所属タスクIDを列挙（＝容れ物）。タグは各アイテムが自分の所属タグIDを持つ（＝横断ラベル）。

### 含まれる範囲（重要）

| export 種別 | タグ情報 |
|---|---|
| 個別 export（`.tsk.xml` 等） | `_arrlst_tagIds{N}`＝**ID 参照のみ**（名前は入らない） |
| フルバックアップ（`backup.xml`） | `<TaskerTag>` で **ID↔名前↔色の定義**を持つ |

- バックアップ/エクスポートに**タグは含まれる**。
- 形式は**旧UI/新UIどちらから出しても同じ**（タグはデータモデル側に保存、UI 非依存。旧UIに戻して個別 export してもタグ参照は残る）。
- 個別 export を別端末へ取り込むと、相手に同 ID のタグが無ければ**名前不明のまま宙に浮く**。開発者が export 時に「タグを外す」ステップを用意したのはこの ID 参照上書き事故を防ぐため。

---

## AutoNotification のグループ化（GroupKey だけでは足りない・サマリー通知が必須）

AutoNotification で「呼び出し元/カテゴリごとに別グループの束」を作りたいとき、**GroupKey を付けるだけでは不十分**。実機（Android 実測）で確認した挙動：

### 通知の識別は3層（混同しない）

| 層 | フィールド | 役割 |
| --- | --- | --- |
| **通知ID** | `notificaitionid` | 1通知の識別子。**同じIDで出し直すと上書き（更新）**。同一IDは同時に1つだけ。キャンセルも1件単位 |
| **グループ** | `GroupKey` | 同じ GroupKey の通知を1束にまとめる。**複数の別ID通知**を束ねられる |
| **サマリー** | `IsGroupSummary=true` | 束の折りたたみ見出し。**これが無いと束が独立表示されない** |

### 症状と原因

- GroupKey を付けても、**同一アプリ（AutoNotification）の通知が全部「AutoNotification」1箱に自動集約**され、グループごとに分かれない。
- 通知ダンプに `%angroup: 0|com.joaomgcd.autonotification|g:Aggregate_S…` ＋ `%ansummary: 1` という **Android 自動生成のアグリゲート・サマリー**が現れる。これが全通知を巻き取っている。

### 正解：グループごとにサマリー通知を1つ出す

各グループにつき、**子通知とは別に**サマリー通知を発行する：

- `IsGroupSummary = true`
- `GroupKey = 子と同じ`
- `Id = 子と別`（例: グループ名 + `__grp`）
- 音・バイブ・ボタンは無し（見出し役なので静かに）

→ そのグループが**独立した束**として表示される。Tasker 本体の通知が独立表示できているのも、Tasker が自前のグループサマリーを出しているため（ダンプで `%ansummary:1` を確認できる）。

### 束ねる/上書きの使い分け（呼び出し設計）

| やりたいこと | 通知ID | GroupKey | サマリー |
| --- | --- | --- | --- |
| 最新1件だけ（上書き更新） | 固定（毎回同じ） | 任意 | 不要 |
| 複数を1グループに蓄積 | アイテムごとに別 | グループ名で固定 | **必要** |

### 制約・注意

- **トップ行が「AutoNotification」になるのは配信元アプリがそれ1つだから**（回避不可）。束の分離はサマリーで可能。
- サマリー無しの単体通知（GroupKey はあるがサマリー無し）は Android の自動集約に入る（＝複数あると1箱にまとまる）。
- サマリー表示・折りたたみの細部は機種／Android バージョン差あり。実機確認前提。
- 実装リファレンス: `Tasker/Doc/Sub_Notify_Ring_仕様.md`（共通サブでの自動サマリー実装）

---

## Scene V2 Raw JSON（`<lj>`タグ）をCLIでデコード/エンコードする方法

`.scn.xml` の `<lj>` タグはシーンのレイアウトJSONを **gzip圧縮→base64エンコード** した文字列。Taskerアプリを開かずにPC上でシーン構造を確認・編集したい場合、PowerShellで直接デコード/エンコードできる。

デコード:

```powershell
$b64 = "<lj>タグの中身>"
$bytes = [Convert]::FromBase64String($b64)
$ms = New-Object System.IO.MemoryStream(,$bytes)
$gzip = New-Object System.IO.Compression.GZipStream($ms, [System.IO.Compression.CompressionMode]::Decompress)
$reader = New-Object System.IO.StreamReader($gzip)
$reader.ReadToEnd()
```

エンコード（JSON文字列 → `<lj>`用base64）:

```powershell
$json = '{"root":{...}}'
$bytes = [System.Text.Encoding]::UTF8.GetBytes($json)
$ms = New-Object System.IO.MemoryStream
$gzip = New-Object System.IO.Compression.GZipStream($ms, [System.IO.Compression.CompressionMode]::Compress, $true)
$gzip.Write($bytes, 0, $bytes.Length)
$gzip.Close()
[Convert]::ToBase64String($ms.ToArray())
```

- 変更後は必ずデコードして元のJSONと意図した差分だけになっているか確認してからXMLに書き戻す
- `cdate`/`edate`等の他フィールドは通常のXMLテキストなので直接編集でよい
