# Tasker Scene V2 公式マニュアル（ローカル保存版）

**出典**: https://tasker.joaoapps.com/userguide/en/scenes_v2.html
**取得日**: 2026-06-26（前回 2026-03-18 から大幅改訂）
**セクション数**: 19

> このページは JS で各セクション（`scenes_v2/01.html`〜`19.html`）を動的読込する構成。
> WebFetch では本文が取れない（"Loading sections..." の殻だけ）。取得は `fetch_url.py` で
> `scenes_v2/NN.html` を個別取得する（手順は `feedback_webfetch_fallback` メモ参照）。

---

## 01. はじめに（Introduction）

Tasker Screen Builder は、コーディングなしでカスタムスクリーンを作成できるビジュアルデザインツール。入力フォーム・ダッシュボード・コントロールパネル・ステータス表示などを作れる。

- レイアウトは**ツリー状に並べた components** で構成。コンテナ（Column / Row / Box）で配置を制御し、コンテンツ要素（Text / Button / Image 等）を追加、modifiers でスタイリング。
- 内部は JSON だが、触る必要はない（触りたければ触れる）。
- 表示形態: **フルスクリーンアクティビティ / フローティングダイアログ / 他アプリ上のオーバーレイ / スクリーンセーバー（Dream）**。
- Tasker タスクに接続してインタラクティブにできる。

---

## 02. クイックスタート（Quick Start）

5ステップで最初のインタラクティブ画面を作る:

1. **新規 scene 作成** — SCENES タブ → Add → 名前入力。Scene Type ダイアログで **Legacy（旧エディタ）** か **Modern (V2)（本マニュアル対象）** を選ぶ。Modern (V2) を選択。
2. **開始テンプレート選択** — "What do you want to create?" で **Simple Layout**（空白：1コンテナ+1 Text、フル制御向き）or **Full Page Layout**（top bar・nav bar・FAB 構成済み、アプリ風向き）。保存済みテンプレートも下に並ぶ。
3. **Text を編集** — プレビューで Text をタップ（1回目で選択、2回目で Settings タブへ）。内容を変更。
4. **Button を追加** — Elements タブ → Text 選択中に "+" FAB → **Add After** で兄弟追加 → Button を選択 → ラベルを "Close" に。
5. **Button で画面を閉じる** — Button 選択 → Event Handling タブ → Add handler → Click イベント → **Dismiss Layout** アクション追加。閉じる時、画面が書いた変数が呼び出しタスクへ返る。

> アンドゥ無制限（undo/redo 長押しで履歴一覧）。完成後は **Show Scene v2** アクションで表示。

---

## 03. エディタ概要（Editor Overview）

編集パネル + ライブプレビューパネルの2分割。編集パネルは**9タブ**（端に並ぶ）。広い画面では左右、狭い画面では上下スタック（区切りはドラッグでリサイズ）。

| タブ | 役割 |
|------|------|
| **Elements** | コンポーネントツリー（ファイルエクスプローラー形式）。選択・ドラッグ並べ替え・親子間移動・"+"追加。長押しで複数選択（コピー/カット/削除）。showWhen で隠れた要素は目に斜線アイコン |
| **Settings** | 選択要素の全編集。General（ID・showWhen・要素固有プロパティ）。コンテナは Provides セクションも。**未選択時は Screen Properties** を表示 |
| **Modifiers** | スタイリング層（Padding/Background/Size/Border/Shadow 等）の管理。各 modifier はカード、ドラッグで順序変更（順序が結果を左右）。Drag-to-Move 設定もここ |
| **State** | 入力要素（Switch/TextInput 等）の現在値を指定変数へ**連続書き込み**（output key ↔ 変数のペア） |
| **Event Handling** | ユーザー操作時の挙動。Add handler → イベント選択（Click/Hold/Multi-Click/Swipe 等）→ アクション列。任意の condition、propagation 停止も |
| **Actions** | 選択要素が外部から命令実行できる **component action handler** の一覧（ID 付き）。テスト可能なものは run ボタンでプレビュー実行 |
| **Animation** | コンテナの content transition（子の出入り/変化アニメ）。子を持つ要素のみ有効 |
| **Code** | 生 JSON 編集。双方向同期。**VALID（緑）/ INVALID（赤）** バッジ。Cut/Copy/Paste |
| **AI** | 自然言語でレイアウト生成・修正。全体 or 選択要素。system instructions のコピー/エクスポート可 |

**その他のパネル/モード:**
- **Test Variables**（オーバーフローメニューのフラスコアイコン）: 一時変数で showWhen 挙動をプレビュー。プレビューで変数解決を ON にすると text 等も test 値で描画。実行時には未使用。
- **Preview**: ライブ描画。タップで選択（再タップで Settings へ）。既定は非インタラクティブ → オーバーフローメニューの Interactive mode でハンドラ実行可。マルチディスプレイ時はサブ画面へ移動可。
- **Editing Modes**: **Screen mode**（root + name 付き完全画面）/ **Element mode**（screen ラッパー無しの単体コンポーネント JSON。Array Merge / Variable で再利用するテンプレート用）。アクションフィールドから開くと既存内容からモード自動判定。

---

## 04. レイアウト基礎（Layout Fundamentals）

### 3つの基本コンテナ

| コンテナ | 説明 |
|---------|------|
| **Column** | 子を縦に配置（上→下）。多くの画面でルートに使う |
| **Row** | 子を横に配置（左→右）。ツールバー・横並びに |
| **Box** | 子を重ねて配置（最後の子が最前面）。各子が Align modifier で自由配置可。オーバーレイ・バッジ・画像上テキスト等、自由配置レイアウト用 |

### ネストで任意レイアウト
- Column 内に Row = グリッド状 / Column（Row ヘッダー + Column ボディ）= アプリ風 / Box（Image + Text）= オーバーレイカード

### Alignment & Arrangement
- **alignment** = メイン軸と直交方向の位置、**arrangement** = メイン軸方向の分配
- **Column**: `horizontalAlignment`（Start / Center / End）+ `verticalArrangement`（Top / Center / Bottom / SpaceBetween / SpaceAround / SpaceEvenly / SpacedBy）
- **Row**: `horizontalArrangement`（同上値）+ `verticalAlignment`（Top / Center / Bottom）
- **Box**: `contentAlignment` 9ポジション（TopStart / TopCenter / TopEnd / CenterStart / Center / CenterEnd / BottomStart / BottomCenter / BottomEnd）。個別の子は Align modifier で上書き可
- `SpaceBetween` = 先頭と末尾を両端へ、間を均等

