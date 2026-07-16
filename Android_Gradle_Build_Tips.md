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
