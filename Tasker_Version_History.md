# Tasker バージョン管理

最終更新: 2026-06-16
インストール済み: **6.7.5-beta**
最新リリース: **6.6.20**（2026/02/24）
ベータ版: **6.7.5-beta**（Projects in new UI, JSON Write, 50 QS Tiles, Scenes V2 WebView/Video — 詳細: `Tasker_ScenesV2_67x-beta.md`）

---

## 凡例

- **最新版機能一覧**: 6.0〜6.6 で追加された主要機能をカテゴリ別に整理
- **変更履歴**: バージョン別の追加機能・変更点（バグ修正は省略）
- 今後の更新: 新バージョンが出たら変更履歴に追記 → 最新版一覧も更新

---

## 最新版（6.6.18）機能一覧

6.0 以降に追加された主要機能を統合。「Tasker でこれできる？」の確認用。

### デバイス制御・システム

| 機能 | 概要 | 追加 |
| ------ | ------ | ------ |
| Shizuku 統合 | root なしでシステムレベル操作（機内モード、WiFi、BT、Kill App、Logcat 等） | 6.6 |
| Device Admin/Owner | root なしでアプリ管理（停止、凍結、強制終了、データ消去、権限設定、再起動） | 6.3 |
| 飛行機モード制御 | root/ADB なしで切り替え可能 | 6.0 |
| Device Effects | グレースケール化、壁紙暗転、AOD 切替（Android 15+） | 6.3 |
| 振動強度制御 | バイブレーション強度の調整 | 6.3 |
| Work Profile/Private Space | Work Profile + Private Space のアプリ有効/無効切替 | 6.2/6.6 |
| Samsung Routines 権限 | Android 14+ で Samsung Routines 権限に対応 | 6.6 |

### ネットワーク・Web

| 機能 | 概要 | 追加 |
| ------ | ------ | ------ |
| Web サーバー機能 | HTTP Request イベント + HTTP Response で Tasker が Web サーバーとして動作 | 6.2 |
| HTTP Request 改善 | Content-Type ヘッダの自動検出・設定 | 6.0 |
| Get Network Info | SSID、IP、速度等を一括取得 | 6.2 |
| Get Network Data Usage | 期間指定でアプリごとのデータ通信量を取得 | 6.3 |
| WiFi Tether | Android 16+ では Shizuku 経由で動作可能 | 6.1/6.6 |
| 5G 対応 | モバイルネットワーク状態で 5G 判定 | 6.1 |
| Wifi Changed イベント | WiFi 状態が変化したときにトリガー | 6.7.4-beta |

### UI・ウィジェット

| 機能 | 概要 | 追加 |
| ------ | ------ | ------ |
| New Main Screen UI（2026 UI） | タグシステム（プロジェクト代替）・Automations概念・フィルター方式・ライブ実行追跡（opt-in） | 6.7.4-beta |
| 新UIにプロジェクト復活 | 新 Main Screen UI にプロジェクトを第一級概念として再導入（左ナビペイン・D&D並べ替え・カラー・項目移動）。タグだけの運用も継続可 | 6.7.5-beta |
| Scenes V2 WebView | シーン内に HTML/JS/CSS を埋め込み表示（v1 同様、オンライン/オフライン両対応） | 6.7.5-beta |
| Scenes V2 Video | シーン内に動画プレーヤー。再生制御・状態監視（playing/position 等）対応 | 6.7.5-beta |
| 50個の動的 QS タイル | 47個の新クイック設定タイル。設定したものだけ表示される動的方式。Clear Quick Setting Tile で消去/再設定で元位置復帰 | 6.7.5-beta |
| Modern UI | Edit Task 画面の新デザイン（インライン編集、マルチ選択、折りたたみ） | 6.3 |
| Widget v2 | ビジュアルエディタ付き高カスタマイズウィジェット | 6.4 |
| Widget v2 強化 | カスタムフォント、円形プログレスバー、画像ブラー、パンくずナビ | 6.5 |
| Progress Dialog | 進捗ダイアログ | 6.1 |
| Design Token カラー | システムのダイナミックカラーに対応（Android 14+） | 6.4 |
| 壁紙 Center オプション | 壁紙アクションに Center 配置オプション追加 | 6.6 |

### 通知

