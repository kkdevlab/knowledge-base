# Tasker バージョン別機能一覧（6.0〜6.6）

最終更新: 2026-02-10
現在のインストール済みバージョン: **6.5.11**

---

## 凡例

- 各バージョンの**主要な新機能**を記載（バグ修正は省略）
- `<=6.5.11` = 現在使える機能
- `6.6〜` = アップデート後に使える機能

---

## v6.0（2022/07）— Tasky、アシスタント統合

| 機能 | 概要 |
|------|------|
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

## v6.1（2023/05）— アクセシビリティ、Quick Settings 強化

| 機能 | 概要 |
|------|------|
| アクセシビリティサービス管理 | 実行中のアクセシビリティサービスの有効/無効切替・状態監視 |
| Progress Dialog | 新しい進捗ダイアログ |
| Pick Photos（Android 13+） | 写真ピッカー対応 |
| Quick Setting Tile 拡張 | アイコン、ラベル、長押し/ダブルクリック/コマンド対応 |
| 5G 対応 | モバイルネットワーク状態で 5G 判定 |
| Convert Into Task | 複数アクションを自動的に新規タスクに変換 |
| Input Dialog 複数行 | 複数行入力に対応 |

**変更点**: Target API 31、WiFi Tether → WiFi Tether (Hotspot) に名称変更

---

## v6.2（2023/11）— Web サーバー、Matter ライト

| 機能 | 概要 |
|------|------|
| Web サーバー機能 | HTTP Request イベント + HTTP Response アクションで Tasker が Web サーバーとして動作 |
| Matter Light Control | Matter 対応ライトの ON/OFF・色・明るさ制御（実験的） |
| Get Network Info | 現在のネットワーク情報（SSID、IP、速度等）を一括取得 |
| Work Profile 管理 | Work Profile アプリの有効/無効切替 |
| Lock Screen Notes（Android 14） | ロック画面のデフォルトメモアプリとして動作 |
| 条件のコピー/ペースト | プロファイル条件のコピペが可能に |

**変更点**: Target API 33、Android 13+ 向け権限対応（NEARBY_WIFI_DEVICES 等）

---

## v6.3（2024/01頃）— 新 UI、Device Owner、WebUI

| 機能 | 概要 |
|------|------|
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

## v6.4（2024/07頃）— Widget v2、リモート実行

| 機能 | 概要 |
|------|------|
| Widget v2 | ビジュアルエディタ付きの高カスタマイズウィジェットシステム |
| リモートアクション実行 | 他端末の Tasker アクションを直接実行（Perform Task、Get Location v2 等） |
| Design Token カラー（Android 14+） | システムのダイナミックカラーに対応 |
| Talkback アクション | 視覚障害者向けアクセシビリティ対応 |

**変更点**: Target API 34（Android 15）

---

## v6.5（2025/05）— AI ジェネレーター、カレンダー ★現在のバージョン

| 機能 | 概要 |
|------|------|
| AI ジェネレーター | 自然言語で自動化を自動生成（Gemini/OpenRouter 対応） |
| Receive Share | Tasker を Android の共有先として使用可能 |
| カレンダー自動化（7アクション） | イベント・リマインダー・参加者の取得/編集 + Calendar Changed イベント |
| キーボード管理 | キーボードの自動切替・情報取得 |
| Widget v2 強化 | カスタムフォント、円形プログレスバー、画像ブラー、パンくずナビ |
| 高度（Altitude）変数 | 位置トラッキングに高度情報を追加 |
| JSON エンコーディング | JSON エンコード機能 |

---

## v6.6（2026/02）— Shizuku、Java コード ※要アップデート

| 機能 | 概要 |
|------|------|
| Shizuku 統合 | root なしでシステムレベル操作（機内モード、WiFi、BT、Kill App、Logcat 等） |
| Java コードアクション | 任意の Java コード + Android API を直接実行（AI アシスタント付き） |
| 日の出/日の入りアクション | オフラインで計算（dawn、dusk、solar noon 等） |
| 通知 Live Update | ステータスバーにチップ/展開ステータス表示 |
| 通知グループ設定 | 通知の整理・分類 |
| クリップボードインポート | CTRL+V でプロファイル/タスクをインポート |
| Extra Triggers | 外部アプリ（Home、Car、Bixby 等）から Tasker を起動 |
| Work Profile + Private Space | Private Space にも対応 |

**変更点**: Target API 35、最小 SDK 24（Android 7.0）、**App Factory 廃止**

---

## バージョン間の主要な進化まとめ

```
6.0  基盤整備（Tasky、バッテリー/画面情報取得）
 ↓
6.1  Quick Settings・アクセシビリティ強化
 ↓
6.2  Web サーバー機能・Matter IoT 連携
 ↓
6.3  新 UI・Device Owner・WebUI（リモート制御の基盤）
 ↓
6.4  Widget v2・リモートアクション実行
 ↓
6.5  AI 自動生成・カレンダー連携・Widget v2 強化 ★現在
 ↓
6.6  Shizuku（root 代替）・Java コード・通知強化
```

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
