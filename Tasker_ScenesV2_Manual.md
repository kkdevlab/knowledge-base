# Tasker Scenes V2 公式マニュアル（ベータ版）

> **ソース**: https://tasker.joaoapps.com/userguide/en/scenes_v2.html
> **記録日**: 2026-03-13（Tasker 6.7.0-beta 時点）
> **関連**: `Tasker_ScenesV2_Beta670.md`（Reddit投稿の全機能リスト）

---

## 01. 概要

**Tasker Screen Builder** — ビジュアルなカスタムスクリーン設計ツール。コード記述不要でフォーム、ダッシュボード、コントロールパネル、ステータス表示などを作成できる。

**技術構成:**
- レイアウトはコンポーネント階層で構築
- コンテナ: Column、Row、Box
- コンテンツ: Text、Button、Image
- スタイリング: Modifiers
- 基盤: JSON形式（ユーザーは直接編集不要、ビジュアルエディタと双方向同期）

**カバー内容:**
- エディタパネルの操作方法
- レイアウト基礎知識
- テーマカラーによるスタイリング
- Taskerタスクとの連携
- 表示モード（全画面、浮動ダイアログ、オーバーレイ）
- AI生成や条件付き表示などの高度な機能

---

## 02. クイックスタート（5ステップ）

**Step 1: レイアウト作成**
- Scenes タブから Scene V2 を選択し、レイアウトタイプを決定
  - **Simple Layout**: 学習向けのシンプル設計用
  - **Full Page Layout**: トップバー・ボトムバー・FABを含むアプリ風画面
- Simple Layout を選ぶと、テキストコンポーネント付きの Column が表示される

**Step 2: テキスト編集**
- プレビュー内の Text を直接タップして選択、再度タップしてプロパティを開く
- 内容を `"Hello, World!"` に変更

**Step 3: ボタン追加**
- Elements tree の「+」ボタンで「Add After」を選択
- Button をリストから選択

**Step 4: ボタン機能設定**
- ボタンのラベルを `"Close"` に変更
- Events タブで `"Tap"` を選択してクリック時にレイアウトが閉じるよう設定（※6.7.0 では Interaction タブ）

**Step 5: プレビューと利用**
- エディタとプレビュー間のセパレータをドラッグして拡大表示
- 「Show Scene V2」アクションで表示

> フル履歴の Undo 機能により、自由に実験可能

---

## 03. エディタ構成

### タブ

| タブ | 機能 |
|------|------|
| **Elements** | コンポーネントツリー（ファイルエクスプローラーのようにネスト表示）。「+」でレイアウト・表示・入力カテゴリから追加。複数選択・ドラッグ操作対応 |
| **Properties** | 選択コンポーネントの編集。ID・showWhen式・コンポーネント固有プロパティ・スタイルレイヤー（パディング、背景等）の設定 |
| **Interaction** | ジェスチャーハンドラ（シングルタップ、ロングプレス等）の設定。アクション実行・レイアウト終了時の値返却を構成 |
| **Raw JSON** | 基盤JSONを直接編集（ビジュアルエディタと双方向同期） |
| **AI** | 自然言語でのレイアウト生成・修正。システム命令のエクスポートも可能 |

### 補助機能
- **Test Variables**: showWhen条件の動作確認用に一時的な変数値を設定するパネル（フラスコアイコンから開く）
- **編集モード**: Screen mode（フルスクリーン）と Element mode（スタンドアロンコンポーネント）の2種類

---

## 04. レイアウト基礎（Column / Row / Box）

### 3つの基本コンテナ

| コンテナ | 説明 | 適用例 |
|---------|------|--------|
| **Column** | 子要素を縦に並べる（上→下）。ほとんどのスクリーンのルートに使用 | リスト、フォーム |
| **Row** | 子要素を横に並べる（左→右） | ツールバー、ボタンバー |
| **Box** | 子要素を重ねる（最後の子が最上部） | オーバーレイ、バッジ、画像+テキストの重ね合わせ |

### アライメント・配置制御

**Column:**
- 水平アライメント: Start / Center / End
- 垂直配置: Top / Center / Bottom / SpaceBetween / SpaceAround / SpaceEvenly

**Row:**
- 水平配置: Start / Center / End / SpaceBetween / SpaceAround / SpaceEvenly
- 垂直アライメント: Top / Center / Bottom

**Box:**
- 9ポジション配置オプション

