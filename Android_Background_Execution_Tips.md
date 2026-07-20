# Android バックグラウンド実行 Tips

## 2026-07-16: BroadcastReceiverからのService起動がAndroid 12+で拒否される

- **エラー内容**: BroadcastReceiverの`onReceive()`から`context.startService()`または`context.startForegroundService()`を呼ぶと、`BackgroundServiceStartNotAllowedException`（通常Service）または`ForegroundServiceStartNotAllowedException`（Foreground Service）が発生し、レシーバーごとクラッシュする。呼び出し元が外部の自動化アプリ（Tasker等）の明示的Intentであっても同様に失敗する（実機Android 16、targetSdk 36で確認）。
- **原因**: Android 12以降、アプリがバックグラウンド状態（Activityも表示されておらず、直前にFGSも実行していない等）のときは、BroadcastReceiverから直接Service/Foreground Serviceを起動できない。これは呼び出し元アプリの特権とは無関係で、「対象アプリ自身がバックグラウンド状態かどうか」で判定される。Tasker側が一時的な背景実行許可（BroadcastOptionsの一時allowlist等）を付与してくれることを期待しても、実際には付与されないケースがある（少なくとも本件の環境では付与されなかった）。
- **解決方法**: WorkManager（`androidx.work:work-runtime-ktx`）に処理を委譲する。`WorkManager.enqueueUniqueWork(name, ExistingWorkPolicy, request)`はJobScheduler経由で実行がスケジュールされ、呼び出し元アプリの状態に依存しないため、この制限を受けずに実行できる。BroadcastReceiver側は「WorkManagerにenqueueするだけ」の薄い層にし、実処理は`CoroutineWorker`に持たせるとよい。
- **備考**: Tasker・外部Intent・システムブロードキャスト等の「外部トリガーでアプリのバックグラウンド処理を起動したい」という設計全般に該当する。通知バー表示が不要な短時間処理でも、Foreground Service化では解決しない点に注意（Foreground Serviceも同じ制限の対象）。

---

## 2026-07-16: adbテスト中のクラッシュループでブロードキャストが黙って無視される

- **エラー内容**: 同一アプリが短時間に複数回クラッシュすると、それ以降`adb shell am broadcast`で明示的Intentを送っても対象アプリのプロセスが起動されず、ブロードキャストが黙って失敗する。`adb shell am force-stop <package>`を挟んでも解消しない。
- **原因**: AndroidのActivityManagerにはクラッシュループ検知（bad process判定）があり、一定時間内に繰り返しクラッシュしたプロセスは一時的に起動をブロックされる。この状態は`dumpsys activity broadcasts history`で該当ブロードキャストが`FAILURE ... reason: remote app`として記録されることで確認できる。`force-stop`はプロセスを止めるだけで、このquarantine状態自体はクリアしない。
- **解決方法**: `adb shell am start -n <package>/<Activity>`でアプリを一度明示的に起動する（ユーザー操作によるlaunchとして扱われ、quarantineが解除される）。その後は通常どおりbroadcastが配信される。
- **備考**: BroadcastReceiver/Service起動系の実装をadbで繰り返しテスト・デバッグする際に頻発しうる。「ブロードキャストを送ったのに何も起きない」場合、まずこのクラッシュループ状態を疑い、`dumpsys activity broadcasts history`で`reason: remote app`が出ていないか確認するとよい。

---

## 2026-07-21: バッテリー最適化によるバックグラウンドネットワーク遮断でUnknownHostExceptionが多発する

- **エラー内容**: Taskerからのバックグラウンド起動(WorkManager経由)のたびにDNS解決失敗・接続断・タイムアウトが発生。Wi-Fi/回線自体は正常で、adb shellからのping/DNS解決は成功するため端末のネットワークには問題がない
- **原因**: `adb shell dumpsys netpolicy`で該当UIDを確認すると`blocked_state={blocked=APP_BACKGROUND, allowed=NONE, effective=APP_BACKGROUND}`となっており、バッテリー最適化(App Standby Bucket=RARE等)によりバックグラウンド実行中のアプリの通信自体がOSのファイアウォールで遮断されていた。外部トリガー(Tasker等)でしか起動されずアプリのUIをユーザーが直接開く頻度が低い場合に起きやすい
- **解決方法**: 端末の「設定→アプリ→対象アプリ→バッテリー→バッテリーの最適化」で「制限なし」に設定する。adbから同等の効果を検証するには`adb shell dumpsys deviceidle whitelist +<パッケージ名>`を実行し、`dumpsys netpolicy`のblocked_state.effectiveがNONEに変わることで確認できる
- **備考**: `dumpsys netpolicy | grep -A1 "UID=<uid>"`で該当UIDの状態を確認できる(UIDは`adb shell pm list packages -U <パッケージ名>`で取得)。端末側の設定でありOS更新等でリセットされる可能性がある

---

## 2026-07-21: adb shell am broadcastの暗黙的Intentがバックグラウンド実行制限でポリシー却下される

- **エラー内容**: `adb shell am broadcast -a <custom.action>`（コンポーネント未指定の暗黙的Intent）を送っても、マニフェスト登録済み（`exported="true"`）のBroadcastReceiverに全く届かない。`am broadcast`コマンド自体は`Broadcast completed: result=0`で正常終了するため、一見成功したように見える
- **原因**: `adb shell dumpsys activity broadcasts <package>`の履歴で確認すると、`reason: skipped by policy at enqueue: Background execution not allowed: receiving Intent {...} to <package>/<Receiver>`として、配信自体がOSレベルでポリシー却下されていた。カスタムアクションの暗黙的Intentは、対象アプリがバックグラウンド状態の場合に配信が拒否されることがある。一方、コンポーネントを明示指定した明示的Intentはこの制限を受けない
- **解決方法**: `adb shell am broadcast -n <package>/<component> -a <custom.action>`のように`-n`でコンポーネントを明示指定して送る。実機のTasker等の自動化アプリは内部的に明示的Intentを使っているため、この制限に引っかからず正常に届く
- **備考**: 「adb broadcastが成功したはずなのにレシーバーが全く動いた形跡がない」場合、まず`dumpsys activity broadcasts <package>`で`skipped by policy`が出ていないか確認する。Log.i/Log.e等のlogcat出力も、そもそもレシーバーに到達していなければ一切残らない点に注意（ログが全く無いこと自体が「配信されていない」ことの強い手がかりになる）
