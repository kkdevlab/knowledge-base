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

## 2026-07-16: EncryptedSharedPreferences（androidx.security:security-crypto）は非推奨・安定版なし

- **エラー内容**: 該当なし（要件定義書に明記された保存方式が実装時点で技術的に陳腐化していると判明したケース）
- **原因**: `androidx.security:security-crypto`は2025年4月リリースの`1.1.0-alpha07`でGoogleにより非推奨化されており、2026年7月時点でも安定版（stable）が存在しない。Google公式の推奨移行先は「Jetpack DataStore + Tink」。
- **解決方法**: `androidx.datastore:datastore-preferences`（2026年7月時点stable: 1.2.1）でPreferencesを保存し、値は`com.google.crypto.tink:tink-android`（同stable: 1.22.0）のAEADで暗号化してから書き込む。
- **備考**: 新規プロジェクトの要件定義・設計時にEncryptedSharedPreferencesを指定する前に、非推奨化状況をWeb検索で確認すること。

## 2026-07-16: Tink AndroidKeysetManagerの現行API（KeyTemplates.get、getPrimitive(Class)は非推奨だが動作する）

- **エラー内容**: `compileDebugKotlin`で`'fun <P : Any!> getPrimitive(targetClassObject: Class<P!>!): P!' is deprecated.`という警告（エラーではない）
- **原因**: Tink 1.22.0時点で`KeysetHandle.getPrimitive(Class)`は非推奨（代替は`getPrimitive(Configuration, Class)`）だが、後方互換のため引き続き機能する。また鍵テンプレート取得も旧`com.google.crypto.tink.aead.AeadKeyTemplates`ではなく`com.google.crypto.tink.KeyTemplates.get("AES256_GCM")`を使うのが現行API。
- **解決方法**:

  ```kotlin
  AeadConfig.register()
  val keysetManager = AndroidKeysetManager.Builder()
      .withSharedPref(context, keysetName, prefFileName)
      .withKeyTemplate(KeyTemplates.get("AES256_GCM"))
      .withMasterKeyUri("android-keystore://$alias")
      .build()
  val aead = keysetManager.keysetHandle.getPrimitive(Aead::class.java)
  ```

- **備考**: 非推奨警告はビルドを失敗させないため無視して問題ない。Android Keystore連携は実機（API 23+）でのみ動作し、JVM単体テストでは検証できない。

## 2026-07-16: Kotlin DSL（build.gradle.kts）の`defaultConfig{}`ブロック内で`java.text.*`等の完全修飾参照が`Unresolved reference`になる

- **エラー内容**: `e: Unresolved reference 'text'.`（`java.text.SimpleDateFormat`の`text`部分）、`e: Unresolved reference 'util'.`（`java.util.Date`の`util`部分）
- **原因**: Android Gradle PluginのDSLブロック（`defaultConfig{}`等）内では、トップレベルの`java`パッケージ参照がAGP側のプロパティ/レシーバーに解決されてしまい、`java.text.X`のような完全修飾参照が期待通りに機能しないことがある
- **解決方法**: ファイル冒頭で`import java.text.SimpleDateFormat`・`import java.util.Date`のように明示的にimportし、ブロック内では`SimpleDateFormat(...)`のように完全修飾せず使う
- **備考**: `buildConfigField`で動的な値（ビルド時刻等）を埋め込む場合など、`defaultConfig{}`内でJava標準クラスを直接呼びたい場面で発生しやすい

## 2026-08-16: GradleはJAVA_HOMEと実際にビルドを実行するJDKが異なることがある

- **内容**: `gradlew`起動時のJAVA_HOME（ラウンチャー用JVM）と、Gradleが実際にタスクを実行するJDKは別。GradleはGradle自身がtoolchain用に`~/.gradle/jdks/`配下へ自動プロビジョニングしたJDK（例: Eclipse Temurin）を優先使用し、JAVA_HOMEを無視することがある
- **確認方法**: procmonでビルド実行中のjava.exeの実プロセスパスを確認するか、`--info`ログの`Received JVM installation metadata`行で確認できる
- **明示指定する方法**: `-Dorg.gradle.java.home=<JDKパス>`をコマンドラインで渡す

## 2026-08-16: AGP 9.1.0 + Windowsで`gradlew clean`が「Unable to delete directory」で決定論的に失敗する既知の不具合

