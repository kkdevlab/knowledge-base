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
| DotNet_Build_Tips.md | .NET SDK / MSBuild のビルドに関する知見（ApplicationIcon が無言で埋め込まれない stale VBCSCompiler.exe 問題 等） |

### Notion

| ファイル | 概要 |
|---|---|
| Notion_MCP_Tips.md | Notion MCP ツールのエラーと対処法（multi_select の渡し方 等） |

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
