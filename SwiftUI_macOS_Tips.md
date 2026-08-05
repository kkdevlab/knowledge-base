# SwiftUI macOS Tips

SwiftUIでmacOSアプリを開発する際に発生したエラー・挙動の癖と対処法をまとめる。

---

## 1. DatePickerの.wheelスタイルはmacOSで利用できない

### 症状

```swift
DatePicker("時刻", selection: $time, displayedComponents: .hourAndMinute)
    .datePickerStyle(.wheel)
```

macOSターゲットでビルドすると以下のコンパイルエラーになる。

```
error: 'wheel' is unavailable in macOS
```

### 原因

`.wheel`スタイル（iOSの時計アプリのようなスクロールピッカー）はiOS専用で、macOSのSwiftUIには実装されていない。

### 解決方法

macOSでは`.field`（テキスト入力のみ）または`.stepperField`（テキスト＋上下ボタンが一体化したコントロール）を使う。

```swift
.datePickerStyle(.stepperField)  // または .field
```

---

## 2. DatePicker(.stepperField)に.frame(height:)を指定すると内部テキストが下寄りになる

### 症状

`.stepperField`スタイルのDatePickerに`.frame(width:height:)`で高さを強制指定すると、灰色の枠内でテキストが下寄りに表示され、上下中央に来ない。横幅を変えても改善しない。また`.font()`でフォントサイズを変えても見た目の大きさが変わらない。

### 原因

`.stepperField`はAppKitのネイティブ複合コントロール（テキストフィールド＋インクリメント/デクリメントボタンが一体化）をSwiftUIにブリッジしたもので、外部から`.frame(height:)`のようなサイズ強制を行うと、内部のAppKitレイアウトが正しく追従せず崩れる。`.font()`も内部コントロールに伝播しないことがある。

### 解決方法

複合コントロールに頼らず、`.field`（テキスト部分のみ）と独立した`Stepper`（上下ボタン）を自分で`HStack`に組み合わせる構成に変更する。

```swift
HStack(spacing: 10) {
    DatePicker("時刻", selection: $selectedTime, displayedComponents: .hourAndMinute)
        .datePickerStyle(.field)
        .labelsHidden()

    Stepper("", onIncrement: { adjustMinute(by: 1) },
                onDecrement: { adjustMinute(by: -1) })
        .labelsHidden()
}
```

視覚的に拡大したい場合は`.font()`ではなく`.scaleEffect()`を使う（`.field`も`.font()`を無視する場合があるため）。

```swift
.datePickerStyle(.field)
.font(.system(size: 20, weight: .medium))
.scaleEffect(1.5)
```

### 備考

ネイティブAppKit/UIKitブリッジ系のSwiftUIコントロール全般に言える傾向として、外部からの強制サイズ指定に弱く内部崩れを起こしやすい。複合コントロールで思い通りのレイアウトにならない場合は、単純な部品に分解して自前でレイアウトを組む方が確実。

---

## 3. Xcode新規macOSアプリはApp Sandboxが有効なままだと他アプリへの自動化命令(Apple Events)が失敗する

### 症状

`NSAppleScript`や`osascript`経由で`tell application "System Events" to ...`のような他アプリを操作する命令を実行しても、権限確認ダイアログすら出ずに黙って失敗する（何も起きない）。

### 原因

Xcodeで新規作成したmacOSアプリはデフォルトで`ENABLE_APP_SANDBOX = YES`（Signing & CapabilitiesのApp Sandbox）が有効になっている。サンドボックス下ではAutomation（Apple Events送信）が既定で拒否される。

### 解決方法

個人利用・配布しないアプリであれば、Xcodeの`Signing & Capabilities`タブでApp Sandbox capability自体を削除する（`project.pbxproj`上は`ENABLE_APP_SANDBOX = NO`になる）。

```
ENABLE_APP_SANDBOX = NO;
```

### 備考

App Store配布する場合はSandboxは必須要件のため外せない。その場合は`com.apple.security.automation.apple-events`entitlementと、対象アプリのbundle identifierを許可リストに追加した上で、Info.plistに`NSAppleEventsUsageDescription`（またはXcodeのTarget > Info タブで「Privacy - AppleEvents Sending Usage Description」を追加）が必要になる。

---

## 4. XcodeのIssue Navigatorに実際のビルドとは無関係な古いエラーが残り続ける

### 症状

`xcodebuild`でのコマンドラインビルドは`BUILD SUCCEEDED`になっているにもかかわらず、Xcode左側のIssue Navigatorには、すでに修正したはずの古いエラー（例: 削除済みの`.wheel`指定に対するエラー）が赤字で表示され続ける。

### 原因

XcodeのSwiftUIプレビュー機能は、アプリ本体のビルドとは別のコンパイルパイプラインを持っており、そちらの解析結果が古いままキャッシュされることがある。

### 解決方法

`Product` → `Clean Build Folder`（`Cmd+Shift+K`）でキャッシュをクリアする。

### 備考

エラーが実際のビルドに影響しているか判断する際は、Xcodeの表示を鵜呑みにせず`xcodebuild -project <proj> -scheme <scheme> -destination 'platform=macOS' build`をターミナルで直接実行し、実際の`error:`行の有無で判断するのが確実。
