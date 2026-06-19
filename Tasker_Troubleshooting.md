# Tasker トラブルシュート

Tasker 利用中に遭遇したエラー・警告の解決記録（汎用的な挙動・不具合）。
同じ問題が発生した場合はここを参照する。プロジェクト固有のトラブルは各プロジェクトの `CLAUDE.md` を参照。

---

## 2026-03-23: Input Dialog の Default フィールドで二段階展開（%%var）が動作する

- **確認内容**: `%%par1`（par1にグローバル変数名の文字列が入っている）を Input Dialog の Default フィールドに指定
- **結果**: 正常に動作。%par1 の値をグローバル変数名として解釈し、その値を Default に表示する
- **活用場面**: Edit タスクで現在のグローバル変数値をダイアログのデフォルト値として表示する場合、中間変数（%current_val等）は不要

---

## 2026-02-13: Tasker で %id が変数名として使用不可

- **エラー内容**: `Error: %id: must be a variable or array name.`
- **原因**: Tasker のローカル変数名は3文字以上必要（%id は2文字で短すぎる）
- **解決方法**: `%target_id` など3文字以上の名前に変更する
- **備考**: Tasker のローカル変数命名ルール: 全小文字、3文字以上

---

## 2026-05-29: 6.7.3-beta 更新後に %DATE のフォーマットが変わる

- **症状**: `%DATE` が `22-04-2026` → `4-22-26` のように変化する。再起動で一時的に直るが再発する
- **原因**: ベータ更新によるアプリ言語設定のリセット
- **解決方法**: Android 設定 → アプリ → Tasker → 言語 → 正しいロケール（例: English (Belgium)）を選択
  - Tasker の設定（Preferences）ではなく Android のシステム設定から変更すること
- **備考**: joaomgcd 確認済み。Android 13+ の System language picker とも連携

---

## 2026-05-29: Scene V2 ランドスケープ回転時にシーンサイズが崩れる

- **症状**: ポートレートで起動したシーンをランドスケープに回転すると、サイズ・位置が崩れる
- **原因**: Show Scene V2 アクションの Width / Height をピクセル値で指定していると回転時に対応できない
- **解決方法**: Width / Height はパーセント指定（例: 100%）にする。ピクセル値は使わない
- **備考**: joaomgcd 回答で確定（6.7.3-beta で修正済みのバグも含む）

---

## 2026-05-29: Scenes V2 インタラクションが 6.7.3-beta 更新後に動作しない

- **症状**: 既存シーンのボタンクリックなどのインタラクションが動作しない
- **原因**: 6.7.3-beta で Event Handler の仕組みが全面刷新された（Interactable モディファイア方式廃止）
- **解決方法**: 各コンポーネントを開いて新しい Event Handler 形式で再設定する
  - ボタンクリック → Event Handler: Set Variable or Run Task
  - テキスト値取得 → Output to Variable
  - シーンを閉じる → Dismiss Layout
- **備考**: Result Binding も廃止。`ScenesV2_Dev_Guide.md` セクション0 を参照

---

## 2026-02-14: Tasker アクションコード code 43 = Else（Stop ではない）

- **エラー内容**: code 43 を Stop と認識してレビュー・疑似コード作成していた
- **原因**: Tasker_Action_Codes.md および外部参照元に code 43 = Stop と記載されていたが、実際は Else
- **解決方法**: Tasker_Action_Codes.md を修正（code 43 = Else）
- **備考**: 外部参照元（Taskomater/Tasker-XML-Info）の情報が不正確な場合がある。実機で確認すること