### ネストの活用例
- Column + Rows → グリッドレイアウト
- Column + Row ヘッダー + Column ボディ → アプリケーションレイアウト
- Box + Image + Text → オーバーレイ付きカード

---

## 05. コンポーネント

### 共通プロパティ
全コンポーネントが持つ: `id`、`showWhen`、`modifiers`、`dismissEntries`

### ID 管理
- 自動生成されるIDは、ツリー操作・インタラクション設定・Taskerアクションでの動的更新に使用
- ID 名変更時、エディタは自動的に他コンポーネント内の参照を更新

### コンポーネント分類

**レイアウト（Layout）**
| コンポーネント | 説明 |
|--------------|------|
| Column | 縦配置コンテナ |
| Row | 横配置コンテナ |
| Box | 重層コンテナ |
| Scaffold | フルページ構造（topBar / bottomBar / FAB / content の4スロット） |

**表示（Display）**
| コンポーネント | 説明 |
|--------------|------|
| Text | テキスト表示（サイズ・色・配置・縦配置設定可能） |
| Image | URL / Material Icon / SVG 対応、contentScale・tint 設定可能 |
| Divider | 区切り線（色・太さ設定可能） |

**入力（Input）**
| コンポーネント | 説明 |
|--------------|------|
| Button | Material 3 スタイル、dismiss + アクション対応 |
| FloatingActionButton | FAB |
| IconButton | アイコン付きボタン（タッチターゲットあり） |
| Switch | トグル、状態トラッキング |
| TextInput | ヒント文字・オートフォーカス・リアルタイム値送信。**オーバーレイモードでは動作しない** |

**特殊コンポーネント**
| コンポーネント | 説明 |
|--------------|------|
| Variable | `%var` の JSON をランタイムでコンポーネントツリーとしてレンダリング。複数展開時は ID に `_1`、`_2` などの接尾辞が自動付加 |
| Placeholder | ランタイム注入スロット（Update Scene V2 アクションで更新） |
| NavigationBar | ボトムナビゲーション |
| NavigationItem | ナビゲーションアイテム |

> コンポーネントは「スロット」の概念を持つ。各コンポーネントに他のコンテンツを配置できるスロットがある。

---

## 06. モディファイア

モディファイアはコンポーネントに適用する視覚スタイル。サイズ・スペーシング・色・枠線などを制御し、**積み重なった順序**が最終的な見た目を決める。

### スタッキングの原則
リストの上（内側）のモディファイアがコンテンツに最も近い。
例: パディング→背景 と 背景→パディング では背景色が適用される領域が異なる。

### 親スコープモディファイア
**Align** と **Weight** は、親コンテナがサポートする場合のみ利用可能。

### モディファイア一覧

**サイジング**
| モディファイア | 説明 |
|--------------|------|
| Size | 固定サイズ（幅・高さ） |
| SizeIn | min/max 制約 |
| FillWidth | 分率ベースの幅（0〜1） |
| FillHeight | 分率ベースの高さ（0〜1） |
| FillSize | 分率ベースの幅・高さ両方 |
| Weight | Row/Column 内での比例サイズ（空きスペースを埋める） |

**スペーシング・位置**
| モディファイア | 説明 |
|--------------|------|
| Padding | all / horiz / vert / 個別辺 |
| Offset | X/Y 位置シフト |
| Align | 親タイプに応じた位置調整（Box 内9ポジション等） |

**視覚スタイル**
| モディファイア | 説明 |
|--------------|------|
| Background | hex色 または Material 3 テーマカラー名 |
| Border | 幅・色・形状（rect / rounded / circle） |
| Clip | 形状クリッピング（半径設定可能） |
| Shadow | 標高ベースのシャドウ |
| Alpha | 透明度（0〜1） |
| Rotate | 回転（度数） |

**スクロール**
| モディファイア | 説明 |
|--------------|------|
| VerticalScroll | 縦スクロール |
| HorizontalScroll | 横スクロール |

**インタラクション**
| モディファイア | 説明 |
|--------------|------|
| Interactable | タップ/長押し/スワイプ/複数クリック + リップル + ハプティック |
| WindowDrag | オーバーレイドラッグのアンカー指定（オーバーレイ専用） |

---

## 07. カラーとテーミング

### デフォルト動作
デフォルトですべてのコンポーネントが **Material You** カラーを使用。Text は `onSurface`、Button は `primary` など設定なしでネイティブな見た目が実現。ライト/ダークモードに自動対応。

