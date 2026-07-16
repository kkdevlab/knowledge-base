# Android / Gradle ビルドТips

## 2026-07-16: コマンドラインでの`gradlew`ビルド時にJAVA_HOMEが必要

- **エラー内容**: `ERROR: JAVA_HOME is not set and no 'java' command could be found in your PATH.`
- **原因**: `gradlew.bat`をコマンドライン（PowerShell等）から直接実行すると、Android Studio内蔵のJBR（JetBrains Runtime）を自動では使わないため、JAVA_HOMEが未設定だと失敗する。
- **解決方法**: `$env:JAVA_HOME = "C:\Program Files\Android\Android Studio\jbr"` を設定してから`gradlew`を実行する（Windows・Android Studio標準インストールの場合のパス）。
- **備考**: Android StudioのGUI（Run/Buildボタン）から実行する場合はIDEが自動でJDKを解決するため、この設定は不要。CLIから`gradlew assembleDebug`等を直接叩く場合のみ必要になる。

## 2026-07-16: ローカルUnitTest(JVM)でorg.json（JSONObject等）を使うとスタブ例外になる

- **エラー内容**: `android.jar`のorg.jsonクラスはスタブ実装のため、ローカルUnitTest（`test`ソースセット、Robolectric等を使わない場合）でJSONObjectを使うと`RuntimeException: Method ... not mocked`が発生する（実際に発生させたわけではなく、既知のAndroidの罠として事前に回避）。
- **原因**: `org.json`はAndroidランタイムには標準搭載されているが、ローカルJVM上のUnitTestが参照する`android.jar`側は実体を持たないスタブになっている。
- **解決方法**: `testImplementation("org.json:json:<version>")`のように実体jarをテストのみに追加する（実装依存には加えない。実行時はAndroid標準搭載のもので上書きされる）。
- **備考**: 同様の罠は`android.util.Base64`等でも起きるため、その場合は`java.util.Base64`（Java標準、Android API 26+で利用可）等プラットフォーム非依存の代替を使うとローカルUnitTestでもそのまま動く。
