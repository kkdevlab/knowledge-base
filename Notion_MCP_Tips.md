# Notion MCP ツール Tips & 注意点

最終更新: 2026-08-13

Claude.ai / Claude Code から Notion MCP ツールを使う際のナレッジ集。
実際の作業で発生したエラーと正しい対処法をまとめる。

---

## 1. multi_select プロパティの値の渡し方

### 正しい形式: JSON配列文字列

multi_select は SQLite 形式で **JSON配列文字列** として渡す必要がある。

```json
"カテゴリ": "[\"スーパー\",\"薬局\",\"ホームセンター\"]"
```

### 間違い1: カンマ+スペース区切り

```json
"カテゴリ": "スーパー, 薬局, ホームセンター"
```

エラー: `Invalid multi_select value for property "カテゴリ": "スーパー, 薬局, ホームセンター"`

### 間違い2: カンマ区切り（スペースなし）

```json
"カテゴリ": "スーパー,薬局,ホームセンター"
```

エラー: 同上

### 確認方法

データベースを `notion-fetch` すると、SQLiteテーブル定義にフォーマットが明記されている:

```sql
"カテゴリ" TEXT, -- JSON array with zero or more of ["値1", "値2", ...]
```

**教訓**: 迷ったら `notion-fetch` でスキーマを確認してからデータ操作する。

---

## 2. select / multi_select に未登録オプションを使うとエラー

### 症状

select や multi_select に、スキーマに存在しないオプション値を指定するとエラー:

```
Invalid select value for property "カテゴリ": "ソニー".
Value must be one of the following: "スーパー", "薬局", ...
```

### 正しい手順

1. **先に** `notion-update-data-source` でオプションを追加
2. **その後** ページを作成/更新

```json
// Step 1: オプション追加
{
  "data_source_id": "xxxxx",
  "properties": {
    "カテゴリ": {
      "type": "multi_select",
      "multi_select": {
        "options": [
          {"name": "既存オプション1"},
          {"name": "既存オプション2"},
          {"name": "新しいオプション", "color": "purple"}
        ]
      }
    }
  }
}

// Step 2: ページ作成
{
  "parent": {"data_source_id": "xxxxx"},
  "pages": [{"properties": {"カテゴリ": "[\"新しいオプション\"]"}}]
}
```

**注意**: `update-data-source` で options を指定する際は、**既存オプションも全て含める**こと。省略すると既存オプションが消える可能性がある。

---

## 3. プロパティ型の変更はデータ消失を伴う

### 症状

single select → multi_select に変更するため、プロパティを一度 `null` で削除して再作成すると、そのプロパティの既存データが **全件消える**。

### 対処法

- **データ移行前に** スキーマを正しく設計する
- やむを得ず変更する場合は、先にデータをバックアップし、型変更後に再挿入する
- 可能であれば Notion の Web UI からプロパティ型を変更する方が安全

### 設計時の判断基準

| 用途 | 型 |
|------|-----|
| 値が常に1つだけ | select |
| 値が複数になり得る | **multi_select** |
| 迷ったら | **multi_select**（後から select に戻すのは困難） |

---

## 4. create-database の properties パラメータ

### 正しい形式: オブジェクト

properties は **オブジェクト** として渡す:

```json
{
  "properties": {
    "商品名": {"type": "title", "title": {}},
    "ステータス": {
      "type": "select",
      "select": {
        "options": [{"name": "購入予定"}, {"name": "購入済み"}]
      }
    }
  }
}
```

### 間違い: 文字列として渡す

```json
{
  "properties": "{\"商品名\": ...}"
}
```

エラー: `Expected object, received string`

---

## 5. Date プロパティの拡張形式

Date プロパティは3つの展開キーで設定する:

```json
{
  "date:購入日:start": "2026-01-10",
  "date:購入日:end": null,
  "date:購入日:is_datetime": 0
}
```

| キー | 必須 | 値 |
|------|------|-----|
| `date:プロパティ名:start` | 必須 | ISO-8601 日付/日時文字列 |
| `date:プロパティ名:end` | 任意 | 範囲指定時の終了日（単一日付なら省略） |
| `date:プロパティ名:is_datetime` | 任意 | 0=日付のみ, 1=日時含む（デフォルト0） |

