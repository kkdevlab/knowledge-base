# Codex CLI Tips

`codex exec`（OpenAI Codex CLI 非対話モード）を外部スクリプトから呼び出す際の知見。

---

## 2026-07-17: codex execには構造化出力(JSON Schema強制)の仕組みがない

- **内容**: Claude Code CLI（`--json-schema`）やGemini API（`generationConfig.responseSchema`）と異なり、`codex exec`にはモデル応答をスキーマに強制するオプションが無い（`codex exec --help`で確認、v0.130.0-alpha.5時点）
- **対処**: JSON Schemaの本文をプロンプトに埋め込み、「このJSONオブジェクトのみを出力し、説明文やMarkdownのコードフェンスを付けない」と明示的に指示する。`--output-last-message <path>`で最終メッセージをファイル出力させ、そのファイルをJSONとしてパースする
- **フォールバック**: 指示に従わずコードフェンスや前置き文が混じった場合に備え、パース失敗時は最初の`{`から最後の`}`までを抜き出して再パースするフォールバックを入れておくと安全（実機では素直にJSONのみが返ったが、保険として有効）
- **確認済みバージョン**: codex-cli 0.130.0-alpha.5

---

## 2026-07-17: reasoning effortは専用フラグがなく `-c model_reasoning_effort` で指定する

- **内容**: Claude Code CLIの`--effort`のような専用オプションは`codex exec`には無い。`-c, --config <key=value>`（`~/.codex/config.toml`の値を上書きする汎用オプション）経由で指定する
  ```powershell
  codex exec -c 'model_reasoning_effort="high"' ...
  ```
- **値はTOMLとして解釈される**: `-c`の値部分はTOMLとして解釈されるため、文字列は`"high"`のようにダブルクォートで囲む（クォートなしでパース失敗した場合はリテラル文字列として扱われるフォールバックもあるが、明示的にクォートする方が安全）
- **デフォルト値の決まり方**: `-c`で上書きしない場合、`~/.codex/config.toml`に`model_reasoning_effort = "high"`のような記載があればそれがそのまま使われる（ローカル設定ファイルの値がCLI実行時にそのまま読まれる仕組み）
- **確認済みバージョン**: codex-cli 0.130.0-alpha.5

---

## 2026-07-19: Windows版はCLI実体がアプリ内蔵・PATH登録・npm/スタンドアロン等で複数系統に分裂しうる

- **内容**: Codexデスクトップアプリ（Windows）をインストールすると、`AppData\Local\OpenAI\Codex\bin\codex.exe`というPATH登録済みのCLI実体が最初に1回だけ配置される。デスクトップアプリ自身はその後、自動更新のたびに`AppData\Local\OpenAI\Codex\bin\<ハッシュ名フォルダ>\codex.exe`という別の場所に新しいバイナリを展開して内部的に使うが、**最初にPATH登録された方は一切更新されない**。加えてnpmグローバル版（`npm install -g @openai/codex`）や公式スタンドアロン版（後述）を別途導入すると、同一マシンに複数系統のCLIが混在し、`codex`コマンドはPATHの並び順で解決されるため、意図せず最も古い（更新されない）バイナリが使われ続けることがある
- **症状**: 最新モデル（例: `gpt-5.6-sol`）を指定すると `"The '<model>' model requires a newer version of Codex. Please upgrade to the latest app or CLI and try again."` のようなエラーになるのに、デスクトップアプリの「About」表示では最新版になっているように見える、という食い違いが起きる
- **確認方法**: `Get-Command codex -All` で全実体を洗い出し、`(Get-Item <path>).VersionInfo` や `codex --version` で各バイナリのバージョンを比較する
- **対処**: OpenAI公式のスタンドアロン版CLIインストーラーで統一するのが確実

  ```powershell
  powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"
  ```

  npm版が入っている場合は競合検出され対話プロンプトでアンインストール確認が出るが、非対話実行時は自動アンインストールされないため`npm uninstall -g @openai/codex`を別途手動実行する。インストーラーは`%LOCALAPPDATA%\Programs\OpenAI\Codex\bin`をPATH先頭に自動追加し、`~\.codex\packages\standalone\current`というジャンクションで現在バージョンを管理する
- **確認済みバージョン**: アプリ内蔵版 0.145.0-alpha.18、PATH登録の凍結版 0.130.0-alpha.5、スタンドアロン版 0.144.6（2026-07-19時点）
