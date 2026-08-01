# knowledge-base

AI（Claude.ai / Claude Code 等）と技術資料を共有するためのパブリックリポジトリ。

## 目的

- 複数の AI サービス間で技術情報を共有する
- 実作業で得た知見（うまくいった／いかなかった方法）を蓄積し、別の場面・別の AI から再利用する
- `raw.githubusercontent.com` 経由で認証なしアクセスを可能にする

## 索引（カタログ）

調べ物・問題解決の前に、関連トピックがここにないか確認する。
raw URL 形式: `https://raw.githubusercontent.com/kkdevlab/knowledge-base/main/<ファイル名>`

### .NET

| ファイル | 概要 |
|---|---|
| DotNet_Build_Tips.md | .NET SDK / MSBuild のビルドに関する知見（ApplicationIcon が無言で埋め込まれない stale VBCSCompiler.exe 問題、実行中プロセスによるファイルロックで MSB3027 失敗、icon.ico のサイズ不足によるタスクバーのボケ 等） |
| NAudio_CoreAudioApi_Tips.md | NAudio（Windows Core Audio API ラッパー）の知見（ドキュメントより先にDLLをリフレクションでAPI確認する手法、`OnSessionCreated`/`OnVolumeNotification`の正しいシグネチャ） |

### Notion

| ファイル | 概要 |
|---|---|
| Notion_MCP_Tips.md | Notion MCP ツールのエラーと対処法（multi_select の渡し方 等） |

### Claude Code

| ファイル | 概要 |
|---|---|
| Claude_Code_MCP_Connection_Tips.md | Claude Code (VS Code拡張) のMCPサーバー接続タイミング問題（セッション開始直後は未接続のMCPがそのチャット内では回復しない仕組み・関連GitHub Issue・対処法として採用したセッション開始チェックの設計） |

### VS Code

| ファイル | 概要 |
|---|---|
| VSCode_Extension_Dev_Tips.md | VS Code拡張機能の自作・インストールに関する知見（手動フォルダコピーではextensions.json未登録のため認識されない問題と、vsce package + code --install-extensionによる正式インストール手順） |

### Windows

| ファイル | 概要 |
|---|---|
| Windows_Explorer_IconCache_Tips.md | アイコンキャッシュクリア・explorer.exe 再起動の知見（Windows 11 で ie4uinit.exe が無い場合の代替手順、Stop-Process -Force 再起動でタスクバーが消えるケースの診断・復旧） |
| PowerShell_Tips.md | PowerShellの仕様の罠（Sort-Objectの複数プロパティ指定、ConvertTo-Jsonの1件/0件配列の挙動、パイプライン0件代入の$null化、Windows PowerShell 5.1でのBOMなしUTF-8スクリプト文字化け 等） |

### Google

| ファイル | 概要 |
|---|---|
| Google_Cloud_OAuth_Tips.md | Google Cloud OAuth同意画面の設定・失効回避に関する知見 |
| Google_Drive_API_Tips.md | Google Drive API の知見（認証なしで直接ダウンロードできるURL形式・共有設定の単位） |

### Email / セキュリティ

| ファイル | 概要 |
|---|---|
| Email_Header_Spoofing_Tips.md | メールヘッダーの偽装手口・検出Tips（Unicode双方向テキスト制御文字によるFrom表示名偽装と軽量な検出方法） |

### SwitchBot

| ファイル | 概要 |
|---|---|
| SwitchBot_API_Tips.md | SwitchBot Web API v1.1（HMAC-SHA256 認証・レート制限・エンドポイント） |

### Join（joaoapps）

| ファイル | 概要 |
|---|---|
| Join_PC_Phone_Relay.md | PC⇄携帯連携と off-LAN コマンド実行の調査記録（Desktop/拡張のアーキテクチャ・Chrome中継・**失敗した手順／誤った前提と訂正**・MV3のハマりどころ・RCE注意・push-receiver-v2 推奨） |

### Tasker（仕様・ノウハウ）

| ファイル | 概要 |
|---|---|
| Tasker_Action_Codes.md | アクションコード一覧（XML `<code>` ↔ アクション名）。Task Actions／Profile States・Events／Scene V2／プラグインコード／Profile XML 要素マッピング 等 |
| Tasker_Troubleshooting.md | Tasker 利用中のエラー・警告の解決記録（汎用的な挙動・不具合） |
| Tasker_Tips.md | Tasker の仕様・ノウハウ全般（変数の型・比較演算子／設定・プロファイル変更時の Enter/Exit 再発火／%WIFII 形式／プロファイル XML 読み方 等） |
| Tasker_XML_Description_Format.md | Description 疑似コード ↔ XML バックアップの対応表（修正提案の記述用） |
| Tasker_Version_History.md | バージョン別機能一覧（6.0〜）／インストール・最新・ベータ状況 |

### Tasker（Scenes V2）

| ファイル | 概要 |
|---|---|
| Tasker_ScenesV2_Official_Manual.md | Scene V2 公式マニュアル ローカル保存版（19セクション・2026-03-18取得） |
| Tasker_ScenesV2_Manual.md | Scene V2 公式マニュアル（ベータ版記録・6.7.0-beta 時点） |
| Tasker_ScenesV2_67x-beta.md | 6.7.x-beta の Scenes V2 全仕様（Reddit 由来・6.7.5-beta まで） |

### Tasker（公式ガイド）

| ファイル | 概要 |
|---|---|
| Tasker_Userguide_Index.md | Tasker 公式ユーザーガイドへのリンク集（v6.x） |

## 注意事項

- 個人利用目的のリポジトリです
- 機密情報・認証情報（APIキー等）は含めません
- 内容は公開情報の整理・自作のまとめです