| 機能 | 概要 | 追加 |
| ------ | ------ | ------ |
| Live Update | ステータスバーにチップ/展開ステータス表示 | 6.6 |
| Short Critical Text | ステータスバーチップに短縮テキスト表示 | 6.6 |
| グループ設定 | 通知の整理・分類（Android 16+ で動作変更あり） | 6.6 |
| SVG アイコン | Notify アクション・クイック設定タイルで SVG を直接使用可能 | 6.7.4-beta |

### 自動化・コード実行

| 機能 | 概要 | 追加 |
| ------ | ------ | ------ |
| AI ジェネレーター | 自然言語で自動化を自動生成（Gemini/OpenRouter 対応） | 6.5 |
| Tasky | 事前作成ルーチンをインポートできる簡易自動化モード | 6.0 |
| Java コードアクション | 任意の Java コード + Android API を直接実行（AI アシスタント、RxJava2 対応） | 6.6 |
| Extra Triggers | 外部アプリ（Home、Car、Bixby 等）から Tasker を起動 | 6.6 |
| Convert Into Task | 複数アクションを自動的に新規タスクに変換 | 6.1 |
| Test Action ボタン | アクション編集画面から直接テスト実行（設定しながら確認可能） | 6.7.4-beta |
| Apply ボタン | タスク編集画面でアクションを即時適用 | 6.7.4-beta |

### 情報取得

| 機能 | 概要 | 追加 |
| ------ | ------ | ------ |
| Get Screen Info | アシスタント権限で現在の URL・テキスト情報を取得 | 6.0 |
| Get Battery Info | バッテリー情報（レベル、温度、電圧等）を取得 | 6.0 |
| Get Pixel Color | 画面上のピクセル色を取得 | 6.0 |
| Get Sunrise/Sunset | オフラインで日の出/日の入り計算（dawn、dusk、solar noon 等） | 6.6 |
| 高度（Altitude）変数 | 位置トラッキングに高度情報を追加 | 6.5 |
| List File Properties | 複数ファイルのプロパティを一括取得 | 6.3 |

### データ処理

| 機能 | 概要 | 追加 |
| ------ | ------ | ------ |
| Array Compare | 複数配列の共通要素・差分要素を抽出 | 6.3 |
| Arrays Merge | 配列の結合（配列内の変数置換にも対応） | 6.6 |
| JSON エンコーディング | JSON エンコード機能 | 6.5 |
| JSON ネイティブ書き込み | 変数名のドット記法で JSON 構造を生成（Variable Set/Clear・Array Set/Push/Pop）。中間構造も自動生成 | 6.7.5-beta |
| Format JSON アクション | JSON の minify / pretty-print（入力変数の上書きオプション付き） | 6.7.5-beta |

### 外部連携・IoT

| 機能 | 概要 | 追加 |
| ------ | ------ | ------ |
| Tasker WebUI | HTTP API でネットワーク上の他端末からリモート制御 | 6.3 |
| リモートアクション実行 | 他端末の Tasker アクションを直接実行 | 6.4 |
| Matter Light Control | Matter 対応ライトの ON/OFF・色・明るさ制御（実験的） | 6.2 |
| Receive Share | Tasker を Android の共有先として使用可能 | 6.5 |
| カレンダー自動化 | イベント・リマインダー・参加者の取得/編集 + Calendar Changed イベント（7アクション） | 6.5 |
| Google Drive Upload | ファイル形式の自動変換に対応 | 6.0 |
| App Factory 復活 | Scenes V2 対応・API 35 ターゲット（6.6 で廃止 → 復活） | 6.7.4-beta |

### アクセシビリティ

| 機能 | 概要 | 追加 |
| ------ | ------ | ------ |
| サービス管理 | アクセシビリティサービスの有効/無効切替・状態監視 | 6.1 |
| Talkback アクション | 視覚障害者向けアクセシビリティ対応 | 6.4 |
| Accessibility Helps Usage Stats | 使用統計にアクセシビリティを活用するオプション | 6.6 |

### その他

| 機能 | 概要 | 追加 |
| ------ | ------ | ------ |
| Quick Setting Tile 拡張 | アイコン、ラベル、長押し/ダブルクリック/コマンド対応 | 6.1 |
| キーボード管理 | キーボードの自動切替・情報取得 | 6.5 |
| クリップボードインポート | CTRL+V でプロファイル/タスクをインポート | 6.6 |
| Lock Screen Notes | ロック画面のメモアプリとして動作（Android 14+） | 6.2 |
| Pick Photos | 写真ピッカー対応（Android 13+） | 6.1 |
| Input Dialog 複数行 | 複数行入力に対応 | 6.1 |
| 条件のコピー/ペースト | プロファイル条件のコピペが可能 | 6.2 |
| Running Tasks / Active Profiles 画面 | 実行中タスク・有効プロファイルの管理 | 6.0 |

