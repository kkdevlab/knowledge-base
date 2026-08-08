# Python Windows Encoding Tips

Windows環境でPythonのファイル読み書きに関する文字コードの知見。

---

## 2026-08-08: WindowsでPythonの`Path.read_text()`がUTF-8ファイルの読み込みに失敗する（cp932デコードエラー）

- **エラー内容**: `UnicodeDecodeError: 'cp932' codec can't decode byte 0x85 in position 78: illegal multibyte sequence`
- **原因**: `pathlib.Path.read_text()`はエンコーディング未指定の場合、OSのロケール既定コードページ（日本語Windowsだとcp932）を使う。UTF-8で書かれたファイル（日本語Markdown等）を読むと文字化け・デコードエラーになる
- **解決方法**: 環境変数`PYTHONUTF8=1`を付けて実行する（PEP 540 UTF-8モード）。例: `PYTHONUTF8=1 python script.py`。スクリプト側を修正できるなら`Path.read_text(encoding="utf-8")`のように明示指定する方がより確実
- **備考**: `open(path, encoding="utf-8")`は同様の問題を起こさない。他人が書いた/配布されたスクリプト（`encoding`引数なしの`read_text()`を使うもの）をWindowsで動かす際に一般的に発生しうる
