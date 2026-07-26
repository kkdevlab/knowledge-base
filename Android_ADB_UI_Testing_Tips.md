# Android / adb 実機UIテスト Tips

## 2026-07-17: スクリーンショットの表示座標をそのままadb tapに渡すとずれる

- **エラー内容**: `adb shell screencap`で撮ったスクリーンショットをRead toolで開くと画像が縮小表示され、「original 1080x2340, displayed at 923x2000. Multiply coordinates by 1.17 to map to original image.」のような注記が付く。これを見落として表示画像上で読み取った座標をそのまま`adb shell input tap`に渡すと、意図した要素からずれた位置をタップしてしまう（実例: 「ルール追加」ボタンを狙ったつもりが、その少し上にあるEditTextに毎回フォーカスしてしまった）。
- **原因**: Read toolは大きい画像を自動的に縮小して表示するため、目視で読み取れる座標はdisplayed基準。一方`adb shell input tap`が要求するのはデバイスのオリジナル解像度座標。
- **解決方法**: Read結果に付くスケール係数（例: 1.17）を、タップしたい座標のx・y両方に必ず掛けてから`adb shell input tap`に渡す。
- **備考**: 縦横で同一のスケール係数になることが多いが、念のため注記に書かれた係数をそのまま使うこと（画像処理側で丸めが入る場合がある）。
- **再発(2026-07-18)**: このルールを一度記録済みと知っていても、長時間セッションで何十枚もスクリーンショットを撮り続けていると、確認を省略して表示座標をそのままタップに渡してしまい、意図と違う項目（別のタスク等）を誤って開いてしまうことがあった。**「知っている」ことと「毎回のタップ前に実際に計算を適用する」ことは別**。特にリスト項目のように隣接要素が近接している場面では、tapの直前に必ずスケール適用後の座標を明示的に計算してから実行する習慣を徹底する。

## 2026-07-17: `adb shell input text`がダイアログ内EditTextへの反映前にOKタップが実行され空文字のまま確定する

- **エラー内容**: AlertDialog等のカスタムビュー内のEditTextに対して「タップでフォーカス→`input text`で文字列送信→直後にOKボタンをタップ」という順で実行すると、実際にはテキストが反映されずEditTextが空のまま処理が進んでしまう（ログ上もaccessibilityイベントのelementTextがhintのまま変化しないことで確認できる）。
- **原因**: タップ後のIME（ソフトウェアキーボード）起動・フォーカス確定に時間がかかり、直後に送った`input text`が間に合わないことがある。特にダイアログのように後から動的にinflateされるビューで起きやすい。
- **解決方法**: EditTextタップ後に1秒程度`Start-Sleep`を挟んでから`input text`を実行し、さらに`screencap`で実際に入力された文字列を目視確認してからOKボタンをタップする。
- **備考**: 通常のActivity直下のEditTextでは比較的発生しにくく、ダイアログ内など画面遷移直後のフォーカス確定が絡む場面で再現しやすい。

## 2026-07-26: `adb shell input text`が日本語IME(Gboard)のローマ字変換に横取りされて文字化けする

- **エラー内容**: EditTextに`adb shell input text "email@example.com"`のようなASCII文字列を送ったところ、実際に入力された内容が`えまいｌ＠...`のような全角ひらがな・カタカナに変換されていた（UI dumpの`text`属性で確認）。
- **原因**: 端末のデフォルトIMEが日本語入力（Gboard等のローマ字変換モード）になっていると、`input text`が送るキー入力がIMEの変換エンジンを経由し、ASCII文字列がローマ字としてかな変換されてしまう。
- **解決方法**: 一時的に変換を行わないIME（Tasker同梱のIME等、自動化目的で文字を素通しするもの）へ切り替えてから入力する。

  ```sh
  # 例: net.dinglisch.android.taskerm/com.joaomgcd.taskerm.keyboard.InputMethodServiceTasker
  adb shell ime enable <パッケージ>/<IMEクラス名>
  adb shell ime set <パッケージ>/<IMEクラス名>
  # ここでinput text等を実行
  # 例: com.google.android.inputmethod.latin/com.android.inputmethod.latin.LatinIME
  adb shell ime set <元のIME>
  adb shell ime disable <一時IME>
  ```

  `adb shell ime list -s`で現在有効なIME一覧、`ime list -a`で端末に
  インストール済みの全IME（Tasker等サードパーティアプリが提供するIMEも含む）を確認できる。
- **備考**: 既存のテキストが残っている場合、切り替え後にまず`input keyevent 123`（MOVE_END）→`input keyevent 67`（DEL）を必要文字数以上繰り返してクリアしてから入力し直すと確実。UI dump（`uiautomator dump`）の対象EditTextの`text`属性を見れば、入力後すぐにASCIIのまま反映されたか検証できる。

## 2026-07-17: 実機テスト中に画面がスリープしてadb操作が空振りする

- **エラー内容**: `adb shell screencap`で撮った画像が真っ黒になり、直前に送ったタップ操作がすべて無効になっていた。
- **原因**: 実機の画面消灯タイマーは短く設定されていることが多く、ユーザーとのやり取りやコマンド間の待機で数十秒〜数分空くと画面が自動的にスリープ・ロックする。
- **解決方法**: 操作再開前に`adb shell input keyevent KEYCODE_WAKEUP`でスクリーンを起こし、続けて`adb shell input swipe <画面下座標> <画面上座標>`でスワイプロックを解除してから操作を続ける。screencapで得た画像のファイルサイズが極端に小さい（同一デバイスでのロック画面キャプチャと同じサイズ）場合は、スリープ/ロック画面である可能性が高いと判断できる。
- **備考**: PIN/パターン等のセキュリティロックが掛かっている端末ではスワイプだけでは解除できないため、その場合はユーザーに手動解除を依頼する必要がある。