### 現在の制約・注意点

| 項目 | 内容 |
| ------ | ------ |
| Target API | 35 |
| 最小 SDK | 24（Android 7.0） |
| App Factory | **復活**（6.7.4-beta〜、Scenes V2 対応・API 35）※6.6 で一時廃止 |
| DND モード切替 | Tasker Settings アプリが必要 |
| Logcat Entry | Shizuku 経由で制限デバイスでも復活 |

---

## 変更履歴

### v6.7.5-beta（2026/06）— Projects in new UI, JSON Write, 50 QS Tiles, Scenes V2 WebView/Video

| 機能 | 概要 |
| ------ | ------ |
| **新UIにプロジェクト復活** | 新 Main Screen UI にプロジェクトを第一級概念として再導入。左ナビペインに表示（折りたたみ時も表示）、ドラッグ＆ドロップ並べ替え、検索フィルタ、アクセントカラー、アイテムのプロジェクト間移動（ロングクリック→移動）。不要ならタグだけで運用も可 |
| Automation のトリガー表示位置変更 | プロファイル編集でコンテキスト（トリガー）が上・アクションが下に。新規作成時はトリガーが折りたたみ「OPTIONAL」セクションに格納 |
| **JSON ネイティブ書き込み** | 変数名に**ドット記法**を使うだけで JSON 構造を生成。対応アクション: Variable Set / Variable Clear / Array Set / Array Push / Array Pop。例: `%json.address.city` で中間構造も自動生成 |
| **Format JSON アクション**（新規） | JSON の minify / pretty-print。入力変数の上書きオプション付き |
| **Scenes V2 WebView**（新規） | シーン内に HTML/JS/CSS を埋め込み表示（オンライン/オフライン両対応） |
| **Scenes V2 Video**（新規） | シーン内に動画プレーヤー。playing / position / formattedPosition の states、再生制御アクション・イベント対応 |
| Scenes V2 タップ座標 | タップ系イベント（short/long/multi-tap）でタスクに `%sv2_tap_x` / `%sv2_tap_y` を渡せる |
| Scenes V2 オーバーレイ拡張 | 画面外への描画、ドラッグで画面外に出たら自動で戻すオプション。各プロパティに (i) 説明ボタン |
| **50個の動的 QS タイル** | 47個の新クイック設定タイルを追加。設定したものだけ表示される「動的」方式。新アクション **Clear Quick Setting Tile**（無効化で消えるが再設定で元の位置に復帰） |
| Java Code 拡張 | callTask に priority 追加。`setProfileEnabled(profile, enabled)` / `toggleProfileEnabled(profile)` 追加 |
| Keep Accessibility Running | 設定にグローバルスイッチ追加 |

**主なバグ修正**:

- 高速トリガー時のメモリ蓄積問題を修正
- 新 Main Screen UI 追加に伴う複数のクラッシュ修正
- App Factory アプリで Shizuku が必要なアクションの修正
- Widget v2 のアプリアイコンベース画像の修正
- List Dialog の First Visible Index がライトテーマで見えない問題、ダーク/ライト切替時のクラッシュ修正

---

### v6.7.4-beta（2026/06）— New Main Screen UI, Scenes V2 Update 3, App Factory Revival

