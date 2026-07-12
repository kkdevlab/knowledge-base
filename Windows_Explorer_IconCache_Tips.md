# Windows Explorer / アイコンキャッシュ Tips

Windows のアイコンキャッシュクリア・explorer.exe 再起動に関する知見まとめ。

---

## ie4uinit.exe が見つからない（Windows 11）

### 症状

- `ie4uinit.exe -ClearIconCache` を実行すると「認識されません」エラーになる

### 原因

`ie4uinit.exe` は一部の Windows 11 環境では `System32`・`SysWOW64` のどちらにも存在しない（バージョン・エディションによる差異）。

### 解決方法

アイコンキャッシュのデータベースファイルを直接削除する。

```powershell
Get-Process explorer -ErrorAction SilentlyContinue | Stop-Process -Force
Start-Sleep -Seconds 1
Remove-Item "$env:LOCALAPPDATA\Microsoft\Windows\Explorer\iconcache*.db" -Force -ErrorAction SilentlyContinue
Remove-Item "$env:LOCALAPPDATA\IconCache.db" -Force -ErrorAction SilentlyContinue
Start-Process explorer.exe
```

---

## Stop-Process -Force で explorer.exe を再起動するとタスクバーが消えることがある

### 症状

- `Stop-Process -Name explorer -Force` の後に `Start-Process explorer.exe` を実行すると、プロセス自体は起動する（`Get-Process explorer` で確認できる）が、タスクバー・デスクトップアイコンが表示されない
- このとき explorer.exe はファイルエクスプローラーの「ホーム」ウィンドウ（`CabinetWClass`）だけを開いており、シェル（タスクバー`Shell_TrayWnd`・デスクトップ`Progman`）としては再初期化されていない

### 診断方法

P/Invoke `FindWindow` で該当ウィンドウの有無を確認する。

```powershell
Add-Type @"
using System;
using System.Runtime.InteropServices;
public class Win32 {
    [DllImport("user32.dll")]
    public static extern IntPtr FindWindow(string lpClassName, string lpWindowName);
}
"@
[Win32]::FindWindow("Shell_TrayWnd", $null)  # 0 ならタスクバー未生成
[Win32]::FindWindow("Progman", $null)         # 0 ならデスクトップ未生成
```

### 解決方法

中途半端に `Start-Process explorer.exe` を重ねて呼ばない。既存の explorer.exe プロセスを完全に終了させてから、改めて1回だけ起動し直す。

```powershell
Get-Process explorer -ErrorAction SilentlyContinue | Stop-Process -Force
Start-Sleep -Seconds 2
Start-Process "$env:WINDIR\explorer.exe"
```

- 1回目の再起動で失敗しても数秒待てば直ることは稀（プロセス自体は「起動中・応答あり」の状態のままシェルが生成されないことがある）。**待つより、完全終了→起動し直しをやり直す方が確実**
- アイコンキャッシュクリアなど explorer.exe の強制終了を伴う操作を自動化する際は、この現象が起こりうることを前提に、再起動後は `Shell_TrayWnd` の存在を確認してから完了報告する