日付がないアイテムは、date 関連キーを **全て省略** すればよい（null 不要）。

---

## 6. 大量データの一括挿入

`notion-create-pages` は最大100件/回だが、安定性のため **20〜25件ずつ** のバッチが推奨。

### バッチ挿入の手順

1. データを20〜25件ずつに分割
2. 各バッチで `notion-create-pages` を呼び出し
3. 成功件数を確認してから次バッチへ
4. エラー発生時はそのバッチのみリトライ

### multi_select を含む大量データの注意

- 全てのオプション値が事前にスキーマに登録されていることを確認
- 未登録値が1件でも含まれるとバッチ全体が失敗する
- **挿入前に**: `notion-fetch` でスキーマを取得し、必要なオプションを `update-data-source` で追加

---

## 7. notion-fetch の使い方

| 目的 | 渡す値 |
|------|--------|
| データベースのスキーマ確認 | データベースURL（例: `https://www.notion.so/xxxxx`） |
| ページの内容確認 | ページURL またはページID |
| data_source_id の確認 | データベースURL → `<data-source url="collection://xxxxx">` から取得 |

**注意**: `data_source_id` を直接 `notion-fetch` に渡すと 404 エラーになる。データベース URL を使うこと。

---

## 8. data_source_id と database_id は別物

Notion MCP と Notion REST API では、データベースを指すIDが異なる。

- **`data_source_id`**: MCP ツール（create-pages, update-data-source 等）で使用。`notion-fetch` の `<data-source url="collection://xxxxx">` から取得
- **`database_id`**: Notion REST API（`/v1/databases/{id}/query` 等）で使用。ブラウザ URL の UUID 部分から取得

### 例

```
MCP data_source_id:  5faa9701-1ff6-487a-b8fb-8703728e188d
REST database_id:    d172ea5b-98bf-43b6-a85b-4245e50f1de9
↑ 同じデータベースでも ID が異なる
```

### database_id の取得方法

ブラウザで DB を開き、URL から取得:

```
https://www.notion.so/d172ea5b98bf43b6a85b4245e50f1de9?v=...
                       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                       これが database_id（ハイフン付きでもなしでも可）
```

### 安定性

- `database_id` はページの永続UUID。移動しても**変わらない**
- `data_source_id` は内部コレクションID。構造変更で変わる可能性あり

---

## 9. Notion Integration のアクセス範囲と DB 移動

Internal Integration のアクセス権は、設定場所によって DB 移動時の挙動が異なる。

- **DB そのものに設定** → アクセス権維持（安全）
- **親ページに設定** → 移動先が対象外なら**アクセス権が外れる**

### 推奨

REST API から DB にアクセスする場合は、Integration を**データベースそのもの**に直接設定する。
親ページ経由の設定だと、ページ整理で DB を移動した際に 404 エラーになるリスクがある。

---

## 10. query_data_sources（SQL）は Business プラン必須 → search で代替

### 症状

`notion-query-data-sources`（SQL モード）でDBを検索すると 400 エラー：

```
This tool requires a Business plan or higher with Notion AI.
Upgrade your plan to query data sources
```

- 単一データソースの SQL / 複数データソース横断 / view モードは **Business プラン以上（Notion AI 付き）** が必要。

### 正解：`notion-search` で代替する

ページIDや行を探すだけなら、`notion-search` に `data_source_url`（`collection://…`）を渡してDB内をセマンティック検索する。

```json
{
  "query": "Tasker",
  "query_type": "internal",
  "data_source_url": "collection://60bb948d-eecc-4661-bb23-037e14f43f31",
  "page_size": 5
}
```

→ 結果の `id` がページID。relation 設定などに使える。

### 使い分け

| 目的 | 使うツール |
|------|-----------|
| DB内のページをキーワードで探す | `notion-search`（`data_source_url` 指定） |
| DBのスキーマ/プロパティ確認 | `notion-fetch`（DB URL） |
| SQL 集計・条件抽出・横断 | `notion-query-data-sources`（**Business プラン必須**） |

---

## 11. update-page(update_content) で特定の漢字が文字化けすることがある

### 症状