### SpacedBy
`verticalArrangement` / `horizontalArrangement` を `SpacedBy` にすると `spacing`（dp）フィールドが出て子間に固定間隔。Spacer 挿入より簡潔。`spacing` は SpacedBy のときのみ有効。

### 3つ以外のコンテナ（特殊用途）
- **Card** — elevation + 角丸 Material 3 背景の縦スタック面
- **FlowRow / FlowColumn** — Row/Column だが折り返しあり
- **FlexBox** — CSS Flexbox 風（折返し・比例サイズ・個別整列）
- **Scaffold / TopAppBar / BottomAppBar / NavigationBar** — アプリ風画面の構造部品

---

## 05. コンポーネント（Components）

各要素は一意の id、任意の showWhen、type 固有プロパティを持つ。

### カテゴリ（"+"ピッカーのトップレベル4分類）

| カテゴリ | コンポーネント |
|---------|--------------|
| **Layout** | Column, Row, FlowRow, FlowColumn, Box, FlexBox, Card, Scaffold, TopAppBar, BottomAppBar, NavigationBar, Dropdown, Spacer, Variable, Placeholder |
| **Display** | Text, Image, Divider, ProgressBar, WebView |
| **Input** | Button, IconButton, FloatingActionButton, Switch, Checkbox, TextInput, Slider, RangeSlider, NavigationItem, SegmentedButtonRow, Segment, Camera |
| **Media** | Video |

ピッカーは現在位置で有効な要素のみ表示。親制限あり: NavigationItem は NavigationBar 内のみ / Segment は SegmentedButtonRow 内のみ / FloatingActionButton は Scaffold 内のみ。

### 重要概念
- **Component ID**: 自動生成 UUID。意味ある名前へ変更推奨。ツリー操作・イベントハンドラ・Update Scene v2 のターゲットに使う。改名は全参照を自動更新。Variable 展開時は `_1`/`_2`… サフィックス付与。
- **showWhen**: false でその要素と子を**レイアウトから完全除去**（場所も取らない）。
- **showWhenMode**: false時の挙動。**Gone**（既定・場所も詰める）/ **Invisible**（場所は確保し非描画）。
- **Slots**: 子を受ける名前付き領域。単純コンテナは `children` 1つ。Scaffold/TopAppBar は複数スロット（受入型・個数制限あり）。
- **provides プロパティ**: コンテナ（Column, Row, FlowRow, FlowColumn, Box, FlexBox, Card, Scaffold）が `@{key}` で子孫へ値注入。

### Layout コンポーネント（主要プロパティ）

- **Column**（"Vertical Column"）: `horizontalAlignment`(Start) / `verticalArrangement`(Top) / `spacing`(SpacedBy時)
- **Row**（"Horizontal Row"）: `horizontalArrangement`(Start) / `verticalAlignment`(Top) / `spacing`
- **Box**（"Box (Z-Stack)"）: `contentAlignment`(TopStart)
- **FlowRow**: `horizontalArrangement` / `verticalArrangement`（折返し行を分配）/ `itemVerticalAlignment` / `spacingHorizontal` / `spacingVertical` / `maxItemsPerLine` / `maxLines` / `overflow`(Clip/Visible)
- **FlowColumn**: 上記の縦版（`itemHorizontalAlignment` 等）
- **FlexBox**（Experimental）: `direction`(Row/RowReverse/Column/ColumnReverse) / `wrap`(NoWrap/Wrap/WrapReverse) / `justifyContent` / `alignItems`(Start/End/Center/Stretch/Baseline) / `alignContent` / `gap` / `rowGap` / `columnGap`。子は Flex modifier 使用可
- **Card**: `style`(Filled/Elevated/Outlined) / `elevation`(Elevated時, dp)
- **Scaffold**: `containerColor` / `contentColor` / `fabPosition`(End/Center)。スロット: `topBar` / `bottomBar` / `floatingActionButton`(FAB max1) / `content`。新規時 TopAppBar+NavigationBar+FAB 入り
- **TopAppBar**: `containerColor` / `contentColor`。スロット: `title`(max1) / `navigationIcon`(max1, 必須) / `actions`
- **BottomAppBar**: `containerColor` / `contentColor`。スロット: `content`（Row 並び）
- **NavigationBar**: `containerColor` / `contentColor` / `selectedIndex`(0, 変数式可)。スロット: `content`(NavigationItem のみ)。単一選択をバーが管理（NavigationItem 自身に selected 無し）
- **Dropdown**: `isExpanded`("" =自動) / `triggerShowsSelected`(true=Select モード) / `selectedIndex` / `triggerMatchesContentWidth`(true)。スロット: `trigger`(max1) / `content`
- **Spacer**: 既定 16dp Size modifier 付き。Size modifier で寸法調整
- **Variable**: `key`（% 付き Tasker 変数名。JSON を保持し実行時に置換）。→ 09章
- **Placeholder**: プロパティ無し。実行時は何も描画せず、エディタのみラベル箱。element-mode テンプレのスロット用

### Display コンポーネント

- **Text**: `text`(必須) / `textSize`(14, sp) / `color`("") / `textAlign`(Start) / `verticalAlignment`(Top) / `maxLines`("")。**font-weight プロパティは無い**（太字不可、色/サイズで強調）
- **Image**: `url`("") = URL / `icon:Settings` / インライン SVG。`errorUrl` / `contentScale`(Fit/Crop/FillBounds) / `tint`
- **Divider**: `color`("") / `thickness`(1, dp)
- **ProgressBar**（"Progress Bar"）: `style`(Linear/Circular) / `mode`(Determinate/Indeterminate) / `progress` / `minProgress`(0) / `maxProgress`(100) / `color` / `trackColor` / `strokeCap`(Round/Square) / `strokeWidth`(4, Circular時) / `showLabel` / `animateChanges` / `animationDuration`(300ms)。`animate`/`stop` アクション、`animation_completed` イベント
- **WebView**: → 下記「WebView 詳細」

### WebView 詳細（このセット中最もリッチ）

ネイティブ Android WebView でリモートページ/ローカルファイル/インライン HTML を描画。設定パネル・命令アクション・ライフサイクルイベント・連続同期 State・双方向 JS ブリッジを持つ。新規時 FillSize modifier 付き。

