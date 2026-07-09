# Tasker 主要プラグイン一覧（joaomgcd系 + join）

最終更新: 2026-07-07

Tasker 本体を拡張する代表的なプラグイン群（開発者 joaomgcd の Auto シリーズ + join）の機能まとめ。公式サイト（joaoapps.com）の記載に基づく汎用情報。プロジェクトごとの導入状況・パッケージ名の実機確認結果は各プロジェクトの `CLAUDE.md` 側に記録する。

---

## 一覧

| プラグイン | 主なパッケージ名 | 用途（公式マニュアル準拠） |
| --- | --- | --- |
| AutoApps | com.joaomgcd.autoapps / com.joaomgcd.autoappshub | Auto系プラグイン20種以上の購入・アンロックを一括管理するハブアプリ。Google Play公式リスティングは `autoappshub` 側 |
| AutoContacts | com.joaomgcd.autocontacts | 連絡先を名前/ニックネームで検索し、電話番号・メール・住所・誕生日等をTasker変数として取得 |
| AutoInput | com.joaomgcd.autoinput | 非root・アクセシビリティサービスによるUI自動操作（タップ・テキスト入力）、画面内テキスト取得、Android 7+では顔認証ロック解除にも対応 |
| AutoLaunch | com.joaomgcd.autolaunch（+ `.unlock`） | アプリ名/ニックネームでの起動、インストール済みアプリのクエリ、カスタムランチャーメニュー作成 |
| AutoLocation | com.joaomgcd.autolocation（+ `.unlock`） | 活動検知（徒歩/車/自転車/静止/傾き）、ジオフェンス監視（最大100件）、高精度位置取得、距離計算・逆ジオコーディング |
| AutoNotification | com.joaomgcd.autonotification | 通知の監視・ブロック・カスタム通知作成（フォント/画像/ボタン最大50個）、Quick Settingsタイル追加、他アプリのToastインターセプト |
| AutoTools | com.joaomgcd.autotools | JSON/HTML/XML読み書き、正規表現、配列操作、ダイアログ、Web Screens、OCR、懐中電灯パターン等の拡張アクション集 |
| AutoVoice | com.joaomgcd.autovoice | 音声コマンド認識・自然言語対応、Bluetoothヘッドセットボタン反応、環境音検知（ベビーモニター等） |
| AutoRemote | com.joaomgcd.autoremote | PC/ブラウザ/他端末とのメッセージ送受信によるリモート制御、URL共有、リモート通知表示 |
| AutoShare | com.joaomgcd.autoshare | Android共有メニューから受けたテキスト/ファイルをTaskerで加工し呼び出し元へ返却（Direct Share対応） |
| join | com.joaomgcd.join | 通知同期・SMS送受信・クリップボード共有・ファイル転送・リモートページ表示（Chrome拡張版・デスクトップ版含む） |

---

## 参照元（公式サイト）

- [AutoApps (Google Play)](https://play.google.com/store/apps/details?id=com.joaomgcd.autoappshub)
- [AutoContacts](https://joaoapps.com/autocontacts/)
- [AutoInput](https://joaoapps.com/autoinput/)
- [AutoLaunch](https://joaoapps.com/autolaunch/)
- [AutoLocation](https://joaoapps.com/autolocation/)
- [AutoNotification](https://joaoapps.com/autonotification/)
- [AutoTools](https://joaoapps.com/autotools/)
- [AutoVoice](https://joaoapps.com/autovoice/)
- [AutoRemote](https://joaoapps.com/autoremote/)
- [AutoShare](https://joaoapps.com/autoshare/)
- [Join by joaoapps](https://joaoapps.com/join/)

---

## 関連（他に確認したSQLite系アプリ、Tasker非連携）

- `com.jordanhotmann.taskersqliteplugin`（SQLite Plugin）: SQL文をTaskerアクションから実行できるプラグイン。Google Playからは削除済みだがAPKは現存
- `com.lastempirestudio.sqliteprime`（SqlitePrime）: 単体のSQLite DB管理・閲覧アプリ。Tasker連携機能なし
