# Windows SendInput / P/Invoke Tips

C#からWin32の`SendInput`・`ToUnicode`等をP/Invokeで扱う際の落とし穴と対処法。

---

## 2026-08-02: ToUnicode()のP/InvokeでCharSet未指定によりキー→文字変換が機能しない

- **エラー内容**: グローバルキーボードフックでキー→文字変換のためToUnicode()をP/Invokeしたが、正しい文字が取得できなかった
- **原因**: DllImportに`CharSet = CharSet.Unicode`を指定していなかったため、StringBuilder引数がANSIとしてマーシャリングされ、ネイティブ側のUTF-16書き込みと食い違っていた
- **解決方法**: `[DllImport("user32.dll", CharSet = CharSet.Unicode)]`を指定する。StringBuilderを使うWin32 API全般（GetWindowText等）で同様の注意が必要

---

## 2026-08-02: SendInput()がERROR_INVALID_PARAMETER(87)で失敗する

- **エラー内容**: 自前定義のINPUT構造体でSendInput()を呼ぶと毎回ERROR_INVALID_PARAMETER(87)で失敗
- **原因**: ネイティブのINPUT構造体はtype+共用体(MOUSEINPUT/KEYBDINPUT/HARDWAREINPUT)で構成され、共用体の実サイズは最大メンバーのMOUSEINPUT（x64で32バイト）で決まる。KEYBDINPUTだけを定義しSizeを明示しないと、CLRがKEYBDINPUTだけのサイズ（x64で24バイト）を採用し、SendInputへ渡すcbSizeが実際のネイティブサイズ（x64で合計40バイト）と食い違う
- **解決方法**: 共用体に`[StructLayout(LayoutKind.Explicit, Size = 32)]`（x64でのMOUSEINPUTサイズ）を明示するか、MOUSEINPUT/HARDWAREINPUTも含め全メンバーを定義してCLRに正しいサイズを計算させる
- **注意**: x86では共用体サイズが異なる（24バイト）。マルチプラットフォーム対応が必要な場合は`IntPtr.Size`等で分岐するか、全メンバー定義方式にする

---

## 2026-08-02: KEYEVENTF_UNICODEで改行文字(\n)が無視される

- **エラー内容**: SendInputのKEYEVENTF_UNICODEで文字列を1文字ずつ直接入力する際、`\n`が無視され複数行が1行にまとまってしまう
- **原因**: KEYEVENTF_UNICODEは指定したUTF-16コード単位をそのまま「文字」として送るが、多くのテキスト入力コントロールは改行文字を印字可能文字として扱わず無視・除去する
- **解決方法**: 文字列走査時に`\n`を検出したら生文字として送らず、実際のEnterキー(VK_RETURN)のkeydown/keyupイベントとして送る。`\r`はCRLFの対として読み飛ばす
