# Tasker 6.7.x-beta 開発者Q&A・確定挙動ノート

r/tasker の [DEV] リリーススレッド（開発者 joaomgcd の直接回答）から、**技術的に有用な確定挙動・回避策・既知バグ**を抽出したもの。単なる感想・要望のみのやりとりは除外。

- 出典: [6.7.5-beta スレッド](https://www.reddit.com/r/tasker/comments/1u7d04g/dev_tasker_675beta_projects_in_new_ui_write_json/)（クローズ想定・アーカイブ）／ [6.7.6-beta スレッド](https://www.reddit.com/r/tasker/comments/1ull0si/dev_tasker_676beta_inline_projects_in_new_ui/)
- 変更点そのものの一覧 → `Tasker_Version_History.md`
- Scene V2 の一般仕様 → `Tasker_ScenesV2_67x-beta.md` / `Tasker_ScenesV2_Official_Manual.md`
- 最終取得元 JSON: `Tasker/Doc/tasker_675beta.json` / `tasker_676beta.json`

---

## A. Scene V2 WebView ↔ Tasker JavaScript ブリッジ（6.7.5 新規・6.7.6 拡張）

WebView 内の JavaScript から Tasker アクションを直接呼べる `Tasker.*` ブリッジ。**全アクションが Promise を返す**ため `await` で逐次・並列制御できる。

- 公式リファレンス:
  - `https://tasker.joaoapps.com/userguide/en/scenes_v2.html#webview-js-bridge`
  - `https://tasker.joaoapps.com/userguide/en/scenes_v2/webview-actions.html`
- **タスク呼び出し**: `var started = Tasker.runTask('My Task', { foo: 'bar', '%num': '3' }, 10);`
  - 第2引数=変数（`%num` のように % 付きでも渡せる）、第3引数=priority、戻り値=started(bool)
- **変数セット**: `Tasker.setVariable("test_var", text);`（名前は % なし）
- **アクション出力の取得**: `const out = await Tasker.runShell({ cmd: 'getprop ro.product.model' }); out.stdout`
  - 解決オブジェクトは**出力変数名（先頭 % なし）**をキーに持つ
- **例**: `await Tasker.flash({ text: 'Step 1' });` / `await Tasker.vibrate({ time: 300 });`
- **エラー処理**: 失敗したアクションは通常の JS 例外として reject される → `try/catch` が効く。`Promise.all([...])` で並列実行も可
- **HTML要素をドラッグハンドル化**（6.7.6）: CSS クラス `tasker-drag-handle` / `tasker-drag-handle-x` / `tasker-drag-handle-y`（全方向／横のみ／縦のみ）
- **Material 3 パレット提供**（6.7.6）: WebView に `--tasker-surface` / `--tasker-onSurface` / `--tasker-primary` 等の CSS 変数が渡る → HTML がアプリテーマに追従できる

**AutoTools Webscreens の代替**: dev 見解として「Scene V2 WebView は AutoTools Webscreens ができることは全部できるはず」。移行候補。

**既知の制限・バグ（6.7.5 時点の報告）**:
- WebView から `setVariable` した値が、**同一シーンの他コンポーネントや後続タスクに伝播しない**（変数メニューには反映される）→ 6.7.6 で「WebView での変数セットが同一画面の別コンポーネントの同名変数に影響する」系の修正あり。**要実機再検証**
- `Tasker.launchApp({app:v})` が効かない（報告時点）
- `readFile()` が期待値（ファイル内容）ではなくメタ情報を返す報告あり
- 旧 Scenes v1 の関数（`flash()`, `shell()`, `sendIntent()` 等）は初期未対応だったが、要望を受け dev が追加（`shell()` は必須と判断）

---

## B. Scene V2 実用テクニック（dev 確認済み）

- **シーンが表示中かの判定**: `Get Scene v2 Values`(code 483) を使う。**シーンが非表示ならエラーで終わる**ことを利用して判定する（v1 の「Test Scene」相当は v2 に無い）。
  - ※ UI の要素ピッカー（虫めがね／magic glass）は**名前付きの既製シーン**でのみ機能する。インライン JSON では使えない
- **イベントハンドラの条件分岐（if/else 相当）**: 「**Only trigger when**」フィールドに `%action`（真/セット時）や `!%action`（偽/未セット時）を入れて発火を出し分ける
- **Screen Shown / Screen Hidden で変数トグル**: 6.7.5 初期は Screen Hidden 側で Variable Set が効かないバグ → dev 修正済み。あわせて Scene V2 に **Variable Clear イベントアクション**が追加された
- **ボタンからタスク呼び出し（JSON 直書き）**:
  ```json
  "eventHandlers": { "handlers": [ {
    "events": [ { "type": "click" } ],
    "actions": [ { "type": "RunTask", "task": "Test" } ]
  } ] }
  ```
- **Arrays Merge Template**（6.7.6 新規）: Tasker 配列を行リストへ動的展開。フィルタ付きドロップダウン例 → template 内の要素に `"showWhen": "%names contains %query"`。※要素数が多いと一部しか描画されない不具合報告あり（**要確認**）
- **Show When / Apply When のマッチ**（6.7.6）: `matches`（Tasker パターン）/ `matchesr`（正規表現）が使える
- **index が 1-based に統一**（6.7.6 fix、従来 0-based）
- **タップ座標**（6.7.5）: tap系イベントで `%sv2_tap_x` / `%sv2_tap_y` をタスクに渡せる

---

## C. JSON ネイティブ書き込み（6.7.5 新規・6.7.6 拡張）

- 変数名の**ドット記法** `%json.address.city` で JSON 構造を生成（**中間構造も自動生成**）。
  - 対応アクション（6.7.5）: Variable Set / Variable Clear / Array Set / Array Push / Array Pop
  - 6.7.6 で **Multiple Variables Set** も JSON 書き込み対応
- **Format JSON** アクション（6.7.5 新規）: minify / pretty-print（Indent 指定）。**Overwrite Source Variable** オプションで入力変数を上書き可
- dev tips: タスク内の**個別アクションを単体で実行できる**（1アクションだけテスト実行）ことは意外と知られていない

> kklab 補足: 従来 JavaScriptlet の `local()` スコープ問題・改行エスケープ問題の回避策になり得る（`Tasker/CLAUDE.md` 参照）。

---

## D. SQL Query (code 667) Raw モードの確定挙動

（`Tasker/Doc/troubleshoot.md` の「DROP 非対応」トラブルシュートを補強する dev/ユーザー確認）

- Raw モードで **INSERT / UPDATE / DELETE / ALTER** は動作する
- **CREATE INDEX は `SQLITE_READONLY` で失敗**する（CREATE 系が read/write 接続を開くキーワード判定に入っていない模様）。dev は「regression ではなく新規要望」と確認 → 将来対応待ち
- **セミコロン区切りの複数文は非対応**。1アクション＝1文が原則（複数書いても**最後の1文のみ**実行される）。初期セットアップ等でまとめて流したい場合は Run Shell + sqlite3 で代替

---

## E. 端末固有の注意 / 「Tasker のバグではない」と確定した事象

- **長時間バックグラウンド後、画面をタップするまでシーンが表示されない**（One UI 8+ / Galaxy S26U で再現）
  → dev 見解：**Tasker のバグではなく、端末が長時間バックグラウンドのアプリを扱う際のクセ**。修正不可。回避は端末側の挙動対策（例: 表示直前に Tasker を開閉する等をユーザーが検討）
- **Vibrate Pattern が Galaxy S26U で一部効かない**: `0,600` は振動するが `0,600,150,200` は**無反応（エラーも出ない）**。未解決（dev が URI 待ち）
- **Device Admin 制限で Tasker がグレーアウト**: Android 設定 > アプリ > Tasker > 3点メニュー > Unblock（Restricted control の解除）

---

## F. New UI（2026 UI）に関する dev の方針・回答

- New UI は **opt-in**（明示的に有効化が必要）。旧UIから自動で切り替わらない
- 画面内の「switch to old UI」ボタンは**その画面限りの一時切替**。恒久的に旧UIに留まるには**旧UIの設定で新UIを無効化**する
- **Projects を新UIに第一級概念として復活**（6.7.5）→ 6.7.6 で**インライン表示**方式も追加。dev はサイドバー/inline/collapsible の3案を試行中でフィードバック募集中
  - dev の立場: Projects は旧UIでも実質「スロット/フィルタ」であり、タグと機能は近い。「All（全表示）」ボタンはタグの横断表示に必要
- **未対応（今後対応予定）**: スコープ変数（project/profile/task）、サードパーティ プラグイン、変数の値表示
- **匿名タスク**: タスクは**名前を付けるまで匿名**。タスクにコンテキストtriggerを足すと名前を付けるのは**プロファイル**側で、タスクは匿名のまま
- 旧UIには無かった **Project 単位の有効/無効マスタースイッチ**が新UIで動作（プロファイルは無効プロジェクト下で paused 表示）

---

## G. その他 dev 回答

- **どこからでも webhook / POST でトリガー**は FCM で既に可能: `https://tasker.joaoapps.com/userguide/en/fcm.html`
- **Extra Trigger アプリ**（Home / Car / Bixby 等）は**個別インストールが必要**。dev は意図的（全ユーザーのランチャーを汚さない＋Secondary App では触れない Samsung 設定等に使えるため）。容量は極小

---

## H. 6.7.6-beta で修正された 6.7.5 報告バグ（追跡メモ）

6.7.5 スレッドで報告 → 6.7.6 changelog で解消が確認できたもの:

- receive intent が 1 つの intent で 2 回発火する → **Fixed**
- Tasker の Preferences 画面から戻ると Wifi Connected 等の **exit task が誤発火** → **Fixed**
- WebView で変数をセットすると同一画面の別コンポーネントの同名変数に影響 → **Fixed**（要再検証・A項参照）
- プロジェクトのアプリアイコンが全部同じアイコンになる → **Fixed**
- 「Overlay With Result」を同一 id で2連続表示すると2つ目が早期終了 → **Fixed**
- List Dialog の First Visible Index がライトテーマで見えない／ダーク⇔ライト切替でクラッシュ → **Fixed**
- 多数の Scene V2 クラッシュ・マルチディスプレイ問題 → **Fixed**
