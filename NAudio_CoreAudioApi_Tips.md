# NAudio CoreAudioApi Tips

NAudio（.NET の Windows Core Audio API ラッパー）を使う際の知見まとめ。

---

## ドキュメントより先に、実際にインストールされたDLLをリフレクションで確認する

### 経緯

NAudio の `AudioSessionManager` に「新規音声セッション作成時に発火するイベント」があるはずだが、ネット上の情報や過去の記録で見た名前（`SessionCreated`）で書いたコードがビルドエラーになった。

### 解決方法

PowerShellで、実際にプロジェクトが参照している NuGet パッケージの DLL をロードし、`System.Reflection` でイベント・メソッドの実シグネチャを確認する。

```powershell
$dll = Get-ChildItem "$env:USERPROFILE\.nuget\packages\naudio.wasapi" -Recurse -Filter "NAudio.Wasapi.dll" | Select-Object -First 1
$asm = [System.Reflection.Assembly]::LoadFrom($dll.FullName)
$t = $asm.GetType("NAudio.CoreAudioApi.AudioSessionManager")
$t.GetEvents() | ForEach-Object { "$($_.Name) : $($_.EventHandlerType)" }
$t.GetEvents() | ForEach-Object { $_.EventHandlerType.GetMethod("Invoke").GetParameters() | ForEach-Object { $_.ParameterType.FullName + " " + $_.Name } }
```

これにより、ドキュメントやネット記事を当てにせず、**今実際にビルドで使われるバージョンの正確なAPI形状**（イベント名・デリゲート・パラメータ型）を確認できる。バージョン間でAPIが変わっている可能性があるため、特に有効。

### 判明した正しいAPI（NAudio 2.2.1時点）

| 用途 | 誤り（存在しない） | 正しいAPI |
|---|---|---|
| 新規音声セッション作成の検知 | `AudioSessionManager.SessionCreated` | `AudioSessionManager.OnSessionCreated`（delegate: `void(object sender, IAudioSessionControl newSession)`） |
| デバイスの音量変化の直接検知 | - | `AudioEndpointVolume.OnVolumeNotification`（delegate: `void(AudioVolumeNotificationData data)`、`data.MasterVolume`が0〜1のスカラー値、`data.Muted`がミュート状態） |

**注意**: `OnVolumeNotification`は「誰が」「なぜ」音量を変えたかは区別できない。ユーザーの手動操作・他アプリの意図的なダッキング（音声入力ソフト等）・デバイス自身の内部同期、いずれも同じイベントとして届く。原因を区別する必要がある場合は、`OnSessionCreated`（新規セッション作成）等の別シグナルと組み合わせる設計が必要。