| 機能 | 概要 |
| ------ | ------ |
| **New Main Screen UI**（opt-in） | Tasker → Preferences → UI → Use Tasker 2026 UI で有効化 |
| タブ廃止 → フィルター方式 | 上下タブが廃止。全アイテムを1リストで表示し、タイプ・タグで絞り込む |
| タグシステム | プロジェクトの進化版。1アイテムに複数タグ付与可能、AND/OR 検索対応 |
| Automations | Profile と Task を1画面で編集。コンテキスト追加や Exit Task も UI がガイド |
| ライブ実行追跡 | 実行中・有効アイテムをリストでリアルタイム表示、直接起動/停止も可能 |
| 新 Settings 画面 | 動的生成される統一デザインの設定画面 |
| Scenes V2 Update 3 | Slider / Range Slider / Progress Bar / FlexBox / Camera の新コンポーネント追加（詳細: `Tasker_ScenesV2_67x-beta.md`） |
| Apply When（Scenes V2） | 全モディファイアに条件付き適用を設定可能（縦横向きで別レイアウト等） |
| App Factory 復活 | Scenes V2 対応・API 35 ターゲット（6.6 で廃止されたものを復活） |
| Test Action ボタン | アクション編集画面から直接テスト実行 |
| Apply ボタン | タスク編集画面でアクションを即時適用 |
| SVG アイコン | Notify アクション・クイック設定タイルで SVG を直接使用可能 |
| Wifi Changed イベント | WiFi 状態変化時にトリガー |
| Shortcut Task Priority | ショートカットタスクの優先度をオプションで設定 |
| Shizuku クリップボード | Shizuku が有効な場合にクリップボード取得を Shizuku 経由で実行 |
| Java Code スクリーンショット | Java Code アクションからアクセシビリティサービス経由でスクリーンショット取得 |

**主なバグ修正**:

- Tasker スタートアップ時のクラッシュ修正
- 多数の Scenes V2 クラッシュ・変数更新・マルチスクリーン問題修正
- `%DATE` フォーマット関連の修正
- Out of Memory クラッシュ修正
- Volume アクションの最大値検出修正

---

### v6.7.3-beta（2026/04）— Scenes V2 Update 2

> **互換性破壊（2回目）**: Scenes V2 のインタラクション設定は再設定が必要。**Result Binding が廃止**（変数ベースに統一）。既存の **Show Scene V2** アクションはすべて「結果を待たない」モードになる → **With Result** モードへの手動変更が必要。

| 機能 | 概要 |
| ------ | ------ |
| Dream Mode | Android スクリーンセーバーに Scene V2 を設定可能 |
| Interactive Editor Mode | エディタ内でそのままレイアウトをテスト操作（変数・ShowWhen 等） |
| Event Handlers 刷新 | 1イベントに複数の順序付きアクションを設定可能（Set Variable / Output to Variable / Toggle Variable / Dismiss Layout / Haptic Feedback / Run Component Action） |
| Screen Events | Back Pressed（キャンセル可）/ Screen Shown / Screen Hidden / Variable Changed |
| Flow Row / Flow Column | 内容に応じて自動折り返しする動的レイアウト |
| Segmented Button Row | ラジオボタン的コンポーネント（複数選択オプションあり） |
| Marquee modifier | テキスト等のコンポーネントを自動スクロール表示 |
| Show When Mode | 非表示時に invisible（不可視）/ gone（完全除去）を選択可能 |
| 表示モード 7種類に拡張 | `With Result` バリアント追加により結果待ちを明示的に選択する形式に |
| エディタからタスク作成 | シーンエディタから直接タスクを作成・編集可能（同プロジェクトに保存） |
| スワイプ変数 | `%sv2_swipe_length` / `%sv2_swipe_direction` 追加 |
| Transitions タブ | コンポーネントの表示/非表示時アニメーションを設定 |

**Scenes V2 以外の変更**:

- Out Of Memory エラー大幅改善
- 実行中タスク画面にアクション実行時間を表示（1秒更新）
- **Get Network Info** に Get Cell Info オプション追加
- Android 17 Advanced Protection Mode 対応（直接購入版: `isAccessibilityTool` フラグ）
- Widget V2 エディタプレビュー表示バグ修正
- `%DATE` フォーマット変更バグ修正（既に変わった場合は Android 設定 → アプリ → Tasker の言語設定からリセット）
- Shizuku 権限リクエスト修正

---

### v6.7.1-beta（2026/03）— Scenes V2 Update 1

> **互換性破壊**: Scenes V2 のインタラクション設定は再設定が必要