> 色をオーバーライドする場合、子要素の色も手動で調整する必要がある（背景を変えると子テキストが見えなくなる可能性があるため）。

### サポートされるカラー形式

**Hex カラー:**
- `#RRGGBB`（例: `#FF5722`）
- `#AARRGGBB`（透明度付き、例: `#80000000` = 50%透過黒）

**テーマカラー名:**

| カテゴリ | 色名 |
|---------|------|
| プライマリ | `primary`、`onPrimary`、`primaryContainer`、`onPrimaryContainer` |
| セカンダリ | `secondary`、`onSecondary`、`secondaryContainer`、`onSecondaryContainer` |
| ターシャリ | `tertiary`、`onTertiary`、`tertiaryContainer`、`onTertiaryContainer` |
| サーフェス | `surface`、`onSurface`、`surfaceVariant`、`onSurfaceVariant` |
| エラー | `error`、`onError`、`errorContainer`、`onErrorContainer` |
| 背景 | `background`、`onBackground` |
| 反転 | `inverseSurface`、`inverseOnSurface` |
| その他 | `outline`、`outlineVariant`、`scrim` |

### 使い分けの推奨
- **テーマカラー**: 背景やテキストなどほとんどのUI要素に推奨
- **Hex カラー**: ブランドカラー・装飾的アクセント・正確な色指定が必要な場面

---

## 08. Provides システム

コンポーネントツリーに名前付き値を注入する機能。親コンテナがキーと値のペアを宣言し、子孫は `@{key}` 構文で参照できる。

### スクリーンレベルの Provides
- レイアウト全体で利用可能
- 優先度が最も高い（コンポーネントレベルより先に解決）
- 用途: テーマカラーやフォントサイズなどの統一値 / Tasker変数に依存しないデフォルト値 / showWhen で動作制御するフラグ

### コンポーネントレベルの Provides
- Column や Row などのコンテナが独自のプロバイズを宣言
- スコープは子要素のみ

### スコープと優先順位
- 同じキーを親と子が宣言した場合、子の値が親の値を「シャドウ」（上書き）
- 兄弟要素には影響しない

### 解決順序
最も内側のスコープから外側へ探索:
1. 親コンテナのプロバイズ
2. 祖父母のプロバイズ
3. スクリーン設定

### 参照特性
- Provides 値は他のプロバイズキーを参照可能（段階的に解決）
- 循環参照は自動検出されて置き換えられない
- 存在しないキーは `@{key}` の文字列のまま残る（欠落を視覚的に確認可能）

---

## 09. 変数バインディング

### Variable コンポーネント
構造的なプレースホルダーとして機能し、実行時にコンテンツに置き換えられる。
- キー（例: `%content`）を設定すると Tasker がテキストまたはコンポーネントのJSON配列で埋入
- 複数の兄弟要素に展開される場合、各要素のIDに自動的に `_1`、`_2` などの接尾辞が付加（例: `title_2` で2番目の項目を更新）

### 文字列変数置換
Tasker変数（`%` 接頭辞）はテキストやプロパティに直接使用可能。

| 変数種別 | 説明 |
|---------|------|
| **グローバル変数**（大文字を含む名前、例: `%VOLM`） | 自動的に更新され、追加処理なしにレイアウトが最新値を常に反映 |
| **ローカル変数**（すべて小文字） | タスク内でスコープされ、更新には **Update Scene V2** アクションで新しい値の設定が必要 |

### 環境変数（9種類）
- `%sv2_display_width`
- `%sv2_render_is_portrait`
- 他 7種類

---

## 10. 条件付き表示（showWhen）

変数式に基づいてコンポーネントの表示/非表示を制御。**式が偽の場合、コンポーネントと子要素はレイアウトから削除され、スペースを占有しない**。

### サポートされる式

| 種類 | 例 |
|------|-----|
| 変数参照 | `%is_logged_in` |
| 等値比較 | `%role == "admin"` |
| 不等比較 | `%count != 0` |
| 数値比較 | `%VOLM > 50`（`>`、`<`、`>=`、`<=` 対応） |
| 論理 AND | `%is_logged_in & %is_premium` |
| 論理 OR | `%is_admin \| %is_moderator` |
| 論理 NOT | `!%is_hidden` |
| グループ化 | `(%a \| %b) & %c` |