**Content & Load Modes**（`content` プロパティの値から自動判定、別途モードスイッチ無し）:
- `http://` / `https://` / `://` を含む文字列 → リモート URL
- `/` 始まり → ローカルファイルパス（`file:///...`）
- `file://` 始まり → ローカルファイル URL
- それ以外 → インライン HTML（`loadDataWithBaseURL`）
- ※ スキーム無しの `example.com` はインライン HTML 扱い。サイト読込は `https://example.com` と完全指定

**WebView プロパティ（抜粋。全て変数式可、bool は true/false）:**
- Content: `content`（主プロパティ）/ `baseUrl`（インライン HTML 時のみ）
- Appearance: `darkMode`(Auto/AlgorithmicDark/ForceLight) / `textZoom`(100) / `backgroundColor`
- JavaScript & Storage: `javaScriptEnabled`(true) / `domStorageEnabled`(true) / `databaseEnabled`(false) / `jsBridgeEnabled`(true)
- Zoom & Viewport: `supportZoom`(true) / `builtInZoomControls`(false) / `displayZoomControls`(false) / `useWideViewPort`(false) / `loadWithOverviewMode`(false)
- Security: `mixedContentMode`(CompatibilityMode/AlwaysAllow/NeverAllow) / `allowFileAccess`(false) / `allowContentAccess`(true)
- Media & Network: `mediaPlaybackRequiresUserGesture`(true) / `userAgentString`("") / `cacheMode`(Default/NoCache/CacheOnly/CacheElseNetwork)
- Location: `geolocationEnabled`(false)

**Reactive Variables**（JS ブリッジ ON 時、ページ内で Tasker 変数をライブ解決・無リロード更新）:
- text/属性/`<style>` 内: `%varName` をそのまま書くと値に置換、変化時に再置換
- `<script>` 内: `%var` は**値式**（文字列ではない）。`"Hello " + %name` のように。テンプレートリテラル `` `${%query}` `` も可。`x % 10` のような剰余は無変更。ブリッジ OFF だと生 HTML のみリロードで解決。完全リアクティブには `getVariable()` + `onVariableChanged`

**JavaScript Bridge**（`jsBridgeEnabled` ON 時、既定名 `Tasker` のオブジェクト注入）主要メソッド:
- 変数: `setVariable(name,value)` / `setVariables(obj)` / `clearVariable(name)` / `clearVariables(names)` / `getVariable(key)` / `getVariables()`
- 要素操作: `updateElementProperties(id,prop,value)` / `executeAction(id,action,params)`(Promise)
- レイアウト: `showLayout(config)` / `showLayoutForResult(config,timeoutMs)` / `waitForLayoutResult(id,timeoutMs)` / `dismissLayout(id)` / `dismissAllLayouts()` / `isLayoutActive(id)`
- その他: `listDisplays()` / `updateOverlayConfig(config)` / `extractOutputVariables(layout)` / `getCurrentScreenId()` / `onVariableChanged(names,cb)`
- タスク: `runTask({name,variables,priority})` / `sendCommand({command})` / `runTaskForResult({name,variables,priority,timeoutMs})` → `{returnValue, variables}`
- `...OnScreen` / `...ForScreen` 変種で別画面ターゲット可。同期 JSON 系は `...Json` 変種あり
- **命名規則**: HTML/CSS/script では `%count`、ブリッジメソッド引数では bare 名 `'count'`。全値は文字列
- runTask の variables 値・task 名は `%variable` 補間可（scene→global の順）。キーは小文字ローカル変数名のみ（MyVar 等は無視）
- **⚠️ セキュリティ**: ブリッジ ON はそのページ（リモート http(s) 含む）に変数読み書き・タスク実行・コマンド送信を許す。**信頼するコンテンツのみ ON**

**Theme colors**（WebView 内で Material 3 テーマ色利用、ライブ更新）:
- CSS（推奨・JS 不要）: `:root` の `--tasker-<role>` カスタムプロパティ（例 `--tasker-primary`）
- JS: `Tasker.colors` または `window.taskerColors`（ブリッジ無しでも後者は使える）。`tasker-colors-changed` イベントで反応
- 全 Material 3 ロール（primary, onPrimary, …, surfaceContainer*, inverse*, *Fixed*）。値は CSS 文字列
- ※ JavaScript 設定が必要（CSS変数/window.taskerColors はブリッジ不要、Tasker.colors はブリッジ要）
- Tip: Dark Mode=Auto なら `<meta name="color-scheme" content="light dark">` を宣言

**WebView Actions**（イベントハンドラ / Update Scene v2 / executeAction から）:
`reload` / `stop_loading` / `go_back` / `go_forward` / `clear_cache` / `clear_history` / `page_up` / `page_down` / `page_top` / `page_bottom` / `load_url`(url) / `find`(text→match_count) / `find_next` / `zoom`(percent)

**WebView Events**: `page_loaded`(url,title) / `page_error`(url,error_code,description) / `url_changed`(url) / `console_message`(message,level,line_number) / `progress_changed`(progress) / `scroll_changed`(scroll_x,scroll_y)

**WebView States**（変数へ連続書込）: `url` / `progress` / `scroll`(scroll_x,scroll_y)

**Tasker Actions（ブリッジ経由）**: `await Tasker.<name>({...})` で個別 Tasker アクションを Promise 実行（flash, runShell 等多数）。一覧は別ページ「Tasker Actions Reference」。

### Input コンポーネント