| 機能 | 概要 |
| ------ | ------ |
| States | 各コンポーネントに状態を持たせ、変数として UI 内部・Tasker 間で利用可能 |
| Events | コンポーネント固有イベント（テキスト変更、スイッチ切替等）で Tasker タスクを実行 |
| Actions | コンポーネントの状態を直接変更（Checkbox/Switch の Toggle 等） |
| Run Scene V2 Action | Tasker アクションからシーンコンポーネントのアクションを実行 |
| Scene V2 Event | シーン内の変化で Tasker タスクをトリガー |
| Java Code 連携 | Java Code アクションから全 Scenes V2 操作が可能、60fps 更新対応 |
| 新コンポーネント | Card / Checkbox / Navigation Bar / Navigation Item / Icon Button |
| 新モディファイア | Blur |
| Input Text in Overlay | オーバーレイでテキスト入力が動作するように |
| アニメーション GIF | Image コンポーネントでループ GIF をサポート |
| テンプレート | レイアウトをテンプレートとして保存可能 |
| AI Component Only モード | 選択中コンポーネントのみ生成（高速・既存を保持） |
| プレビュー変数解決 | プレビュー上で変数を実際の値に展開 |
| テスト変数ピン留め | エディタを離れても Test Variables が消えない |
| デフォルト表示モード | 画面プロパティでデフォルト表示モードを設定可能 |
| Keep Display On | シーン表示中に画面スリープを防止 |
| Undo/Redo 長押し | 複数ステップを一気に戻る/進む |
| エディタ 8 タブ化 | Tree / Properties / Modifiers / States / Events / Actions / JSON / AI |

**Scenes V2 以外の変更**:

- **Wifi Connected** 状態：位置情報権限不要オプション追加
- **Calendar Entry** 状態：`Start Early` / `End Later` 引数追加
- `ACCESS_LOCATION_EXTRA_COMMANDS` 権限追加
- Shizuku 非使用オプション追加

---

### v6.6（2026/02）— Shizuku、Java コード

| 機能 | 概要 |
| ------ | ------ |
| Shizuku 統合 | root なしでシステムレベル操作（機内モード、WiFi、BT、Kill App、Logcat 等） |
| Java コードアクション | 任意の Java コード + Android API を直接実行（AI アシスタント、RxJava2 対応） |
| 日の出/日の入りアクション | オフラインで計算（dawn、dusk、solar noon 等） |
| 通知 Live Update | ステータスバーにチップ/展開ステータス表示 |
| 通知 Short Critical Text | ステータスバーチップに短縮テキスト表示 |
| 通知グループ設定 | 通知の整理・分類 |
| クリップボードインポート | CTRL+V でプロファイル/タスクをインポート |
| Extra Triggers | 外部アプリ（Home、Car、Bixby 等）から Tasker を起動 |
| Work Profile + Private Space | Private Space にも対応 |
| 壁紙 Center オプション | 壁紙アクションに Center 配置オプション追加 |
| Arrays Merge 変数置換 | 配列内の変数置換に対応 |
| Samsung Routines 権限 | Android 14+ で Samsung Routines 権限追加 |
| Accessibility Helps Usage Stats | プリファレンスに新オプション追加 |

**変更点**:

- Target API 35、最小 SDK 24（Android 7.0）
- **App Factory 廃止**
- 既存アクションが Shizuku 対応（飛行機モード、WiFi、BT、Kill App 等）
- WiFi Tether が Android 16+ で Shizuku 経由で動作可能
- Logcat Entry が制限デバイスで復活
- DND モード切替に Tasker Settings アプリが必要
- Android 16+ 向け通知グループ動作変更

---

### v6.5（2025/05）— AI ジェネレーター、カレンダー

| 機能 | 概要 |
| ------ | ------ |
| AI ジェネレーター | 自然言語で自動化を自動生成（Gemini/OpenRouter 対応） |
| Receive Share | Tasker を Android の共有先として使用可能 |
| カレンダー自動化（7アクション） | イベント・リマインダー・参加者の取得/編集 + Calendar Changed イベント |
| キーボード管理 | キーボードの自動切替・情報取得 |
| Widget v2 強化 | カスタムフォント、円形プログレスバー、画像ブラー、パンくずナビ |
| 高度（Altitude）変数 | 位置トラッキングに高度情報を追加 |
| JSON エンコーディング | JSON エンコード機能 |

---

### v6.4（2024/07頃）— Widget v2、リモート実行

| 機能 | 概要 |
| ------ | ------ |
| Widget v2 | ビジュアルエディタ付きの高カスタマイズウィジェットシステム |
| リモートアクション実行 | 他端末の Tasker アクションを直接実行（Perform Task、Get Location v2 等） |
| Design Token カラー（Android 14+） | システムのダイナミックカラーに対応 |
| Talkback アクション | 視覚障害者向けアクセシビリティ対応 |