### 重要な制限
- showWhen 式は **Tasker の `%variables` のみ**を解決
- Provides の値は利用不可
- 存在しない変数は式全体を偽に評価
- 変数変更時は自動的に再評価

### エディタでのテスト
フラスコアイコンから「Test Variables」パネルを開き、変数値を設定してプレビューでリアルタイム確認。

---

## 11. インタラクション（Event Handlers）

### 設定手順
1. Elements ツリーからコンポーネントを選択
2. Interaction タブに切り替え
3. 「Add Handler」をタップして新規ハンドラを作成

### ジェスチャータイプ

| ジェスチャー | 内部コード | 説明 |
|------------|-----------|------|
| シングルクリック | `click:1` | 標準的なタップ |
| ダブルタップ | `click:2` | 2回タップ |
| トリプルタップ | `click:3` | 3回タップ |
| N クリック | `click:N` | 任意回数のタップ |
| 長押し | `longPress` | プレス＆ホールド |
| スワイプ | `swipe` | 8方向スワイプ（ジェスチャーパスデータ取得可能） |

### アクションタイプ

| アクション | 説明 |
|-----------|------|
| **コマンド** | テキスト文字列を送信。Taskerの専用イベントで検知 |
| **タスク** | Taskerの特定タスクを実行。ローカル変数として値を渡せる |

### 自動提供されるスコープ変数
- `%sv2_screen_name`
- `%sv2_layout_id`
- `%sv2_element_id`
- `%sv2_element_path`
- `%sv2_element_value`

### 追加機能
- **ハプティックフィードバック**: タップ時の振動（独立して機能）
- **ドラッグして移動**: オーバーレイ専用、コンポーネントをドラッグハンドルにする

### Clickable modifier — JSON 形式（確認済み 2026-03-16）

タスクを呼び出すだけの場合:

```json
{
  "type": "Clickable",
  "actions": {
    "click:1": { "task": "タスク名" }
  }
}
```

タスクに変数（ローカル変数）を渡す場合（`variables` キーを追加）:

```json
{
  "type": "Clickable",
  "actions": {
    "click:1": { "task": "タスク名", "variables": { "val": "値" } }
  }
}
```

シーンを閉じながらタスクを呼ぶ場合（`dismissInteractionType` を追加）:

```json
{
  "type": "Clickable",
  "actions": {
    "click:1": { "task": "タスク名" }
  },
  "dismissInteractionType": "click:1"
}
```

- `variables` で渡した値は呼び出し先タスクのローカル変数として受け取れる（例: `%val`）
- ジェスチャーは `"click:1"`（シングルタップ）/ `"longPress"` など複数定義可能

### レイアウト終了時の値返却
- TextInput や Switch は「result binding」設定で、終了ボタンの値を自動的に含められる

---

## 12. Tasker アクション

> アクションコードは実機 XML（Scene_V2_Action_Code.tsk.xml）で確認済み（2026-03-16）。

### アクション一覧

| Code | アクション名 | 説明 |
| ---- | ---------- | ---- |
| 479 | Show Scene v2 | シーンを表示（arg1: JSON, arg2: Screen ID, arg3: Display Mode, arg8: Blocking Overlay） |
| 480 | Dismiss Scene v2 | シーンを閉じる（arg0: Screen ID） |
| 481 | Update Scene v2 | 要素を1つ更新（arg0: Screen ID, arg1: Element ID） |
| 482 | Update Scene v2 Overlay | ウィンドウ設定変更（arg0: Screen ID, arg5: Blocking Overlay, arg6: Transition Duration ms） |
| 483 | Get Scene v2 Element Value | 要素の値を読み取り（arg1: Screen ID, arg2: Element ID） |
| 484 | Wait For Scene v2 Result | シーンが閉じるまでタスクをブロック（arg1: Screen ID） |

### Get Scene v2 Element Value (483) — 戻り変数

| 変数 | 説明 |
| ---- | ---- |
| `%sv2_value()` | 取得した値（配列。単一要素なら `%sv2_value(1)` で参照） |
| `%sv2_element_id()` | 取得した要素のID |
| `%sv2_result_count` | 結果件数 |

読み取り可能な要素: TextInput（入力テキスト）、Switch（on/off）など

### Update Scene v2 (481) — 注意事項

- **Element ID 単位で1要素ずつ更新**（バッチ変数更新ではない）
- グローバル変数はシーンに**自動反映**されるため Update Scene v2 不要
- ローカル変数の更新のみ必要な場合に使用
- 複数要素を更新する場合は Update Scene v2 を複数回呼ぶ

