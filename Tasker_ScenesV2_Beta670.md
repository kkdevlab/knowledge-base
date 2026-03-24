# Tasker 6.7.0-beta: Scenes V2 仕様メモ

> **注意**: これはベータ版（6.7.0-beta）の仕様です。正式版リリース時に変更される可能性があります。
> 正式版では差分情報のみ公開される可能性があるため、ベータ版の全仕様をここに記録しています。

- **ソース**: Reddit r/tasker 投稿（[DEV] Tasker 6.7.0-beta - Scenes V2!）
- **記録日**: 2026-03-13
- **デモ動画**: https://youtu.be/SU0pG36GkUo

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

### UI 要素
| コンポーネント | 説明 |
|--------------|------|
| TopAppBar | Material 3 トップバー |
| BottomAppBar | Material 3 ボトムバー |
| FloatingActionButton | FAB |
| Text | テキスト（サイズ/色/配置/縦配置設定可能） |
| Image | URL / Material Icon / SVG 対応、contentScale・tint 設定可能 |
| Divider | 区切り線（色・太さ設定可能） |
| Button | Material 3 スタイル、dismiss + アクション対応 |
| Switch | トグル、状態トラッキング |
| TextInput | ヒント文字・オートフォーカス・リアルタイム値送信 |
| NavigationBar | ボトムナビゲーション |
| NavigationItem | ナビゲーションアイテム |

### 特殊コンポーネント
| コンポーネント | 説明 |
|--------------|------|
| Variable | `%var` の JSON をランタイムでコンポーネントツリーとしてレンダリング（動的UIリスト等） |
| Placeholder | ランタイム注入スロット（Update Scene V2 アクションで更新） |

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

---

## 表示モード（4種類）

| モード | 説明 |
|--------|------|
| Fullscreen | エッジtoエッジ、ステータスバー/ナビバー制御、イマーシブモード |
| Dialog | 中央ウィンドウ、ブラーバックグラウンド、コンテンツに自動サイズ |
| Overlay | フローティングウィンドウ、任意位置、ドラッグ可能、パススルー |
| Accessibility Overlay | 通知パネル・システム設定画面等のシステム画面上にも表示可能 |

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

---

## インタラクション詳細

- タップ / ダブルタップ / トリプルタップ / N クリック
- 長押し
- スワイプ（8方向、ジェスチャーパスデータ取得可能）
- ハプティックフィードバック
- 各ジェスチャーで可能な操作: Taskerタスク実行 / シーン閉じる / 結果値返却 / Taskerコマンド送信

---

## 条件付き表示（showWhen）

- 任意コンポーネントに変数式で表示/非表示を設定
- 比較演算子: `==` `!=` `>` `<` `>=` `<=`
- ブール演算子: `&`（AND） `|`（OR） `!`（NOT）
- タスク不要 — 完全に宣言的

---

## 変数バインディング

- JSON フィールド内に `%var` を記述可能（テキスト/色/表示条件等）
- グローバル変数はリアルタイムでシーンを自動更新（例: `%WIN`, `%BLUE`, `%BATT`）
- Update Scene V2 アクション + 変数パススルーでライブ更新
- 環境変数（9種類）:
  - `%sv2_display_width`
  - `%sv2_render_is_portrait`
  - 他 7種類

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
| Elements | コンポーネントツリー（ドラッグ&ドロップ、複数選択、コピー/カット/ペースト等） |
| Properties | 選択コンポーネントのプロパティ設定 |
| Interaction | インタラクション設定 |
| Raw JSON | JSON直接編集 |
| AI | 自然言語でのレイアウト生成 |

### その他エディター機能
- ライブプレビュー（プレビュー内クリックでツリー選択）
- Undo/Redo（50スナップショット）
- 複数選択・コピー/カット/ペースト・インデント/アウトデント・名前変更・タイプ変更・複製
- **マルチモニター対応** — 最大4ディスプレイにエディターパネルを分散
- カラーピッカー（HSV + アルファ + hex + M3テーマカラー）
- アイコンピッカー（Material 3 アイコン検索、プレビュー内タップで選択切替）
- showWhen プレビュー用テスト変数パネル
- タグ検索（例: "Padding" で "Spacing" を検索）
- 全コンポーネント・モディファイアにツールチップ
- インラインシーン構築（Show Scene V2 アクション内で直接）
- 作成時レイアウトタイプ選択（Scaffold または Column）
- Scaffold のスロットデフォルトエディター
- スコープ対応プロパティ（ColumnScope / RowScope 等）

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
- **回避策**: 隠し Text 要素（`"showWhen": "false"`）を用意し、Update Scene V2 でテキストを書き込んで状態を保持する

```json
{"type": "Text", "id": "val_busy", "text": "", "showWhen": "false"}
```