**変更点**: Target API 34（Android 15）

---

### v6.3（2024/01頃）— 新 UI、Device Owner、WebUI

| 機能 | 概要 |
| ------ | ------ |
| Device Admin/Owner Action | root なしでアプリ管理（停止、凍結、強制終了、データ消去、権限設定、再起動） |
| Modern UI（新 UI） | Edit Task 画面の新デザイン（インライン編集、変数サポート、マルチ選択、折りたたみ） |
| Tasker WebUI | HTTP API でネットワーク上の他端末からリモートでタスク制御 |
| List File Properties | 複数ファイルのプロパティを一括取得 |
| Get Network Data Usage | 期間指定でアプリごとのデータ通信量を取得 |
| Array Compare | 複数配列の共通要素・差分要素を抽出 |
| Device Effects（Android 15+） | グレースケール化、壁紙暗転、AOD 切替 |
| 振動強度制御 | バイブレーション強度の調整 |

**変更点**: Device Owner 権限があれば WiFi/Bluetooth/Kill App で root 不要に

---

### v6.2（2023/11）— Web サーバー、Matter ライト

| 機能 | 概要 |
| ------ | ------ |
| Web サーバー機能 | HTTP Request イベント + HTTP Response アクションで Tasker が Web サーバーとして動作 |
| Matter Light Control | Matter 対応ライトの ON/OFF・色・明るさ制御（実験的） |
| Get Network Info | 現在のネットワーク情報（SSID、IP、速度等）を一括取得 |
| Work Profile 管理 | Work Profile アプリの有効/無効切替 |
| Lock Screen Notes（Android 14） | ロック画面のデフォルトメモアプリとして動作 |
| 条件のコピー/ペースト | プロファイル条件のコピペが可能に |

**変更点**: Target API 33、Android 13+ 向け権限対応（NEARBY_WIFI_DEVICES 等）

---

### v6.1（2023/05）— アクセシビリティ、Quick Settings 強化

| 機能 | 概要 |
| ------ | ------ |
| アクセシビリティサービス管理 | 実行中のアクセシビリティサービスの有効/無効切替・状態監視 |
| Progress Dialog | 新しい進捗ダイアログ |
| Pick Photos（Android 13+） | 写真ピッカー対応 |
| Quick Setting Tile 拡張 | アイコン、ラベル、長押し/ダブルクリック/コマンド対応 |
| 5G 対応 | モバイルネットワーク状態で 5G 判定 |
| Convert Into Task | 複数アクションを自動的に新規タスクに変換 |
| Input Dialog 複数行 | 複数行入力に対応 |

**変更点**: Target API 31、WiFi Tether → WiFi Tether (Hotspot) に名称変更

---

### v6.0（2022/07）— Tasky、アシスタント統合

| 機能 | 概要 |
| ------ | ------ |
| Tasky | 事前作成ルーチンをインポートできる簡易自動化モード |
| Get Screen Info (Assistant) | アシスタント権限で現在の URL・テキスト情報を取得 |
| 飛行機モード制御 | root/ADB なしで飛行機モードを切り替え可能 |
| Get Battery Info | バッテリー情報（レベル、温度、電圧等）を取得 |
| Get Pixel Color | 画面上のピクセル色を取得 |
| Running Tasks 画面 | 実行中タスクの管理・編集・停止 |
| Active Profiles 画面 | 現在有効なプロファイルの一覧表示 |
| Google Drive Upload 強化 | ファイル形式の自動変換に対応 |
| HTTP Request 改善 | Content-Type ヘッダの自動検出・設定 |

---

## ソース

- [公式リリースノート一覧](https://tasker.joaoapps.com/changes.html)
- [v6.0](https://tasker.joaoapps.com/changes/changes6.0.html)
- [v6.1](https://tasker.joaoapps.com/changes/changes6.1.html)
- [v6.2](https://tasker.joaoapps.com/changes/changes6.2.html)
- [v6.3](https://tasker.joaoapps.com/changes/changes6.3.html)
- [v6.4](https://tasker.joaoapps.com/changes/changes6.4.html)
- [v6.5](https://tasker.joaoapps.com/changes/changes6.5.html)
- [v6.6](https://tasker.joaoapps.com/changes/changes6.6.html)
- [公式ブログ](https://joaoapps.com/)
