# Windows UI Automation Tips

C#から他プロセスのUIを読み取り・操作するUI Automation (IUIAutomation) 利用時の知見。

---

## 2026-08-02: WinUI3/非WPFプロジェクトでのUI Automationクライアント利用は`Interop.UIAutomationClient`パッケージが軽量

- **内容**: `System.Windows.Automation`（管理ラッパー）はWPF参照アセンブリ扱いのため、WinUI3など非WPFプロジェクトで使うには`UseWPF=true`が必要になり余計な依存が増える。`Interop.UIAutomationClient`（NuGet）は`IUIAutomation`等をピュアなCOM相互運用として提供し、WPF依存なしで使える。AutoIt/WinAppDriver等でも使われている
- **注意**: 型は`Interop.UIAutomationClient`名前空間。パターンIDは`UIA_PatternIds.UIA_TextPatternId`等の定数として提供される

---

## 2026-08-02: UI Automationの呼び出しは専用STAスレッドで行う

- **内容**: 多くのUIAプロバイダーは呼び出し元スレッドがSTA (Single-Threaded Apartment) であることを前提とする。`Task.Run`（既定でMTAのThreadPool）から直接呼ぶと失敗・不安定になりうる
- **解決方法**: `Thread`を生成し`SetApartmentState(ApartmentState.STA)`を設定してから呼び出す。呼び出し頻度が低ければ都度スレッドを起こして`TaskCompletionSource`で結果を待つ方式で十分

---

## 2026-08-02: TextPatternの対応状況・読み取り内容の信頼性はアプリ依存。確認失敗時に副作用のある操作を無条件リトライすると危険

- **内容**: UI AutomationのTextPattern対応はアプリによってまちまち。Windows 11標準メモ帳のようなネイティブコントロールは概ね安定して対応するが、VS Code（Monacoエディタ、Electron）はTextPatternの取得自体は成功することがあっても、読み取れる内容が実際の編集内容とリアルタイムに同期していないことがある
- **危険な実装パターン**: 「UI Automationで結果を確認し、反映されていなければ操作を再送する」という設計は、確認そのものが信頼できないアプリでは「実際には毎回成功しているのに確認だけ失敗し続け、副作用のある操作（Ctrl+Vによる貼り付け等）が何度も実行されて内容が重複する」事故を起こす
- **対策**: 削除のように多少の過不足が軽微な操作はリトライしてよいが、貼り付け・送信のように再実行が実害（重複）を生む操作は、確認できなくても再送せず1回で諦める設計にする