`content_updates` の `old_str`/`new_str` に特定の漢字（例: 「常駐」）を含めて送信すると、ツール呼び出し自体はエラーを返さず成功するが、実際にページに反映された文字が別の字に化ける（例: 「常駐」→「常駓」、さらに再送すると「常駓（常に常驅）」のように悪化することもある）。

### 対処法

- 同じ文字を使い回して再送・再修正を試みても直らないことがある
- **その漢字を使わない言い回しに書き換えて回避する**のが確実（例:「常駐しない」→「バックグラウンドで動き続けない」）
- 修正後は必ず `notion-fetch` で再取得し、文字化けが直っているか目視確認する

### 注意

- 原因不明（特定の漢字コードポイントに対するエンコーディング/変換処理のバグと推測される）。再現する文字は事前に予測できないため、反映確認は都度行う

---

## 12. ALTER COLUMN SET SELECT で既存オプションの色を変えるとエラー

### 症状

`notion-update-data-source` の `ALTER COLUMN "プロパティ名" SET SELECT(...)` で新しいオプションを追加する際、既存オプション名を全て含めていても、現在の色と異なる色（例: 適当に`default`を指定）を書くとエラーになる:

```
Cannot update color of select with name: Tasker.
```

### 原因

`ALTER COLUMN SET SELECT(...)` は「オプション一覧の完全な置き換え」として扱われるため、既存オプションの色を1つでも現在の値と違えると「色の変更」とみなされ拒否される。

### 正しい手順

1. 先に `notion-fetch` で対象データソースのスキーマを取得し、既存オプションの `color` を確認する
2. `ALTER COLUMN SET SELECT(...)` には、既存オプションを**現在と同じ色**で全て含め、新規オプションのみ好きな色を指定する

```json
// 例: 既存 Tasker(pink)/Windows(blue)/... に Android(brown) を追加
{
  "data_source_id": "xxxxx",
  "statements": "ALTER COLUMN \"対象\" SET SELECT('Tasker':pink, 'Windows':blue, 'Android':brown)"
}
```

**教訓**: 未登録オプション追加時は「先にfetchで現在の色を確認」を徹底する（項目2の「既存オプションも全て含める」に加えて、色も完全一致させる必要がある）。

---

## 13. create-pages で parent を省略すると、DBプロパティが無言で無視される

### 症状

`notion-create-pages`でデータベース配下のページを作るつもりで、プロパティ（select/relation等）を指定して呼び出したが、`parent`（`data_source_id`）を渡し忘れた。エラーは一切出ず成功レスポンスが返るが、実際にはタイトルもプロパティも全て空のスタンドアロンページとして作成される。

### 原因

`parent`を省略すると、そのページは「データベース配下」ではなく「ワークスペース直下のスタンドアロンページ」として扱われる。スタンドアロンページが受け付けるプロパティは`title`のみのため、DB用のプロパティ（select/relation/date等）は黙って無視される。

作成後に同じページへ`notion-update-page`（`update_properties`）で同じプロパティを送っても、ページがスタンドアロンのままである限り同様に無視される（エラーにもならない）。

### 正しい手順

1. `notion-create-pages`実行前に、必ず`parent`に`data_source_id`を指定する
   ```json
   {"parent": {"type": "data_source_id", "data_source_id": "xxxxx"}, "pages": [...]}
   ```
2. うっかり`parent`を省略して作成してしまった場合は、`notion-move-pages`で対象データソース配下へ移動してから、`update_properties`でプロパティを再設定する
   ```json
   {"page_or_database_ids": ["<page_id>"], "new_parent": {"type": "data_source_id", "data_source_id": "xxxxx"}}
   ```
3. 作成・修正後は必ず`notion-fetch`でタイトル・主要プロパティが実際に反映されているか確認する（成功レスポンスだけでは判断しない）

**教訓**: `create-pages`の成功レスポンスは「ページが作られたこと」の保証であって、「意図した場所・プロパティで作られたこと」の保証ではない。DB配下作成のつもりなら`parent`指定の有無をセルフチェックし、作成後は`notion-fetch`で裏取りする。

---

## 14. create-pages で parent を pages 配列の各要素に入れるとエラー

### 症状

