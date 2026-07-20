# Android ロギング Tips

## 2026-07-20: Android Log.getStackTraceString()はUnknownHostException（を含む例外チェーン）で空文字を返す

- **エラー内容**: `Log.e(tag, message, throwable)` は正常にログ出力されるのに、`Log.getStackTraceString(throwable)` を自前のファイルログ等に書き込むと、その部分だけ完全に空文字になり原因が追えない。
- **原因**: AndroidのLog.getStackTraceString()実装は、渡された例外またはその原因(cause)チェーンのいずれかが`java.net.UnknownHostException`だった場合、意図的に空文字列を返す仕様（「ネットワーク不通時のログスパムを減らすため」という設計）。ドキュメント化されていないが既知のAOSP挙動。
- **解決方法**: 空のスタックトレースを見たら「例外が発生しなかった」ではなく「UnknownHostException（名前解決失敗＝DNS/ネットワーク不通）の可能性」を疑う。原因を確実に特定したい場合は`Log.getStackTraceString()`に頼らず、`throwable.javaClass.simpleName`や`throwable.toString()`を別途明示的にログへ含める。
- **備考**: 今回はTasker連携アプリ（DocomoMailGuardAndroid）のIMAP接続失敗ログで発見。UnknownHostExceptionはリトライ対象外の例外だったため、この挙動がなければ即座に気づけた。