- **Button**: `text`(必須) / `textColor` / `buttonColor`
- **IconButton**（"Icon Button", 48dp タッチ域）: `icon`(`icon:Settings`等) / `tint` / `contentScale`
- **FloatingActionButton**: `containerColor` / `contentColor`。スロット `content`(max1, 通常 Image)。Scaffold の floatingActionButton スロットのみ
- **Switch**: `checked`(false, 変数式可)。`checked_changed` イベント、State 同期可
- **Checkbox**: `checked`(false)。Switch と同モデル
- **TextInput**（"Text Input"）: `label` / `placeholder` / `text` / `textSize`(14) / `color` / `multiLine`(false) / `autoFocus`(false) / `style`(outlined/Normal) / `borderColor` / `focusedBorderColor` / `labelColor` / `placeholderColor` / `backgroundColor` / `cursorColor` / `shape`。`text_changed`/`key_pressed` イベント、focus/unfocus アクション。**⚠️ オーバーレイモードでは動作しない**（Android 制限）
- **Slider**: `value`(0) / `min`(0) / `max`(1) / `steps`(0=連続) / `thumbColor` / `activeTrackColor` / `inactiveTrackColor` / `activeTickColor` / `inactiveTickColor`。`value_changed`、value/percent 同期
- **RangeSlider**（"Range Slider"）: `start`(0) / `end`(1) / `min` / `max` / `steps` + 各色。start/end/percent 同期
- **NavigationItem**: `icon` / `label` / `alwaysShowLabel`(true) / `selectedIconColor` / `unselectedIconColor` / `selectedLabelColor` / `unselectedLabelColor` / `indicatorColor` / `unselectedIndicatorColor`。NavigationBar 内のみ
- **SegmentedButtonRow**: `selectionMode`(Single/Multi) / `selectedIndices`(CSV or JSON配列) / `allowDeselect`(Single時)。スロット `content`(Segment のみ)。`selection_changed`、`toggleItem` アクション
- **Segment**（型名 `SegmentedButtonItem`）: `label` / `icon`。SegmentedButtonRow 内のみ
- **Camera**: `lens`(`<facing>[:<tier>]` 例 `front`/`back:wide`) / `contentScale`(Crop) / `showErrorMessage`(true)。`switchLens` アクション、`ready`/`error` イベント。camera 権限要

### Media コンポーネント

- **Video**: `source`("") / `autoPlay`(false) / `loop`(false) / `muted`(false) / `volume`(1.0) / `speed`(1.0, 0.25〜4.0) / `contentScale`(Fit)
- **Source Detection**: YouTube URL（youtube.com/youtu.be）→ 埋込プレーヤー（確定）/ `.gif` → アニメ GIF / その他 → 動画ファイル。GIF・動画はヒューリスティック判定で失敗時もう一方へ自動フォールバック
- **Video Actions**: `play` / `pause` / `togglePlayback`(→is_playing) / `seek`(position ms, GIF無視)
- **Video Events**: `loaded`(duration) / `ended` / `error`(code,message) / `buffering`(is_buffering)
- ※ GIF は seek/speed/volume/muted 無視、play/pause/toggle は Android 9+ 必要

### Common Properties（全要素共通）
`id` / `modifiers` / `showWhen` / `showWhenMode`(Gone/Invisible) / `eventHandlers` / `contentTransition` / `treeLabel`（エディタツリーの表示名）

### Content Transition（Animation タブで設定）
コンテンツ入替時（showWhen 反転・変数変化）のアニメ: **Crossfade** / **Blur** / **CardFlip**。未設定なら無アニメ即時入替。

---

## 06. モディファイア（Modifiers）

スタイル。リストの**上から下へ適用**され、各 modifier が前のものを包む（順序が結果を決める）。

例: Padding→Background は色の内側に余白／Background→Padding は色の外側に透明余白。問題時はまず順序を確認。

### Parent-Scoped Modifiers（親が対応する場合のみ表示、"Layout"カテゴリ）
- **Align**: Box/Column/Row/FlowRow/FlowColumn 内。値は親に依存（Box=9位置, Column/FlowColumn=Start/Center/End, Row/FlowRow=Top/Center/Bottom）
- **Weight**: Row/Column/FlowRow/FlowColumn 内。残空間を比例配分
- **Flex**: FlexBox 内のみ
- 非対応の親に残しても描画時に無視

### Sizing
| Modifier | プロパティ |
|---------|----------|
| Size | `all` / `width` / `height`（dp） |
| SizeIn | `minWidth` / `maxWidth` / `minHeight` / `maxHeight` |
| FillWidth | `fraction`(1.0) |
| FillHeight | `fraction`(1.0) |
| FillSize | `fraction`(1.0) 両方向 |
| Weight | `amount`（親内比例。Weight 1×2 で 50/50）|

### Spacing & Position
- **Padding**: `all` / `horizontal` / `vertical` / `start` / `top` / `end` / `bottom`（細かい指定が広い指定を上書き）
- **Offset**: `x`(0) / `y`(0) dp
- **Align**: `alignment`（親依存・Parent-scoped）

