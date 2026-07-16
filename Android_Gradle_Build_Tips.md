# Android / Gradle ビルドТips

## 2026-07-16: コマンドラインでの`gradlew`ビルド時にJAVA_HOMEが必要

- **エラー内容**: `ERROR: JAVA_HOME is not set and no 'java' command could be found in your PATH.`
- **原因**: `gradlew.bat`をコマンドライン（PowerShell等）から直接実行すると、Android Studio内蔵のJBR（JetBrains Runtime）を自動では使わないため、JAVA_HOMEが未設定だと失敗する。
- **解決方法**: `$env:JAVA_HOME = "C:\Program Files\Android\Android Studio\jbr"` を設定してから`gradlew`を実行する（Windows・Android Studio標準インストールの場合のパス）。
- **備考**: Android StudioのGUI（Run/Buildボタン）から実行する場合はIDEが自動でJDKを解決するため、この設定は不要。CLIから`gradlew assembleDebug`等を直接叩く場合のみ必要になる。