`notion-create-pages`で、`parent`を`pages`配列の各アイテム(properties/contentと同じ階層)に入れて呼び出すと、次のバリデーションエラーになる。

```
Invalid arguments for tool notion-create-pages: Unrecognized key: "parent"
```

### 原因

`parent`は`pages`配列の各要素のプロパティではなく、ツール呼び出し全体のトップレベルパラメータ。1回の呼び出しで作成する複数ページは同じ親を共有する設計のため、`parent`は`pages`と同階層に1つだけ渡す。

```json
{
  "pages": [{"properties": {...}, "content": "..."}],
  "parent": {"type": "data_source_id", "data_source_id": "xxxxx"}
}
```

### 関連

項目13(`parent`を省略した場合は無言でスタンドアロンページになる)とは異なり、こちらは即座にエラーで気づける。

---

## 15. update-page は必ず `command` パラメータが必要（省略するとバリデーションエラー）

### 症状

`notion-update-page`を`page_id`と`properties`だけのフラットな引数で呼び出すと、次のエラーになる。

```
Input validation error: Invalid arguments for tool notion-update-page: page_id: Invalid input: expected string, received undefined, command: Invalid option: expected one of "update_properties"|"update_content"|"replace_content"|"insert_content"|"apply_template"|"update_verification"
```

### 原因

`notion-update-page`は複数の更新モードを1つのツールに集約しており、`command`でどのモードか明示する設計。プロパティだけ更新するつもりでも`command: "update_properties"`は省略できない。

### 正しい形式

```json
{
  "page_id": "xxxxx",
  "command": "update_properties",
  "properties": {"ステータス": "進行中"}
}
```

---

## 16. update_content の old_str は「fetchで返ってくる表示用テキスト」と一致しないことがある

### 症状

`notion-fetch`で取得したページ本文（`text`フィールド）から`old_str`をコピーして`update_content`（部分置換）を呼ぶと、次のように「一致なし」エラーになることがある。

```
No matches found for <old_strに指定した文字列>.
```

### 原因

`notion-fetch`が返す`text`は、リンク等を読みやすく変換した**表示用テキスト**であり、実際にNotion側に保存されている生Markdownと文字列表現が異なる場合がある（特にURLを含むリンク周辺）。

### 対処法

- 部分置換（`update_content`）で原因不明の不一致が出たら、深追いせず`replace_content`（本文全体を書き換え）に切り替える方が確実
- 特にリンクを含む段落を`old_str`に使うのは避け、リンクを含まない前後の短い文だけで囲む方が一致しやすい

---

## 17. query-data-sources（SQL）は日本語列名を必ずダブルクォートで囲む。IN(?,?)のパラメータ化はエラーになりやすい

### 症状

```sql
SELECT 名前, 注文番号 FROM "collection://..." WHERE 注文番号 IN (?, ?)
```

のように列名をダブルクォートで囲まずに書き、かつ`WHERE`句で`IN (?, ?)`という複数バインドパラメータを使ったところ、次のエラーになった。

```
Failed to execute query: Only a single SELECT query against the provided collection views is permitted. Reason: the query could not be parsed safely.
```

### 正しい形式

列名を`"名前"`のようにダブルクォートで囲み、`IN(?,?)`のようなパラメータ配列展開は避けて`=`の`OR`連結に書き換えたところ成功した。

```sql
SELECT "名前", "注文番号" FROM "collection://..." WHERE "注文番号" = '250-XXXXXXX-XXXXXXX' OR "注文番号" = '250-YYYYYYY-YYYYYYY'
```

### 教訓

- 日本語（非ASCII）列名は`SELECT`・`WHERE`のどちらでも必ずダブルクォートで囲む
- 複数値の一致判定は`IN (?, ?)`のようなパラメータ配列展開に頼らず、`= 'A' OR = 'B'`のように単純な等価比較を並べる方が安定する

---

## まとめ: データ操作の安全な手順

```
1. notion-fetch でスキーマ確認（プロパティ名、型、オプション値）
2. 必要なら update-data-source でオプション追加
3. 少量（1〜2件）でテスト挿入
4. 成功を確認してからバッチ挿入
5. 挿入後に notion-search で結果検証
```
