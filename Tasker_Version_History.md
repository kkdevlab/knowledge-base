# Tasker バージョン管理

最終更新: 2026-02-11
インストール済み: **6.5.11**
最新リリース: **6.6.18**（2026/02/05）

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

### UI・ウィジェット

| 機能 | 概要 | 追加 |
| ------ | ------ | ------ |
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

### 自動化・コード実行

| 機能 | 概要 | 追加 |
| ------ | ------ | ------ |
| AI ジェネレーター | 自然言語で自動化を自動生成（Gemini/OpenRouter 対応） | 6.5 |
| Tasky | 事前作成ルーチンをインポートできる簡易自動化モード | 6.0 |
| Java コードアクション | 任意の Java コード + Android API を直接実行（AI アシスタント、RxJava2 対応） | 6.6 |
| Extra Triggers | 外部アプリ（Home、Car、Bixby 等）から Tasker を起動 | 6.6 |
| Convert Into Task | 複数アクションを自動的に新規タスクに変換 | 6.1 |

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

### 外部連携・IoT

| 機能 | 概要 | 追加 |
| ------ | ------ | ------ |
| Tasker WebUI | HTTP API でネットワーク上の他端末からリモート制御 | 6.3 |
| リモートアクション実行 | 他端末の Tasker アクションを直接実行 | 6.4 |
| Matter Light Control | Matter 対応ライトの ON/OFF・色・明るさ制御（実験的） | 6.2 |
| Receive Share | Tasker を Android の共有先として使用可能 | 6.5 |
| カレンダー自動化 | イベント・リマインダー・参加者の取得/編集 + Calendar Changed イベント（7アクション） | 6.5 |
| Google Drive Upload | ファイル形式の自動変換に対応 | 6.0 |

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
| App Factory | **廃止**（将来のアプリ生成は別手段で対応予定） |
| DND モード切替 | Tasker Settings アプリが必要 |
| Logcat Entry | Shizuku 経由で制限デバイスでも復活 |

---

## 変更履歴

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