### Visual Style
- **Background**: `color`
- **Border**: `width`(1) / `color`(#000000) / `shape`(Rectangle/Rounded/Circle) / `radius`(Rounded時)
- **Clip**: `shape` / `radius`
- **Shadow**: `elevation`(4) / `shape` / `radius`
- **Alpha**: `value`（0.0〜1.0）
- **Rotate**: `degrees`
- **Scale**: `all` / `x` / `y`（描画ピクセルの拡縮、レイアウトサイズは不変。負で反転）
- **Blur**: `radiusX`(3) / `radiusY`(3) / `uniform`(true)。**Android 12 (API 31)+ 必要**、旧端末は無視

### Animation
- **Marquee**: `iterations`(無制限) / `velocity`(30) / `spacing`(0.33)。溢れた内容を横スクロール（ティッカー）

### Flex（FlexBox 子のみ）
`basis`(Auto/dp/%) / `grow` / `shrink` / `alignSelf`(Auto/Start/Center/End/Stretch/Baseline) / `order`

### Scrolling（"Behavior"カテゴリ、各1個のみ）
- **VerticalScroll**: `enabled`(true)
- **HorizontalScroll**: `enabled`(true)

### applyWhen（条件付き modifier）
全 modifier に任意の `applyWhen` 式。false でその modifier をスキップ（要素自体は表示のまま）。showWhen と同構文（`== != > < >= <=` と `& | !`）。エディタでは各 modifier 先頭の "Apply When" フィールド。

```json
{ "type": "Border", "width": "2", "color": "error", "applyWhen": "%hasError == true" }
```

> ⚠️ ウィンドウのドラッグは modifier ではなく Modifiers タブの **Drag to Move** トグル。

---

## 07. カラー & テーマ（Colors & Theming）

既定で全要素が Material You（デバイステーマ）色。Text=`onSurface`、Button=`primary` 等。ライト/ダーク自動対応。親の色を変えると子は自動調整されないので個別設定が必要。

色を受ける箇所はどこでも: **hex / テーマ色名 / `transparent` / `@{key}` 参照** が使える。

### Hex
- `#RRGGBB`（例 `#FF5722`）/ `#AARRGGBB`（alpha 付き、例 `#80000000`）/ `transparent`

### テーマ色
Material 3 スキームの全ロール: primary/onPrimary/primaryContainer/onPrimaryContainer、secondary系、tertiary系、background/onBackground、surface/onSurface/surfaceVariant/onSurfaceVariant、surfaceTint/surfaceBright/surfaceDim、surfaceContainer(Lowest/Low/High/Highest)、inverseSurface/inverseOnSurface/inversePrimary、error系、outline/outlineVariant、scrim、"fixed"アクセント（primaryFixed 等）。
- 名前は**大小無視**（primary も Primary も可）。未一致は**マゼンタ**にフォールバック（ミスが目立つ）
- WebView 内でも利用可（`--tasker-<role>` CSS変数 / `window.taskerColors` / `Tasker.colors`、`tasker-colors-changed` イベント）→ 05章 WebView

### `@{key}` 参照
色フィールドも provides 参照を受ける（親で1回定義→子孫で再利用）。

---

## 08. Provides システム（Provides System）

親コンテナが key-value を宣言し、子孫が `@{key}` で参照。`%var` が実行時のデータ注入の主役なのに対し、provides はレイアウト自己完結の既定値を与え、共有しやすくする。

### Screen-Level Provides
Screen Properties（未選択時）で設定 → 全体で利用可。**最初に解決**されスコープの基底。テーマ値・スタンドアロン用既定値・設定フラグ（showWhen 用）に。

### Component-Level Provides
コンテナ（Column, Row, FlowRow, FlowColumn, Box, FlexBox, Card, Scaffold）が宣言 → その子孫のみ。同名キーは子が親を**シャドウ**（子孫のみ。兄弟・親には影響なし）。

### Transitive Resolution
provides 値が別の provides キーを参照可（A→B→具体値）。反復パスで解決。**循環参照**は終了保証のみ（ループ検知で停止、片側が未解決 `@{...}` のまま）。循環は避ける。

### Reference Syntax
`@{key}` を任意の文字列プロパティで使用（component プロパティ・modifier プロパティ）。`Hello, @{name}!` のようなインライン置換、複数参照可。未定義キーは `@{...}` のまま残る（未設定が見える）。

### Fallback `@{key ?: fallback}`
キーが無ければ literal の fallback を使用。`@{accentColor ?: primary}`。`?:` 周りの空白はトリム。fallback は literal テキストのみ（参照不可）。両側が非空必須。

### Escaping `@@{key}`
`@@{key}` で literal `@{key}` を描画（解決されない）。ヘルプ画面等で provides 構文自体を見せる時に。

### Scope Resolution Order
親 → 祖先 → … → Screen-Level。最初の一致が勝つ（だから内側がシャドウできる）。エディタは provides スコープ有効時に `@{key}` 補完候補表示。

---

## 09. 変数（Variables）

2つの仕組み: **Variable コンポーネント**（実行時にコンポーネントツリーごと置換）と **文字列変数置換**（text/プロパティ内の `%var` 置換）。

### Variable コンポーネント
唯一の編集可プロパティ `key`（% 付き Tasker 変数名、例 `%content`）。実行時その値を **JSON パース**:
- 単一 component オブジェクト `{ "type": "Text", ... }` → その1要素に
- component オブジェクト配列 `[ {...}, {...} ]` → 兄弟へ展開
- 不正な component JSON → プレーンテキスト（Text）として描画。空/不正キー → 生キーを示す Text

例: HTTP Request で Reddit 投稿取得 → Array Merge で各投稿を component 化 → Variable に渡してスクロールリストを動的生成。

### ID Suffixing
複数兄弟へ展開時、各兄弟サブツリーの全 id に 1-based index を付与（`_1`/`_2`…）。単一兄弟でも `_1`。サブツリー内の id 参照（modifier の source 等）も同 index で追従。内部マーカーで別 Variable 同士は衝突しない。

### 文字列変数置換
- **グローバル変数**（大文字を1つ以上含む、例 `%VOLM`）: **自動更新**（追加アクション不要で最新反映）
- **ローカル変数**（全小文字、例 `%name`）: scene を表示したタスクのスコープ。更新には値を変えてから **Update Scene v2** を実行
- `%` 変数（Tasker、実行時置換）と `@{key}`（レイアウト内解決）は併用可。provides 値に `%` 変数を含めることも可

---

## 10. 条件付き表示（Conditional Visibility）

`showWhen` の boolean 式で表示制御。false で既定は完全除去（showWhenMode で変更可）。空/未設定は常に表示。

### Values & Literals
| 種類 | 構文 | 詳細 |
|-----|------|------|
| 変数参照 | `%name` | `%` 必須、続く先頭は英字/`_`、以降は英数字/`_`/`.`。単独使用時は bool 化（"true"/非0数値=true、それ以外 false、未存在=false）。prefix 無しの bare ワードも変数名として読む |
| 文字列リテラル | `"text"` | ダブルクォート。比較の右辺 `%role == "admin"` |
| 数値リテラル | `123` / `3.5` | 負数可。変数が数値文字列 "800" なら比較時に数値化 |
| boolean | `true` / `false` | クォート無し。bare 変数参照が既に `== true` 相当なので稀 |

### 比較演算子
| 演算子 | 名前 | 詳細 |
|-------|------|------|
| `==` | Equals | 両辺数値なら数値比較、否なら文字列。欠損変数は false |
| `!=` | Not equals | `==` の逆 |
| `>` `<` `>=` `<=` | 数値比較 | 両辺数値必須、否なら false |
| `contains` | 部分一致（大小無視） | |
| `ccontains` | 部分一致（大小区別） | |
| `matches` | パターン一致 | `*`=任意連続、`+`=1文字以上、他は literal。大文字含まなければ大小無視。Tasker の Matches と同じ |
| `matchesr` | 正規表現 | 値内のどこかに一致（全体一致は `^...$`）。大小区別（`(?i)` で無視）。Tasker `~R` 相当 |

### boolean 演算子（優先度: 高→低）
| 演算子 | 名前 | 詳細 |
|-------|------|------|
| `!` | NOT | 直後の式を否定。比較より緩く AND/OR より固い。`!%role == "admin"` = `!(%role == "admin")` |
| `&` | AND | 両辺 true。OR より高優先 |
| `\|` | OR | いずれか true。最低優先 |
| `( )` | グループ化 | 優先度上書き・ネスト可 |

### 優先度
1（最固）比較 → 2 NOT → 3 AND → 4（最緩）OR。例 `%a | %b & %c > 5` = `%a | (%b & (%c > 5))`。迷ったら括弧。

### Gone vs Invisible
`showWhenMode`: **Gone**（既定・除去し詰める）/ **Invisible**（場所確保し非描画）。大小無視、未知値は Gone、`%variable` 参照可（欠損は Gone）。showWhen 設定時のみ有効（エディタの "When Hidden" フィールド）。

### 変数解決
showWhen は**変数のみ**読む（provides キー名は値として比較されないが、`@{key}` は評価前に文字列置換される）。供給源2種: **Tasker %変数**（ローカル/グローバル）と **Screen 変数**（SetVariable / ToggleVariable イベントアクションが書く）。同一名前空間。欠損は false。参照値変化でリアクティブに再評価。

### テスト
オーバーフロー → Variables（フラスコ）→ Test Variables で値シミュレート。エディタ専用、実行時未使用。

---

## 11. イベントハンドラー & アクション（Event Handlers & Actions）

各要素と screen は `eventHandlers` を持つ。ハンドラは複数イベントを listen しアクション列を実行。

```json
"eventHandlers": { "handlers": [ ... ], "stopPropagation": false }
```

各ハンドラ3部: **events**（いずれか発火）/ **condition**（任意 bool、Conditional Visibility と同構文、イベント出力値を key 名で参照可）/ **actions**（順次実行）。

**stopPropagation**（既定 false）: 子で true にすると親ハンドラへ伝播しない。

### イベント種別

**Pointer & Gesture（全要素）:**
| type | 発火 | プロパティ | 出力 |
|------|------|----------|------|
| `click` | タップ | - | - |
| `multi_click` | 連続 N 回タップ | `count`(2) | - |
| `hold` | 長押し | `duration`(500ms) | - |
| `swipe` | スワイプ | - | `direction`(Up/Down/Left/Right), `length`(dp) |

**Input-Value（対応要素のみ）:**
- `checked_changed`(checked) — Switch/Checkbox
- `text_changed`(text) — TextInput
- `slider_value_changed`(value, percent) — Slider
- `range_slider_value_changed`(start, end, start_percent, end_percent) — RangeSlider
- `selection_changed`(selected_index, selected) — SegmentedButtonRow/Dropdown
- `nav_selection_changed`(selected_index) — NavigationBar

**Component-State:**
- `expanded_changed`(expanded) — Dropdown
- `animation_completed` — ProgressBar
- `ready`(lens) / `error`(code, message) — Camera（error は Video も）
- WebView: `page_loaded` / `page_error` / `url_changed` / `console_message` / `progress_changed` / `scroll_changed`
- Video: `loaded`(duration) / `ended` / `buffering`(is_buffering)

**Key（TextInput）:** `key_pressed` — `key`（一致キー名、空=任意）/ `consume`(false)

**Screen-Level（全要素未選択で編集）:**
- `screen_shown` / `screen_hidden`
- `screen_back_pressed` — `cancelEvent`(true で Back を消費し閉じない)
- `screen_variable_changed` — `variableName`（% 無し）→ value

### Event Output Values
イベントが生む typed 値。変数ではなく、①ハンドラの condition で key 名参照、②**OutputToVariable アクション**で screen 変数化、の2箇所で使える。（旧 "Output Variables" 設定の代替）

### Event Actions

**Variable:**
- **SetVariable**: `variable`(% 無し) / `value`(%補間可)
- **ToggleVariable**: `variable`（truthy: true/1/yes/on）
- **OutputToVariable**: `bindings`（出力 key → 変数名 map）

**Component:**
- **RunComponentAction**: `targetId` / `action` / `params`(%補間) / `resultPrefix`(結果を `%{prefix}_{key}`)
- **SimulateInteraction**（"Trigger Event"）: `targetId` / `eventId` / `eventParams` / `outputs`

**Feedback & Screen:**
- **HapticFeedback**: `feedbackType`(Click/Heavy/Double)
- **DismissLayout**（"Dismiss Screen"）

**Tasker:**
- **RunTask**: `task` / `variables`(map, %補間) / `priority`(空=最大)
- **TaskerCommand**: `command`（Tasker Command イベントで受ける）

> Tip: HapticFeedback は上位、DismissLayout は最後（SetVariable/OutputToVariable の後）に置く。

### Drag to Move（Modifiers タブのトグル）
オーバーレイのみ有効。component をドラッグハンドル化しウィンドウ移動。swipe ハンドラがあり hold が無い場合は「hold で開始」挙動に切替（両ジェスチャー併用可）。

### Dismissing Layouts
`DismissLayout` アクションで閉じる（独立した dismiss ジェスチャー/結果設定は無い）。閉じる時、書いた screen 変数（SetVariable/OutputToVariable）が Show Scene v2 の呼出タスクへ返る。TextInput 値を返すには text_changed + OutputToVariable で常時記録。
> ⚠️ 同一イベントで RunTask + DismissLayout を実行すると、起動タスクと呼出タスクは**並行**実行（順序保証なし）。

### Variables Sent to a Task（RunTask 時）
- `%sv2_interaction_type`（single / multi_2 / long_press / swipe / イベント id）
- `%sv2_scene_name` / `%sv2_screen_id` / `%sv2_element_id` / `%sv2_element_value` / `%sv2_element_path`
- swipe 時: `%sv2_swipe_direction` / `%sv2_swipe_length` / `%sv2_swipe_path_x()` / `%sv2_swipe_path_y()`
- 加えて、書いた screen 変数と RunTask の variables map も渡る

---

## 12. Tasker アクション（Tasker Actions）

Scene v2 用 Tasker アクションは**7つ**（Scene v2 カテゴリ）。

### Show Scene v2（主力）
引数: Name/JSON / Screen ID（既存 ID なら in-place 更新、新 ID なら新表示、空=scene 自身の ID）/ Display Mode（空=scene 既定）/ Overlay 引数（オーバーレイ時のみ）/ Auto Dismiss(ms, 0/空=手動まで)/ Physical Display。**待機の有無は display mode が決める**（別トグル無し）。

**Output Variables**（"WithResult" モード時、dismiss まで block）: `%sv2_element_id` / `%sv2_scene_name` / `%sv2_screen_id` / `%sv2_element_path` / `%sv2_interaction_type`。swipe 時 `%sv2_swipe_path_x()` / `_y()` / `_length` / `_direction`（index 1=開始、最終=終了）。加えてレイアウト定義の output 変数も。

### Update Scene v2
ID で要素プロパティ変更（text/色/可視性/modifier 等）。Element ID 入力後に Property/Value 表示。**Local Variable Passthrough**（全タスク変数を scene へ、Limit Passthrough To で限定）は Element ID 無しでも動く。Screen ID 空で全 display に適用。Variable 展開要素はサフィックス ID（`title_2`）で個別ターゲット。

### Update Scene v2 Overlay
オーバーレイ窓設定変更（X/Y・幅高・blocking）。transition duration + easing でアニメ可。Screen ID 必須。

### Dismiss Scene v2
Screen ID の display を閉じる（空=全 display）。

### Run Scene v2 Action
アクティブ要素に named action を命令実行し出力収集。引数: Screen ID（必須）/ Element ID（必須）/ Action Name（必須）/ Names・Values（任意、数一致必須）。返り値は output 変数化。

### Get Scene v2 Values
dismiss を待たず現在の output 変数値を読む。引数: Screen ID（必須）/ Element ID（任意）/ Property（任意）。Element ID 無し → 全 output 変数。Element ID + Property → その live 値を Value 出力変数に（複数値なら配列）。長命オーバーレイの状態ポーリングに。

### Wait For Scene v2 Result
非 WithResult で表示したレイアウトの結果を待つ（show → 別作業 → 必要時 call）。引数: Screen ID（必須）/ Timeout(ms, 0/空=無期限)。成功時 Show Scene v2 と同じ output 変数をセット。

---

## 13. スクリーンプロパティ（Screen Properties）

未選択時に Settings タブが表示。JSON のトップレベルフィールドとして保存。

### General
- `name`（空→ファイル名→"Screen"。アクションから開くとロックされ得る）/ `description`
- `defaultDisplayMode`（Show Scene v2 の Display Mode 空時に使う）: Fullscreen / FullscreenWithResult / Dialog / DialogWithResult / Overlay / OverlayWithResult / AccessibilityOverlay / AccessibilityOverlayWithResult / Dream
- オーバーレイ既定（overlay モード時）: `defaultOverlayX` / `defaultOverlayY` / `defaultOverlayWidth` / `defaultOverlayHeight` / `defaultOverlayBlocking` / `defaultOverlayAnimation`(Fade/Slide/Zoom/Bounce)

### Scaling
- `referenceDp`（設計基準幅 dp、`pr` サフィックス有効化）→ 19章

### Provides
- `provides`（全ツリーへ。スコープ最上位、Tasker からのデータ注入に最適）→ 08章

### Display
- `keepScreenOn`（true で表示中スリープ無効、既定 false）

### System Bars（Fullscreen/Dialog のみ適用）
| プロパティ | 説明 |
|----------|------|
| `statusBarColor` | ステータスバー背景（hex/テーマ名、@{key} 可） |
| `navigationBarColor` | ナビバー背景 |
| `isLightStatusBars` | true でアイコン暗色（明背景時） |
| `isLightNavigationBars` | 同上（ナビバー） |
| `hideSystemBars` | true で両バー非表示（没入モード） |

> オーバーレイは他アプリ上に浮くため system bars を制御不可。

### Other Stored Fields（Settings 非表示・エディタ管理）
`pinnedTestVariableEntries`（旧 `pinnedTestVariables` も読込）/ `resolveVariablesInPreview`(true) / `eventHandlers`（screen 全体）

---

## 14. 表示モード（Display Modes）

各ベースモードに plain と "WithResult" 変種。

- **Fullscreen / FullscreenWithResult** — 単独アクティビティ全画面。ダッシュボード・フルページフォーム
- **Dialog / DialogWithResult** — 背景アプリが見えるフローティング窓。確認・入力・通知
- **Overlay / OverlayWithResult** — 他アプリ上の浮遊窓。最も柔軟（→15章）
- **AccessibilityOverlay / …WithResult** — アクセシビリティサービス経由のオーバーレイ（"Show Over Everything" ホスト。通常オーバーレイが覆えない面に）。アクセシビリティサービス要
- **Dream** — Android スクリーンセーバー（daydream）。ドック/充電+idle 時にシステムが起動・停止（Show Scene v2 では呼べない）

### With Result vs Plain
"WithResult" が**待機を制御**（別トグル無し）。plain = 即継続（永続オーバーレイ向き）/ WithResult = dismiss まで block し結果変数セット。非ブロッキング表示の結果は後で **Wait For Scene v2 Result** で取得。Display Mode 空なら scene 既定が待機可否を決める。

### その他
- **Auto-dismiss**: Show Scene v2 の Auto Dismiss(ms)（0/空=手動まで）
- **Physical Display Targeting**: Physical Display 引数で特定ディスプレイ（未接続 ID は失敗、空=プライマリ）
- **Long-Click Root To Dismiss**: オーバーレイで root 長押し dismiss（実行時に一時ハンドラ追加、scene 定義は不変）
- 非 Dream モードは ID ベース: 既存 Screen ID で in-place 更新、新 ID で新表示（スタック/更新）
- ⚠️ Overlay は "Display over other apps" 権限要。ロック画面/通知トレイ上には "Show Over Everything"（アクセシビリティサービス要）

---

## 15. オーバーレイ機能（Overlay Features）

### Position & Size
- **dp**: 固定。正値=左/上端から、負値=右/下端から（`100`=左から100dp、`-20`=右から20dp）
- **%**: 画面相対。オーバーレイ中心をその % に（`50%`=中央、`-25%`=右から25%）
- X/Y 引数で位置。幅高は明示(dp/%)or 空でコンテンツラップ。空 X/Y は 0
- 各値（X/Y/W/H）は `portrait,landscape` のカンマ区切りペア可（`300,500`）。単値は両方向、片側空は他方継承

### Blocking Overlay
既定はタッチを遮断。Blocking Overlay を無効化で下のアプリへ透過（情報オーバーレイ向き）。
> ⚠️ Android 12+ で非ブロッキングは `block_untrusted_touches = 0` 必要。WRITE_SECURE_SETTINGS 権限があれば Tasker が自動設定。

### Animations（show / dismiss 独立、dismiss は逆再生）
None / SlideFromBottom / SlideFromTop / SlideFromLeft / SlideFromRight / FadeIn / ZoomIn / BounceIn

### Config Transitions（Update Scene v2 Overlay 時）
transition duration(ms) + easing で位置/サイズ変化を滑らかに。easing: Linear / EaseIn / EaseOut / EaseInOut / Overshoot / Bounce

### Draggable Overlays
Modifiers タブの Drag to Move トグル（→11章）

### Show Over Everything
アクセシビリティサービスオーバーレイを使用 → ロック画面/通知トレイ上に表示可。
> ⚠️ Tasker アクセシビリティサービス稼働必須（無いとエラー通知で失敗）。

### Limitations
**TextInput はオーバーレイ非対応**（Android 制限）。要テキスト入力なら fullscreen/dialog。

---

## 16. AI レイアウト生成（AI Layout Generation）

AI タブで自然言語からレイアウト生成・修正。
1. AI タブ → 2. 説明入力（例: "WiFi/Bluetooth/Dark Mode トグルの設定パネル"）→ 3. Generate
- 初回は "Set Up AI"（モデル + API キー設定）が必要
- タブ上部に対象表示: 未選択="Editing: Entire layout"、選択中="Editing: <id>"（"this" が使える）
- 既定は全体再生成 → 選択時は **Component only** トグルで選択要素+子のみ修正
- アンドゥ可

### AI Menu（オーバーフロー）
- Copy System Instructions / Export System Instructions（full schema + ルール。ChatGPT/Claude 等に system prompt として貼り別 AI で生成可）/ Change AI Configuration

### プロンプトのコツ
構造を具体的に / 色は特別な理由が無ければ省略（Material You 既定）/ インタラクションを記述 / 反復改善。

---

## 17. Raw JSON 編集（Raw JSON Editing / Code タブ）

Code タブ。ビジュアル ↔ JSON 双方向同期。用途: コピペ共有・一括編集（色の find-and-replace 等）・デバッグ。

### Validity Indicator
**VALID（緑）** / **INVALID（赤、構文エラー。ビジュアル更新停止）**

### JSON 構造（`root` のみ必須、他は任意で既定あり）
```json
{
  "name": "My Screen",
  "description": "...",
  "provides": { "key": "value" },
  "statusBarColor": "surface",
  "navigationBarColor": "surface",
  "isLightStatusBars": "true",
  "isLightNavigationBars": "true",
  "hideSystemBars": false,
  "keepScreenOn": false,
  "referenceDp": "360",
  "root": { "type": "Column", "id": "root_col", "children": [ ... ] }
}
```
- 他フィールド（オーバーレイ/エディタ状態）: `defaultDisplayMode` / `defaultOverlay*` / `resolveVariablesInPreview` / `pinnedTestVariableEntries` / screen `eventHandlers`（エディタが管理）
- 各 component は `type` + `id`。コンテナは `children`（Scaffold/app bar は named slots）。任意 `modifiers`。色は hex / テーマ名 / `transparent`
- 不正にすると同期停止（修正で復帰）

---

## 18. Tips & パターン（Tips & Patterns）

### 共通パターン
- **App screen with Scaffold** — フル画面の推奨起点。topBar / bottomBar / floatingActionButton / content（必須）スロット（各 1要素配列）。content 自動でバー回避
- **Scaffold with bottom bar** — BottomAppBar の content は Row。各 IconButton に Weight で均等配分
- **Card** — `style`(Filled/Elevated/Outlined)。手動カードは Column + Clip(Rounded) + Background + Padding
- **Settings panel** — ラベル行 + Switch。各 Switch に `checked_changed` ハンドラ + SetVariable（`%checked`）
- **Grid layout** — Row/Column の直接の子に Weight で等幅
- **Centered content** — Box + `contentAlignment: "Center"` + FillSize
- **Data-driven list** — Variable component の key に Tasker 変数（component JSON 保持）
- **Conditional content** — showWhen で状態別 UI（Test Variables で確認）
- **Image background with text** — Box で Image + scrim(Spacer + 半透明 Background) + Text 重ね
- **Compact overlay** — 小型浮遊パネル。閉じるは IconButton + click → DismissLayout
  > ⚠️ **旧 Clickable modifier は廃止**。タップで閉じるは event handler（click → DismissLayout）で行う
- **Table layout** — Column + 複数 Row、各セル Weight で等分

### Best Practices
- フル画面は Scaffold から / 色は基本省略（Material You）/ クリック可アイコンは IconButton（48dp + ripple）/ インタラクションは eventHandlers(click + アクション) / 固定サイズより Weight / ツリーは浅く（6+ 段は見直し）/ 参照する要素には意味ある ID / 柔軟な隙間は Spacer + Weight / 共有値は provides + `@{key}` / 状態別 UI は showWhen

---

## 19. 比例スケーリング（Proportional Scaling）

1つの画面サイズで設計し、他サイズでも比率を一定に保つ。

### 仕組み
1. **基準幅設定** — Screen Properties > Scaling で設計幅（dp）。JSON の `referenceDp`（例 360＝一般的なスマホ）
2. **`pr` サフィックス** — 寸法値に付加（`16pr` = 基準で 16dp、他で比例スケール）

実行時、`pr` 値に `(currentScreenWidth / referenceWidth)`（共に dp）を乗算。基準デバイスで係数 1。幅2倍で 32。大小無視（`16pr`=`16PR`）。基準未設定/解析不可/0/負なら係数 1（スケールなし）→ `pr` を先に付けても安全。

### 対応プロパティ（framework が画面依存寸法と解決するもののみ）
- Text: `textSize`
- TextInput: `textSize` / `shape`(角丸)
- Padding: `all` / `horizontal` / `vertical` / `start` / `top` / `end` / `bottom`
- Size: `all` / `width` / `height`
- SizeIn: `minWidth` / `maxWidth` / `minHeight` / `maxHeight`
- Offset: `x` / `y`
- Border: `width` / Shadow: `elevation` / Blur: `radiusX` / `radiusY`
- Divider: `thickness` / Spacer: `width` / `height` / ProgressBar: `strokeWidth`
- FlowRow / FlowColumn: `spacingHorizontal` / `spacingVertical`

非寸法（alpha / weight / fill fraction / rotation / scale / color）は無影響（サフィックスは剥がされるだけ）。

### 基準幅の目安
360（標準スマホ・最多）/ 400（大型スマホ）/ 600（小型タブ）/ 840（大型タブ）。`pr` と通常値は同一レイアウトで混在可。
</content>
</invoke>
