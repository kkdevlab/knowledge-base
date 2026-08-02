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

---

## 2026-08-02: KEYEVENTF_UNICODEの大量一括送信でTSFベースのコントロールが文字化けする

- **エラー内容**: SendInputでKEYEVENTF_UNICODEを使い文字列を1文字ずつ間隔ゼロで大量送信すると、Windows 11標準メモ帳（TSFベースのテキストコントロール）で文字の順序が入れ替わって表示される（例: "Project"が"Proje"+"ct"のように分断・入れ替わる）。VS Code・秀丸など単純な編集コントロールでは発生しない
- **原因**: TSF (Text Services Framework) は文字入力を非同期・別スレッドで組み立てる。実タイピングでは自然に生じるキー間の時間差がSendInputの一括送信ではゼロになるため、TSF側の内部処理が追いつかず、受信順と反映順がずれる
- **解決方法**: 大量の文字をKEYEVENTF_UNICODEで逐次送るのではなく、クリップボードに文字列を書き込んでCtrl+Vで貼り付ける方式にする。対象アプリは1回のペースト操作として受け取るため、文字ごとの処理順の問題を回避できる

---

## 2026-08-02: WH_KEYBOARD_LLでキー押下を抑止する際はキー離しイベントも合わせて抑止する

- **エラー内容**: グローバルキーボードフックのコールバックで特定のキー押下（WM_KEYDOWN）だけを握りつぶし（CallNextHookExを呼ばず`(IntPtr)1`を返す）、対象アプリへ配信しないようにした
- **原因**: WM_KEYDOWNだけ抑止してもWM_KEYUPは素通りしてしまい、対象アプリが「押されていないキーの離しイベント」を受け取ることになる。多くのアプリは無害に無視するが、修飾キー状態の追跡等に影響しうる
- **解決方法**: 押下を抑止した仮想キーコードを一時的に記録しておき、直後のWM_KEYUP/WM_SYSKEYUPで同じvkCodeが来たらそれも合わせて抑止する
