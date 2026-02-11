# Notion MCP ツール Tips & 注意点

最終更新: 2026-02-11

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

## まとめ: データ操作の安全な手順

```
1. notion-fetch でスキーマ確認（プロパティ名、型、オプション値）
2. 必要なら update-data-source でオプション追加
3. 少量（1〜2件）でテスト挿入
4. 成功を確認してからバッチ挿入
5. 挿入後に notion-search で結果検証
```
