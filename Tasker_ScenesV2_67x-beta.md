# Tasker 6.7.x-beta: Scenes V2 仕様メモ

> **注意**: これはベータ版の仕様です。正式版リリース時に変更される可能性があります。
> 正式版では差分情報のみ公開される可能性があるため、ベータ版の全仕様をここに記録しています。

- **ソース**: Reddit r/tasker 投稿（[DEV] Tasker 6.7.0-beta / 6.7.1-beta Update 1 / 6.7.3-beta Update 2 / 6.7.4-beta Update 3 / 6.7.5-beta WebView・Video）
- **初版記録日**: 2026-03-13（6.7.0-beta）
- **更新日**: 2026-06-16（6.7.5-beta: WebView / Video コンポーネント追加）
- **デモ動画（6.7.0）**: [youtu.be/SU0pG36GkUo](https://youtu.be/SU0pG36GkUo)
- **デモ動画（6.7.1）**: [youtu.be/pffxU1p2wlt4](https://youtu.be/pffxU1p2wlt4)
- **デモ動画（6.7.3）**: [youtu.be/3mId6ekwGuI](https://youtu.be/3mId6ekwGuI)
- **デモ動画（6.7.4）**: [youtu.be/A-Nk5VyH3RM](https://youtu.be/A-Nk5VyH3RM)
- **デモ動画（6.7.5 WebView）**: [youtu.be/jGdw6mZH9B8](https://youtu.be/jGdw6mZH9B8) / **（6.7.5 Video）**: [youtu.be/-xIHjZrsUHo](https://youtu.be/-xIHjZrsUHo)

---

## Scenes V1 からの改善点

| V1 の問題 | V2 での改善 |
|-----------|------------|
| 画面サイズ固定（機種によって崩れる） | レスポンシブレイアウト |
| 古いコントロール | Material 3 対応 |
| 位置指定が不安定 | 宣言的レイアウト |
| 非オーバーレイの同時表示不可 | 複数シーン同時表示可能 |
| Undo ボタンなし | Undo/Redo（50スナップショット）|

---

## Scenes V2 主要機能ハイライト

- **宣言的・レスポンシブレイアウト** — ピクセルグリッド不要、画面サイズに自動適応
- **Material 3 フルテーマ** — ダーク/ライトモード自動切替
- **4つの表示モード**: fullscreen / dialog / overlay / accessibility overlay
- **Scaffold** — TopAppBar・BottomAppBar・FAB を持つアプリライクなUI
- **リアクティブ変数バインディング** — タスクから渡すローカル変数 + グローバル変数のリアルタイム更新
- **インタラクション** — タップ/スワイプ/長押し/複数クリック、ハプティックフィードバック付き
- **条件付き表示（showWhen）** — 変数式での表示/非表示切替（タスク不要）
- **オーバーレイシステム** — ドラッグ可能・アニメーション・パススルー・スタック可能・自動消去
- **AI生成** — 自然言語でレイアウト生成、フォローアッププロンプトで反復改善
- **インラインシーン** — Show Scene V2 アクション内に直接シーンを構築（別シーン定義不要）
- **フルビジュアルエディター** — ライブプレビュー・Undo/Redo・ドラッグ&ドロップツリー・JSONエディター

---

## コンポーネント一覧

> 基本コンポーネントのみ。エンジンの安定後に追加予定。

### レイアウト
| コンポーネント | 説明 |
|--------------|------|
| Column | 縦方向フレキシブルコンテナ |
| Row | 横方向フレキシブルコンテナ |
| Box | スタック型コンテナ |
| Spacer | 固定ギャップ |
| Scaffold | 4スロット構成のアプリ構造（topBar/bottomBar/FAB/content） |
| FlexBox | 縦横を1つで制御できる動的コンテナ（6.7.4 新規） |

### UI 要素
| コンポーネント | 説明 |
|--------------|------|
| TopAppBar | Material 3 トップバー |
| BottomAppBar | Material 3 ボトムバー |
| FloatingActionButton | FAB |
| Text | テキスト（サイズ/色/配置/縦配置設定可能） |
| Image | URL / Material Icon / SVG 対応、contentScale・tint 設定可能、アニメーション GIF 対応（6.7.1） |
| Divider | 区切り線（色・太さ設定可能） |
| Button | Material 3 スタイル、dismiss + アクション対応 |
| Switch | トグル、状態トラッキング |
| TextInput | ヒント文字・オートフォーカス・リアルタイム値送信、オーバーレイ内で動作可（6.7.1）、label プロパティ追加（6.7.1） |
| NavigationBar | ボトムナビゲーション |
| NavigationItem | ナビゲーションアイテム |
| Card | カードコンテナ（6.7.1 新規） |
| Checkbox | チェックボックス、Toggle アクション対応（6.7.1 新規） |
| IconButton | アイコンボタン、URL・SVG・base64 画像も使用可能（6.7.1 新規、6.7.3 で画像ソース拡張） |
| Segmented Button Row | ラジオボタン的セレクター、複数選択オプションあり（6.7.3 新規） |
| Segmented Button Item | Segmented Button Row の子要素（6.7.3 新規） |
| Flow Row | 内容に応じて自動折り返しする横方向コンテナ（6.7.3 新規） |
| Flow Column | 内容に応じて自動折り返しする縦方向コンテナ（6.7.3 新規） |
| Slider | スライダー（6.7.4 新規） |
| Range Slider | 範囲指定スライダー（6.7.4 新規） |
| Progress Bar | プログレスバー、アニメーション付きオプションあり（6.7.4 新規） |
| Camera | カメラプレビュー、複数カメラの同時表示対応（6.7.4 新規） |
| WebView | HTML/JS/CSS を埋め込み表示するWebブラウザ（v1同様、オンライン/オフライン両対応）。シーン全体をWebスクリーン化も可能（6.7.5 新規） |
| Video | 動画プレーヤー。再生制御アクション + playing / position / formattedPosition の states・events で監視可（6.7.5 新規） |

### 特殊コンポーネント
| コンポーネント | 説明 |
|--------------|------|
| Variable | `%var` の JSON をランタイムでコンポーネントツリーとしてレンダリング（動的UIリスト等） |
| Placeholder | ランタイム注入スロット（Update Scene V2 アクションで更新） |
| WebView | HTML/JS/CSS を埋め込み表示（6.7.5 実装）。JS から `Tasker.*` ブリッジでアクション呼出可（6.7.6）。→ 下部「WebView コンポーネント」節 |
| Video | 動画プレーヤー（6.7.5 実装）。playing / position / formattedPosition の states、再生制御アクション・イベント対応 |
| ArraysMergeTemplate | Tasker 配列（`%icon`/`%item` 等）を行リストへ動的展開（6.7.6） |

### スロット概念
各コンポーネントにスロットがあり、他のコンテンツを配置可能。

---

## モディファイア一覧

| モディファイア | 説明 |
|--------------|------|
| Padding | all / horiz / vert / 個別辺 |
| Size / SizeIn | 固定サイズ または min/max 制約 |
| FillSize / FillWidth / FillHeight | 分率ベース（0〜1） |
| Weight | Row/Column 内の比例サイズ（空きスペースを埋める） |
| Offset | X/Y 位置シフト |
| Align | Box 内9ポジション配置 |
| Background | hex色 または Material 3 テーマカラー名 |
| Border | 幅・色・形状（rect / rounded / circle） |
| Clip | 形状クリッピング（半径設定可能） |
| Shadow | 標高ベースのシャドウ |
| Alpha | 透明度（0〜1） |
| Rotate | 回転（度数） |
| Interactable | タップ/長押し/スワイプ/複数クリック + リップル + ハプティック |
| VerticalScroll / HorizontalScroll | スクロール |
| WindowDrag | オーバーレイドラッグのアンカー指定 |
| Blur | ぼかし効果（6.7.1 新規） |
| Marquee | テキスト等のコンポーネントを自動スクロール表示（iterations / velocity / spacing 設定可）（6.7.3 新規） |
| Scale | コンポーネントを自由にリサイズ（レイアウトフローに影響しない）（6.7.4 新規） |
| Apply When | モディファイアに条件を付与し、特定状況でのみ適用（全モディファイア対応）（6.7.4 新規） |

---

## 表示モード（7種類）※6.7.3 で拡張

> **6.7.3 破壊的変更**: 既存の Show Scene V2 アクションはすべて「結果を待たない」モードに変更される。結果を受け取りたい場合は **With Result** バリアントに手動で変更が必要。

| モード | 説明 |
|--------|------|
| Fullscreen | エッジtoエッジ、ステータスバー/ナビバー制御、イマーシブモード |
| Fullscreen With Result | Fullscreen + 結果を待つ |
| Dialog | 中央ウィンドウ、ブラーバックグラウンド、コンテンツに自動サイズ |
| Dialog With Result | Dialog + 結果を待つ |
| Overlay | フローティングウィンドウ、任意位置、ドラッグ可能、パススルー |
| Overlay With Result | Overlay + 結果を待つ |
| Accessibility Overlay | 通知パネル・システム設定画面等のシステム画面上にも表示可能 |
| Dream | Android スクリーンセーバーとして表示（6.7.3 新規） |

---

## Tasker アクション（Scene V2 カテゴリ）

| アクション | 説明 |
|-----------|------|
| Show Scene V2 | fullscreen / dialog / overlay + インラインシーン対応 |
| Dismiss Scene V2 | シーンを閉じる |
| Update Scene V2 | 変数値プッシュ または 要素直接更新 |
| Update Scene V2 Overlay | ランタイムで位置変更・アニメーション・リサイズ |
| Get Scene V2 Element Value | 要素の現在状態を読み取り |
| Wait For Scene V2 Result | シーンが閉じるまでタスクをブロック |
| Run Scene V2 Action | コンポーネントのアクションを実行（6.7.1 新規） |
| Trigger Event | 他コンポーネントのイベント（クリック・長押し等）をプログラムからトリガー（6.7.4 新規） |

**Tasker イベント（6.7.1 追加）**: Scene V2 Event — シーン内コンポーネントの変化で Tasker タスクをトリガー

### Screen Events（6.7.3 新規）

シーン自体にイベントハンドラを設定可能:

| イベント | 説明 |
| ------- | ---- |
| Back Pressed | デバイスの戻るボタン押下（キャンセル可能） |
| Screen Shown | シーン初回表示時 |
| Screen Hidden | シーン非表示時 |
| Variable Changed | シーン内の任意変数が変化したとき |

---

## インタラクション詳細

- タップ / ダブルタップ / トリプルタップ / N クリック（長押し時間はカスタマイズ可）
- 長押し
- スワイプ（8方向、`%sv2_swipe_length` / `%sv2_swipe_direction` で詳細取得）
- ハプティックフィードバック（light click / heavy click / double tap）
- クリックイベントの伝播防止オプション

### Event Actions（6.7.3 刷新）

1イベントに複数の順序付きアクションを設定可能:

| アクション | 説明 |
| ---------- | ---- |
| Set Variable | 変数に値をセット（`%var` 展開対応） |
| Output to Variable | イベントが生成したデータ（テキスト入力値・スイッチ状態等）を変数に保存 |
| Toggle Variable | boolean 変数を true/false 反転 |
| Dismiss Layout | シーンを閉じ、出力変数を呼び出し元に返す |
| Haptic Feedback | デバイスを振動（light / heavy / double tap） |
| Run Component Action | 別コンポーネントのアクションを実行（スイッチ切替等）、パラメータ渡し可 |

---

## 条件付き表示（showWhen）

- 任意コンポーネントに変数式で表示/非表示を設定
- 比較演算子: `==` `!=` `>` `<` `>=` `<=`
- 文字列演算子: `contains`（部分一致）、`ccontains`（大文字小文字区別の部分一致）（6.7.4 新規）
- ブール演算子: `&`（AND） `|`（OR） `!`（NOT）
- タスク不要 — 完全に宣言的

---

## 変数バインディング

- JSON フィールド内に `%var` を記述可能（テキスト/色/表示条件等）
- グローバル変数はリアルタイムでシーンを自動更新（例: `%WIN`, `%BLUE`, `%BATT`）
- Update Scene V2 アクション + 変数パススルーでライブ更新
- **縦横向きで別サイズ/位置を指定可能**（6.7.4 新規）— シーン・オーバーレイの portrait/landscape で異なる設定を適用
- 環境変数（9種類）:
  - `%sv2_display_width`
  - `%sv2_render_is_portrait`
  - `%sv2_display_is_landscape`（旧 `%sv2_display_is_widescreen` から改名 ※6.7.4）
  - 他 6種類
- **タップ座標変数**（6.7.5 新規）— タップ系イベント（short/long/multi-tap）でタスク呼び出し時に渡される:
  - `%sv2_tap_x` / `%sv2_tap_y`
- **オーバーレイ拡張**（6.7.5 新規）— 画面外への描画、ドラッグで画面外に出たら自動で画面内に戻すオプション。各プロパティに (i) 説明ボタン追加

---

## テーマ

- Material 3 フルカラーシステム
- テーマカラー名（primary / surface / onBackground 等）
- ダーク/ライトモード自動切替
- **Provides** — シーンレベルまたはコンテナコンポーネントレベルでカスタムカラー定義、子コンポーネントで使用可能

---

## アニメーション

### プリセット（10種）
- SlideFromBottom / Top / Left / Right
- FadeIn / FadeOut
- ZoomIn / ZoomOut
- BounceIn
- Slide+Fade コンボ
- 閉じる時に自動で逆アニメーション

### イージングカーブ（6種）
- Linear / EaseIn / EaseOut / EaseInOut
- Overshoot / Bounce

---

## オーバーレイ機能

- **位置指定**: dp / パーセント / dpFromEnd
- ランタイム再配置（アニメーション・継続時間・イージング指定可能）
- **パススルー** — 背後のアプリにタッチを通過
- **WindowDrag モディファイア**でドラッグ可能
- 複数オーバーレイ同時表示（同IDで上書き更新）

---

## 画面プロパティ

- ステータスバー色 + アイコン明暗
- ナビゲーションバー色 + ボタン明暗
- システムバー非表示（イマーシブモード）
- 画面名（履歴画面に表示）

---

## エディター機能

| タブ | 内容 |
|------|------|
| Tree | コンポーネントツリー（ドラッグ&ドロップ、複数選択、コピー/カット/ペースト等）※旧 Elements |
| Properties | 選択コンポーネントのプロパティ設定 |
| Modifiers | モディファイア設定（6.7.1 で Properties から分離） |
| States | コンポーネントの状態管理・変数バインディング（6.7.1 新規） |
| Events | イベントハンドラ設定（6.7.1 で Interaction タブを置き換え） |
| Actions | コンポーネントへのアクション設定（6.7.1 新規） |
| JSON | JSON直接編集（旧 Raw JSON）、cut/copy/paste ボタン追加（6.7.3） |
| AI | 自然言語でのレイアウト生成 |
| Transitions | コンポーネントの表示/非表示時アニメーション設定（6.7.3 新規） |

> **6.7.0 との違い**: 5タブ → 8タブに拡張（6.7.1）→ Transitions タブ追加で9タブに（6.7.3）→ タブ数変更なし（6.7.4）

### その他エディター機能

- **Interactive Mode**（6.7.3 新規）— スイッチ切替・変数設定・ShowWhen 確認をエディタ内で直接テスト
- ライブプレビュー（プレビュー内クリックでツリー選択）
- Undo/Redo（50スナップショット）
- 複数選択・コピー/カット/ペースト・インデント/アウトデント・名前変更・タイプ変更・複製
- **マルチモニター対応** — 最大4ディスプレイにエディターパネルを分散
- カラーピッカー（HSV + アルファ + hex + M3テーマカラー）
- アイコンピッカー（Material 3 アイコン検索、プレビュー内タップで選択切替）
- showWhen プレビュー用テスト変数パネル（オン/オフ切替可、6.7.3）
- タグ検索（例: "Padding" で "Spacing" を検索）
- 全コンポーネント・モディファイアにツールチップ
- インラインシーン構築（Show Scene V2 アクション内で直接）
- 作成時レイアウトタイプ選択（Scaffold または Column）
- Scaffold のスロットデフォルトエディター
- スコープ対応プロパティ（ColumnScope / RowScope 等）
- **エディタからタスク作成/編集**（6.7.3 新規）— 編集後は同プロジェクト内に保存
- Size Modifier に **Uniform** オプション追加（幅・高さを同時編集、6.7.3）
- ファイルシステムから画像ファイルを直接選択可能（6.7.3）
- コンポーネント削除ボタンをドラッグバーに追加（6.7.3）
- 特定コンポーネントは専用の親コンポーネント内にのみ追加可能（Segmented Button Item 等）

---

## AI 生成機能

- 自然言語 → レイアウト生成
- フォローアッププロンプトで反復改善（既存内容を把握してる）
- カスタム AI セットアップ用システムプロンプトのエクスポート

---

## マルチディスプレイ対応

- Show Scene V2 で特定の displayId をターゲット指定可能
- 外部モニター対応
- エディターのマルチモニタープレビュー

---

## その他追加アクション

- **Wait For Command** アクション（Scenes V2 とは別の追加機能）

---

## 未解決・検討中

- V1 から V2 に移行できていない機能がある可能性（ユーザーからのフィードバック収集中）
- コンポーネント・モディファイアは今後追加予定（エンジン安定後）

---

## WebView コンポーネント（6.7.5 実装済み・6.7.6 で JS ブリッジ拡張）

長らく要望されていた WebView が **6.7.5-beta で実装**され、HTML/JavaScript/CSS によるカスタム UI を Scene V2 内で表示できるようになった（Scenes V1 の WebView と同様、オンライン/オフライン両対応）。Video コンポーネントも同時追加。

- 履歴: 6.7.0-beta 時点では「エンジン安定を優先」で未実装だったが、6.7.5-beta で追加。dev 見解では **AutoTools Webscreens ができることは概ね全部代替可能**
- コンテンツ指定: `"content"` にインライン HTML 文字列、またはファイルパス（例 `/storage/emulated/0/Tasker/files/html/xxx.html`）

### WebView → Tasker JavaScript ブリッジ（6.7.6）

WebView 内 JS から Tasker アクションを **`Tasker.*`** で直接呼べる。全アクションが Promise を返し `await` 対応。

- タスク: `var started = Tasker.runTask('My Task', { foo: 'bar', '%num': '3' }, 10);`（第3引数=priority）
- 変数: `Tasker.setVariable("name", value);`（% なし）
- 出力取得: `const out = await Tasker.runShell({ cmd: '...' }); out.stdout`（出力変数名をキーに持つ）
- `await Tasker.flash({text})` / `Tasker.vibrate({time})` 等。失敗は JS 例外 → try/catch 可
- HTML 要素のドラッグハンドル化: CSS クラス `tasker-drag-handle` / `-x` / `-y`
- Material 3 パレットが CSS 変数（`--tasker-surface` 等）で渡る
- **呼び出し例・既知バグ（変数伝播など）の詳細は `Tasker_DevNotes_67x-beta.md` A項を参照**
- 公式: `https://tasker.joaoapps.com/userguide/en/scenes_v2/webview-actions.html`

> 従来 SceneV2 側の `RunTask` イベント（Button 等）とは別系統。WebView 内から任意タイミングで能動的にアクションを叩けるのが強み。

---

## 実装Tips（2026-03-20 実機確認）

### インラインシーン vs 名前付きシーン

- Show Scene V2 に JSON を直接記述する「インラインシーン」は文字数制限（約5400文字）がある
- 複雑なシーンは Tasker のシーンタブで名前付き Scene V2 を作成し、Show Scene V2 にはシーン名のみ指定する

### タスク間の変数スコープ

```text
TaskA（シーン表示） → SceneA（ローカル変数を参照可）→ TaskB（TaskA のローカル変数は不可）
```

- TaskA → SceneA: ローカル変数がそのまま参照できる（`%screen_id` 等）
- SceneA → TaskB: Clickable の `variables` フィールドで明示的に渡す必要がある

```json
"variables": {"val": "値", "screen_id": "%screen_id"}
```

### ボタン色の動的切替

- showWhen ペア（_off/_on の2要素）はシーン表示時のみ評価される（ローカル変数では動的更新不可）
- **正解**: 単一ボタン + Update Scene V2 (481) で `buttonColor` プロパティを変更

```json
Update Scene V2: [screen_id, element_id, "buttonColor", "primary"]
```

### Get Scene V2 Element Value (483) の制限と回避策

- テキスト内容のみ取得可能（`buttonColor` 等のプロパティは取得不可）
- **回避策（6.7.0）**: 隠し Text 要素（`"showWhen": "false"`）を用意し、Update Scene V2 でテキストを書き込んで状態を保持する

```json
{"type": "Text", "id": "val_busy", "text": "", "showWhen": "false"}
```

- **回避策（6.7.1 以降）**: States タブでコンポーネントの状態を変数にバインドし、その変数を Tasker 側で参照する方法が推奨（隠し Text 要素が不要になる場合がある）
