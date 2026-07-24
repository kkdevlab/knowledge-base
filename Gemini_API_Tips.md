# Gemini API Tips

Google Gemini API (`generativelanguage.googleapis.com`) の generateContent REST呼び出しに関する実地確認済みTips。

---

## 構造化出力（JSON Schema指定）は2方式ある

`generationConfig` 配下に新旧2つの構造化出力指定方法が共存している。

### 方式A: responseFormat（新方式・enum型で厳格）

```json
"generationConfig": {
  "responseFormat": {
    "text": {
      "mimeType": "APPLICATION_JSON",
      "schema": { "type": "object", "properties": { ... } }
    }
  }
}
```

- `mimeType` は **enum**（`MIME_TYPE_UNSPECIFIED` / `APPLICATION_JSON` / `TEXT_PLAIN`）。`"application/json"` のようなMIME文字列を渡すと400エラー（`Invalid value at 'generation_config.response_format.text.mime_type'`）
- `schema` 配下のtype値は**小文字**（`object`/`array`/`string`/`integer`等）。`anyOf` 等JSON Schema寄りの表現も使える

### 方式B: responseMimeType + responseSchema（旧方式・後方互換として現存、こちらの方が実運用で安定）

```json
"generationConfig": {
  "responseMimeType": "application/json",
  "responseSchema": { "type": "OBJECT", "properties": { ... } }
}
```

- `responseMimeType` は素の文字列（`"application/json"`）でよい
- `responseSchema` のtype値は**大文字enum**（`STRING`/`NUMBER`/`INTEGER`/`BOOLEAN`/`ARRAY`/`OBJECT`）。小文字を渡すとエラーになる
- 2つの方式でtype値の大文字/小文字ルールが逆になっている点に注意（唯一混同しやすいポイント）

**実運用では方式Bの方がドキュメント通りに素直に動く。方式Aはenum値の取り違えで400エラーになりやすいので、迷ったら方式Bを使う。**

## モデル名は都度 ListModels で確認する

- `gemini-2.5-flash` のように一見現行世代に見えるモデル名でも、**新規発行のAPIキーでは404（"no longer available to new users"）**になることがある（旧世代が新規ユーザーへの提供を先に締め切るケースがある）
- `GET https://generativelanguage.googleapis.com/v1beta/models`（ヘッダー `x-goog-api-key`）で `supportedGenerationMethods` に `generateContent` を含むモデル一覧を確認してから使うこと
- エイリアス `gemini-flash-latest` / `gemini-pro-latest` はGoogleが指す実体を随時更新するため、個別モデル名の廃止に振り回されにくい（2026-07時点、`gemini-flash-latest`の実体は`gemini-3.5-flash`）

## 無料枠のレート制限（RPD）はモデルごとに独立している

- 無料枠は「1日あたりのリクエスト数（RPD）」の上限がモデルごとに個別に設定されている（例: 2026-07時点で`gemini-3.5-flash`は20回/日）。全モデル合算のクォータではないため、あるモデルが429（`RESOURCE_EXHAUSTED`）になっても、別モデルはまだ枠が残っていることが多い
- 429のレスポンスボディに`quotaId`（例: `GenerateRequestsPerDayPerProjectPerModel-FreeTier`）と`quotaValue`が含まれ、日次上限であることが明記される。日次リセットなので、同じモデルへ即座にリトライしても無駄（`retryDelay`が数十秒でも実際は翌日まで解消しない）
- 429と5xx（一時的な高負荷）は原因が異なるため、エラーハンドリングでは区別した方がよい（429=クォータ切れで別モデルへの切替が有効、5xx=一時障害で同モデルへの再試行が有効）
- 2026-07-24時点で無料枠利用可能を確認したモデル: `gemini-3.5-flash` / `gemini-3.6-flash` / `gemini-3.5-flash-lite`。`gemini-2.5-flash`は新規ユーザーだけでなく既存プロジェクトからも404（廃止済み）

## 認証

- APIキーは `x-goog-api-key` ヘッダーで渡せる（`?key=`クエリパラメータに書くとログ・URL履歴に残りやすいので、可能ならヘッダー方式を優先）

## classic generateContent（REST）のTool objectフィールド名はcamelCase

- 新しい「Interactions API」（`/v1beta/interactions`）のドキュメント例では`{"type": "google_search"}`のようなsnake_caseの表記が出てくるが、従来の`/v1beta/models/{model}:generateContent`エンドポイントではフィールド名はcamelCaseになる
  ```json
  "tools": [
    { "googleSearch": {} },
    { "urlContext": {} }
  ]
  ```
- `googleSearch`（Web検索によるgrounding）は無料枠だと`429 RESOURCE_EXHAUSTED`になることがある（`urlContext`単体・toolなしはどちらも成功したため、`googleSearch`固有の割り当て制限と判断できる）
- `urlContext`（プロンプト内に書いたURLを直接取得する）は無料枠でも問題なく動作する。自由なWeb検索はできないが、確認先URLが決まっている用途ならこれで十分

## generationConfig.thinkingConfig.thinkingLevelでreasoning effortを指定できる

- classic generateContentでは`generationConfig.thinkingConfig.thinkingLevel`に`minimal`/`low`/`medium`/`high`のいずれかを指定する（Gemini 3以降のモデル対象。古いモデルに指定するとエラーになる）
  ```json
  "generationConfig": {
    "thinkingConfig": { "thinkingLevel": "high" }
  }
  ```
- 旧方式の`thinkingBudget`（トークン数指定）と`thinkingLevel`は同時指定不可
- 未指定時の既定値はGemini 3系で"medium"（モデル側の既定であり、ローカルの設定ファイル等には依存しない）

---
*(2026-07-12 DocomoMailGuard Gemini版構築時に実地確認)*
*(2026-07-17 PromptRunner ai-news統合・effort機能追加時に実地確認)*
