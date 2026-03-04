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
- `columns`（固定列数）または `minItemWidth`（最小幅自動計算）で指定

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
