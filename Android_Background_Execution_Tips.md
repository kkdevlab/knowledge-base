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