### Show Scene v2 の主要オプション
- `fullscreen` / `dialog` / `overlay`
- インラインシーン対応（アクション内に直接シーンを定義）
- 特定の `displayId` をターゲット指定可能（マルチディスプレイ対応）
- **Auto-dismiss**: ミリ秒単位で設定し、タイムアウト後に自動閉鎖
- **Continue Task Immediately**: レイアウト閉鎖を待たずにタスク継続（常時表示オーバーレイ等に有用）

### ID ベースシステム
- 同一 ID で表示 → 既存レイアウトを上書き更新
- 新規 ID で表示 → 別ウィンドウとして起動

---

## 13. スクリーンプロパティ

### 一般設定（General）
スクリーン名はエディタとTaskerでの識別に使用。JSONの `"name"` フィールドに保存。

### Provides（スクリーンレベル）
全コンポーネントツリーで利用可能なキーと値のペアを設定。同じレイアウトで異なるデータを表示する際に活用。

### システムバー制御（System Bars）
**フルスクリーンおよびダイアログモードのみ適用（オーバーレイには適用されない）**

| プロパティ | 説明 |
|-----------|------|
| `statusBarColor` | ステータスバーの背景色（hex または テーマカラー名） |
| `isLightStatusBars` | `true` で暗いアイコン表示 |
| `navigationBarColor` | ナビゲーションバーの背景色 |
| `isLightNavigationBars` | `true` でライトテーマに対応 |
| `hideSystemBars` | `true` で両バーを完全非表示（フルスクリーンモード） |

---

## 14. 表示モード（Display Modes）

| モード | 説明 | 適用例 |
|--------|------|--------|
| **Fullscreen** | 画面全体をスタンドアロンアクティビティとして表示 | ダッシュボード、複雑なフォーム |
| **Dialog** | 浮遊ウィンドウ（背後のアプリが見える状態） | 確認ダイアログ、入力プロンプト |
| **Overlay** | 他のアプリの上に重ねて表示（最も柔軟） | 常時表示情報、フローティングパネル |

### パーミッション要件
- オーバーレイモード: 「Display over other apps」権限が必要
- ロック画面表示: 「Show Over Everything」オプションが必要

### マルチディスプレイ対応
折りたたみ型端末や外部モニター接続時、物理ディスプレイIDを指定して特定画面をターゲット化できる。

---

## 15. オーバーレイ詳細

### 位置とサイズ設定

| 単位 | 説明 |
|------|------|
| **dp** | 正の値は左/上から、負の値は右/下から配置 |
| **パーセンテージ** | オーバーレイの中心が指定のパーセンテージに配置 |

幅と高さは明示的に設定するか、コンテンツに合わせて空にできる。

### ブロッキング機能
- デフォルト: タッチをブロック
- 「Blocking Overlay」オプションを無効にする → タッチが背後のアプリに通過（情報表示用に有用）
- **Android 12+**: 非ブロッキングオーバーレイが特別な設定を必要とする

### アニメーション（11プリセット）
SlideFromBottom / Top / Left / Right / FadeIn / FadeOut / ZoomIn / ZoomOut / BounceIn / Slide+Fade コンボ / その他

### 設定トランジション
「Update Scene V2 Overlay」アクション使用時、位置やサイズの変更をミリ秒単位の継続時間とイージング曲線でアニメーション化できる。

### ドラッグ可能機能
「Drag to Move」トグルでユーザーがオーバーレイウィンドウを移動可能。

### 制限事項
- **TextInput コンポーネントはオーバーレイモードで動作しない**（テキスト入力が必要な場合は別モードを使用）

---

## 16. AI 生成機能

### 使用手順
1. AI タブに切り替える
2. レイアウトの説明を入力（例: 「設定パネルをWiFi、Bluetooth、ダークモード切り替え機能付きで作成」）
3. 「Generate」をタップ

コンポーネントを選択すると、AIがそれを認識し「このテキストを大きくして」などの指示が可能。

### メニューオプション（3ドットメニュー）
- **Copy Instructions**: AIシステム指示をコピー
- **Export Instructions**: 指示をファイルに保存
- **Change Config**: AI設定を変更（モデル、APIキーなど）

これらの指示は ChatGPT や Claude などの他の LLM でも使用可能。