- **エラー内容**: `java.io.IOException: Unable to delete directory 'app\build'` / `Failed to delete some children. This might happen because a process has files open...`。`clean`タスクに限らず`generateDebugBuildConfig`等、Gradleの削除処理全般で発生
- **原因**: procmonで解析した結果、対象ディレクトリは実際には空にもかかわらず、`SetDispositionInformationEx`（`FILE_DISPOSITION_POSIX_SEMANTICS`フラグ）が`CANNOT DELETE`で失敗する。OneDrive・Windows Defender・Gradleデーモン・Gradle VFSファイル監視・JDKバージョン(21/25)はいずれも無関係と切り分け済み。AGPを8.5.2系にダウングレードすると解消するため、AGP 9.1.0（Kotlinビルトインサポート等の新機能を含む新しいタスク実装）自体のバグと推定される
- **解決方法**: AGP 9.1.0の使用を避け、AGP 8.x系（Kotlin Gradle Pluginを明示的に追加する構成）にダウングレードする。または将来のAGP修正版リリースを待つ
- **備考**: Gradle公式でもこの種のエラーメッセージ自体が真因を提示できていないことを開発チームが認めている（[gradle/gradle#25984](https://github.com/gradle/gradle/issues/25984)）。JDK単体の`Files.delete()`は正常動作するため（`jshell`のスクリプトファイル末尾に`/exit`を書くとヘッドレス実行できる）、JDK自体の欠陥ではなくGradle/AGP側の問題と判断できる

## 2026-08-16（訂正）: 上記「Unable to delete directory」バグはAGPバージョンとは無関係と判明

- **訂正内容**: 上記エントリで「AGP 9.1.0自体のバグ」と推定したが、後日の追加調査でAGP 8.5.2にダウングレードしても実プロジェクトでは同じエラーが再現することを確認。AGPバージョンに依存しない、Gradle自身の削除処理（`SetDispositionInformationEx`/`FILE_DISPOSITION_POSIX_SEMANTICS`）特有の既知バグ（gradle/gradle#25984相当）である可能性が高い
- **切り分け結果**: CLI（`gradlew`）・IDE（Android StudioのUI「Clean Project」）どちらでも再現。PC再起動後も再現。通常の差分ビルド（コード修正→Run App）には影響せず、`clean`を明示的に呼んだ場合のみ発生
- **実務上の回避策**: `gradlew clean`やIDEの「Clean Project」を使わず、Windowsエクスプローラーから`app\build`フォルダを手動削除する（成功を確認済み）。AGPダウングレードは対処にならない上、後述の別問題（compileSdk上限・依存ライブラリ矛盾）を誘発するだけなので推奨しない

## 2026-08-16: Kotlin Gradle Plugin 1.9.24はJDK 25（バージョン文字列 "25.0.2"）を解析できない

- **エラー内容**: `java.lang.IllegalArgumentException: 25.0.2`（`org.jetbrains.kotlin.com.intellij.util.lang.JavaVersion.parse`）
- **原因**: Kotlin 1.9.24のバンドルJavaVersionパーサーが、JDK 25系のバージョン文字列形式に対応していない
- **解決方法**: Gradle実行時のJDK（`JAVA_HOME`または`-Dorg.gradle.java.home`）をJDK21系に切り替える。Gradleが自動プロビジョニングした`%USERPROFILE%\.gradle\jdks\eclipse_adoptium-21-amd64-windows.*`等が使える
- **備考**: Android Studio付属JBRの既定バージョンが25系に上がったことで、古いKotlin Gradle Pluginとの組み合わせで顕在化する

## 2026-08-16: AGP 8.5.2はcompileSdk 34が上限、依存ライブラリ側の要求バージョンと衝突しうる

- **内容**: AGP 8.5.2はcompileSdk 34までしかテストされておらず、compileSdk 35/36を要求する構成では`checkDebugAarMetadata`等のタスクが失敗する。特に`androidx.work:work-runtime:2.10.0`はcompileSdk 35以上を要求するため、AGP8.5.2との組み合わせは非両立
- **教訓**: AGPをダウングレードする際は、compileSdk自体の対応上限だけでなく、依存ライブラリ側が要求する最低compileSdkバージョンも事前に確認する必要がある
