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

---

## dotnet build が MSB3027/MSB3026 で失敗する（ファイルが別プロセスにロックされている）

### 症状

```
warning MSB3026: "...\apphost.exe" を "bin\...\ProjectName.exe" にコピーできませんでした。1000 ミリ秒以内に 1 回目の再試行を開始します。The process cannot access the file '...\ProjectName.exe' because it is being used by another process. このファイルは "ProjectName (12345)" によってロックされています。
error MSB3027: ...10 回の再試行回数を超えたため、失敗しました。
```

### 原因

そのプロジェクトの exe を**前回起動したまま**（バックグラウンドで実行中・監視ループ中・デバッグ実行中断し忘れ 等）になっており、OS がそのファイルへの書き込みロックを保持している。`dotnet build` は出力先の exe に書き込もうとして失敗する。

### 解決方法

エラーメッセージに含まれるプロセスID（例の場合 `12345`）を使って該当プロセスを終了してから再ビルドする。

```powershell
Get-Process -Id 12345 -ErrorAction SilentlyContinue | Select-Object Id, ProcessName, StartTime
Stop-Process -Id 12345 -Force
```

プロセス名で一括終了してもよい（該当exeが1つしか動いていないことを確認してから）:

```powershell
Get-Process -Name "ProjectName" -ErrorAction SilentlyContinue | Stop-Process -Force
```

### 注意

- 常駐しない設計の一回実行型ツールでも、デバッグ用の長時間監視モード（例: 10分間待機するバックグラウンド処理）を手動起動して放置していると同じ問題が起きる
- ビルド前に `dotnet build` を試してこのエラーが出たら、まず「このexeを自分がテスト用に起動しっぱなしにしていないか」を疑う

---

## タスクバーでアイコンがボケる（icon.icoのサイズ不足）

### 症状

- exeのアイコンはエクスプローラー等では綺麗に見えるが、タスクバーだけ他アプリより粗く/ボケて見える

### 原因

icon.icoに`16/32/48/256px`など一部サイズしか含まれていない場合、Windowsがタスクバー表示用に要求する`20px`/`24px`に一致するフレームが無く、直近の**小さいサイズ（16px）を引き伸ばして**表示してしまう。

### 診断方法

`System.Drawing.Icon(path, w, h)`コンストラクタ（Explorerのアイコン選択とほぼ同等の挙動）で実際に選択されるフレームサイズを確認できる。

```powershell
Add-Type -AssemblyName System.Drawing
$icon = New-Object System.Drawing.Icon($icoPath, 24, 24)
Write-Output "$($icon.Width)x$($icon.Height)"  # 期待値24と異なれば別サイズにフォールバックしている
```

### 解決方法

256px等の高解像度ソースから`16, 20, 24, 32, 40, 48, 64, 96, 128, 256`の10サイズをLanczos等の高品質リサンプリングで生成し、icon.icoに含める（小サイズを拡大するのではなく、大サイズから縮小する）。

---

## WinUI3プロジェクトでdotnet buildがXamlCompiler.exeエラー(終了コード1)で失敗する

- **エラー内容**: `Microsoft.UI.Xaml.Markup.Compiler.interop.targets` 内で `XamlCompiler.exe "obj\...\input.json" "obj\...\output.json"` がコード1で終了、MSB3073エラー
- **原因**: `obj/`配下のXAMLコンパイラ用キャッシュ(input.json/output.json等)が古い/破損した状態になっている
- **解決方法**: `obj/`・`bin/`を削除してクリーンビルドする
  ```powershell
  Remove-Item -Recurse -Force obj, bin
  dotnet build
  ```
- **備考**: コード変更（C#ファイルの編集のみ）とは無関係に発生することがある。ビルドエラーの原因調査より先に一度クリーンビルドを試す価値がある

---

## dotnet build単独実行時、csprojのx64フォールバックが効かずarm64ビルドになる

### 症状
- csprojに `<Platforms>x86;x64;arm64</Platforms>` と、Platform未指定時はx64にするフォールバック
  ```xml
  <Platform Condition="'$(Platform)' == '' or '$(Platform)' == 'AnyCPU'">x64</Platform>
  ```
  を書いていても、`dotnet build` を引数なしで実行すると `bin\arm64\...` に出力される
- **ホストのCPUアーキテクチャとは無関係に発生する**。ARM64機種（Snapdragon搭載Copilot+ PC等）だけでなく、x64機種（AMD Ryzen AI 5 340搭載機）でも同様にarm64が選ばれることを確認済み

### 原因
複数Platform指定のWinUI/.NET SDKプロジェクトで`dotnet build`に明示的な`-p:Platform`を渡さない場合、MSBuildの既定Platform解決処理がcsproj側の「空 or AnyCPUならx64」という条件式より先に働き、`<Platforms>`列挙の中からarm64を選んでしまう。SDK内部のどの処理がこれを行っているかは未特定（ホストアーキテクチャによる分岐ではないことのみ確認済み）。

### 解決方法
ビルド時にPlatformを明示指定する。

```powershell
dotnet build -p:Platform=x64
```

### 備考
- VS CodeのC# Dev Kit拡張機能を使っている場合、ワークスペースを開くたびにバックグラウンドで`Platform`未指定の`dotnet restore`が自動実行され、同じ理屈でarm64の空`bin`/`obj`フォルダが自動生成されることがある（実害はなく`.gitignore`対象なら放置可）
- 他プロジェクトとの出力構成（`bin\x64\...`）を揃えたい場合や、x64前提のツール連携がある場合は、常に`-p:Platform=x64`を付ける運用にするとよい
