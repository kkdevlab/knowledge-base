# .NET Build Tips

.NET SDK / MSBuild でのビルドに関する知見まとめ。

---

## ApplicationIcon が exe に埋め込まれない（ビルドは成功・警告0件）

### 症状

- csproj に `<ApplicationIcon>Assets\icon.ico</ApplicationIcon>` を設定して `dotnet build` を実行
- ビルドは「0 個の警告、0 エラー」で成功する
- しかし生成された exe に実際にはアイコンが埋め込まれていない
  - タスクバー・エクスプローラーでデフォルトアイコンのまま
  - `ExtractIconEx`（shell32.dll）で確認するとアイコンハンドル数が 0

### 原因

MSBuild の共有コンパイラサーバー（`VBCSCompiler.exe`）が、**別セッション・別プロジェクトの作業ディレクトリを保持したまま常駐し続けている**ことがある。

.NET のビルドフローは以下の通り:
1. `CoreCompile` ターゲットの `Csc` タスクが `Win32Icon="$(ApplicationIcon)"`（csproj からの相対パス）を渡してコンパイル → アイコンをマネージド DLL の Win32 リソースとして埋め込む
2. `_CreateAppHost` ターゲットの `CreateAppHost` タスクが、その DLL の Win32 リソース（アイコン・バージョン情報等）を native な apphost.exe（実際に実行される exe）にコピーする

ここで `Csc` タスクが `VBCSCompiler.exe`（永続プロセス）にコンパイル要求を委譲する際、そのプロセスのカレントディレクトリが要求元プロジェクトと異なっていると、csproj 内の**相対パス**（`Assets\icon.ico` 等）を解決できず、**エラーも警告も出さずにアイコン埋め込みだけが無言でスキップされる**。

### 解決方法

```
taskkill /F /IM VBCSCompiler.exe
```

でプロセスを終了させてから再ビルドする。新しい `VBCSCompiler.exe` が正しい作業ディレクトリで起動し直され、以降は通常の `dotnet build`（特別なフラグ不要）で正しく埋め込まれる。

### 診断方法（確実な検証手順）

`Icon.ExtractAssociatedIcon()`（System.Drawing）は **Windows のシェルアイコンキャッシュ**を参照するため、実際には埋め込まれていなくても古いキャッシュ結果を返すことがあり、当てにならない。

確実に検証するには `ExtractIconEx`（shell32.dll）を直接 P/Invoke で呼び、返るハンドル数を見る:

```csharp
[DllImport("shell32.dll", CharSet = CharSet.Auto)]
static extern uint ExtractIconEx(string file, int index, IntPtr[] large, IntPtr[] small, uint count);
```

`count == 0` ならアイコンは埋め込まれていない。

### 備考

- `-p:UseSharedCompilation=false` を付けてビルドすれば共有コンパイラを使わずその場しのぎで回避できるが、根本原因（stale なプロセス）は解消しないため推奨しない。プロセスを kill して正しい状態に戻すのが本質的な解決策。
- WPF プロジェクト（`net9.0-windows`, `UseWPF=true`）で確認。WinForms・コンソールアプリ等、apphost を使う .NET Core/5+ プロジェクト全般で起こりうる。