### プロンプト作成のコツ
- 構造を具体的に記述（例: 「3枚のカード下部を持つヘッダー行付きの列」）
- 特定の色が必要でない限り色について言及しない
- インタラクションを説明（例: 「右上隅に閉じるボタンを追加」）
- 初期版を生成後、個別コンポーネントを選択して調整

---

## 17. Raw JSON 編集

### 基本
ビジュアルエディタとJSONは双方向同期。すべてのビジュアル変更がJSONに反映され、すべてのJSON変更がビジュアルエディタに反映される。

### 主な用途
1. **レイアウト共有**: JSONをコピーペーストして設定を共有
2. **一括編集**: 複数コンポーネント（例: 20個）の色変更で効率化
3. **デバッグ**: 各コンポーネントの設定値を正確に確認

### 妥当性インジケーター
| バッジ | 状態 |
|--------|------|
| **Valid（緑）** | 正しい形式で両エディタが同期中 |
| **Invalid（赤）** | 構文エラー発生時、ビジュアル更新が停止 |

### JSON 構造（スクリーン定義の最上位要素）

```json
{
  "name": "スクリーン名",
  "provides": {},
  "statusBarColor": "surface",
  "navigationBarColor": "surface",
  "isLightStatusBars": false,
  "isLightNavigationBars": false,
  "hideSystemBars": false,
  "root": {
    "type": "Column",
    "id": "root",
    "modifiers": [],
    "children": [
      {
        "type": "Text",
        "id": "text1",
        "text": "Hello, World!",
        "modifiers": []
      }
    ]
  }
}
```

---

## 18. Tips & パターン（レイアウトレシピ）

### 基本パターン

#### 1. Scaffold を使ったアプリ画面
```
Scaffold（推奨の出発点）
├── topBar: TopAppBar（タイトル + ナビゲーションアイコン）
├── fab: FloatingActionButton（主要アクション）
└── content: Column（スクロール可能なコンテンツ + 自動パディング）
```

#### 2. ボトムナビゲーション付き Scaffold
```
Scaffold
├── topBar: TopAppBar
├── bottomBar: BottomAppBar（複数の IconButton）
├── fab: FloatingActionButton
└── content: メインコンテンツ
```

#### 3. カードパターン
```
Column（rounded Clip + Background + Padding）
├── Text（タイトル）
└── Text（説明文）
```

#### 4. 設定パネル
```
Column
├── Row（SpaceBetween）
│   ├── Text（ラベル）
│   └── Switch
├── Divider
├── Row（SpaceBetween）
│   ├── Text（ラベル）
│   └── Switch
└── ...
```

#### 5. グリッドレイアウト
```
Column
└── Row（繰り返し）
    ├── Item（Weight modifier）
    ├── Item（Weight modifier）
    └── Item（Weight modifier）
```

#### 6. 中央配置コンテンツ
```
Box（FillSize + Center alignment）
└── コンテンツ
```

#### 7. データドリブンリスト
```
Variable（key: "%dashboard_content"）
→ Taskerから動的にコンポーネントツリーを注入
```

#### 8. 条件付きコンテンツ
```
Component A（showWhen: "%is_logged_in"）
Component B（showWhen: "!%is_logged_in"）
```

#### 9. 画像背景テキスト（ヒーローセクション）
```
Box
├── Image（背景）
├── Spacer（暗いスクリム）
└── Column（テキストオーバーレイ）
```

#### 10. コンパクトオーバーレイ
```
Column（rounded + Padding）
├── Row（閉じるボタン + タイトル）
└── Row（コントロールアイコン）
```

### ベストプラクティス

| 推奨 | 理由 |
|------|------|
| フルスクリーンアプリは Scaffold から始める | 構造が整い、Material 3 準拠 |
| Material You カラーをハードコードせずに活用 | ダーク/ライトモード自動対応 |
| アイコン操作には IconButton を使用 | 適切なタッチターゲットサイズ確保 |
| 固定サイズより Weight ベースのサイズを優先 | レスポンシブデザイン |
| コンポーネント階層を浅く保つ（6階層以上を避ける） | パフォーマンスと可読性 |
| 参照するコンポーネントには意味のある ID を割り当てる | メンテナンス性 |
| フレキシブルスペーシングには Spacer を使用 | 柔軟なレイアウト |
| 再利用する値には Provides を使用 | DRY 原則 |
| 条件付きUIには showWhen を使用 | タスク不要で宣言的に実現 |
