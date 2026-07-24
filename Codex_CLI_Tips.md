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

---

## 2026-07-22: VS Code拡張機能(openai.chatgpt)も独自にcodex.exeを同梱している(4系統目)

- **内容**: 07-19に整理した3系統(デスクトップアプリ内蔵/PATH登録凍結版/npm)に加え、VS Code拡張機能`openai.chatgpt`(表示名「Codex – OpenAI's coding agent」)も拡張機能フォルダ内に独自の`codex.exe`を同梱している
  - 場所: `<拡張機能フォルダ>\bin\windows-x86_64\codex.exe`(例: `%USERPROFILE%\.vscode\extensions\openai.chatgpt-<version>-win32-x64\bin\windows-x86_64\codex.exe`)
  - バージョンは拡張機能ごとに固定・同梱(`codex-package.json`に記載。拡張機能v26.715.61943で同梱codex v0.145.0-alpha.27だった)
  - 設定`chatgpt.cliExecutable`(既定`null`)は「開発用のみ」と明記されており、明示指定しない限りPATH上のスタンドアロン版ではなくこの同梱版が使われる
- **確認方法**: 拡張機能フォルダ配下を`codex`でファイル名検索。`codex-package.json`にバージョン情報が入っている
- **更新の仕組み**: 拡張機能フォルダはバージョン番号入りのフォルダ名で管理されており、VS Codeが拡張機能を更新するとフォルダごと新規展開される。実機で`codex.exe`・`extension.js`・`package.json`の更新日時がほぼ同時刻だったことを確認済み → **拡張機能の更新＝同梱`codex.exe`も同時に更新される**（部分更新ではなく丸ごと差し替え）
- **3系統の更新方式の違い**（デスクトップアプリ内蔵/スタンドアロン版/VS Code拡張機能同梱版）:
  - スタンドアロン版: ターミナル起動時に更新通知が表示され、`codex-update`等のコマンドを**手動実行**して初めて更新される
  - デスクトップアプリ: アプリ自身が裏で自動更新(ハッシュフォルダを差し替え)、**手動操作不要**
  - VS Code拡張機能: VS Code本体の拡張機能自動更新の仕組みに従う。`settings.json`に`extensions.autoUpdate`の明示設定が無ければVS Code既定(自動更新ON)が有効なため、これも**基本手動操作不要**
- **注意**: 「デスクトップアプリ内蔵版」はOpenAI純正の「Codexデスクトップアプリ」のことで、Claude Code Desktopとは無関係（混同しやすいので注意）

---

## 2026-07-24: 非対話実行時にシステムプロンプト/ユーザープロンプトをファイルから読み込ませる方法

- **内容**: `codex exec`（非対話モード）実行時、システムプロンプトとユーザープロンプトはそれぞれ別の渡し方になる
  - **システムプロンプト**: 専用のCLIフラグは無く、`config.toml`のキーを`-c`で上書きする形でファイルパスを渡す
    - `-c model_instructions_file="<path>"`: デフォルトの組み込み指示(AGENTS.mdの代わり)を丸ごと置き換え。`~/.codex/config.toml`に恒久設定として書いても、`-c`でその実行だけ上書きしてもよい
    - `developer_instructions`(文字列)は追記寄りのオプションとして存在するが、ファイルパス対応の専用キーは未確認
  - **ユーザープロンプト**: `codex exec [PROMPT]`の`PROMPT`引数を省略する(または`-`を渡す)と、標準入力(stdin)から読み込む仕様になっている。ファイルの中身をパイプで流し込める

    ```powershell
    Get-Content "C:\path\to\user_prompt.txt" -Raw | `
      codex exec -c 'model_instructions_file="C:\path\to\system_prompt.txt"'
    ```

- **AGENTS.mdとの違い**: `model_instructions_file`はその実行1回だけに効く一時的な指定。プロジェクト配下やホームディレクトリ(`~/.codex/AGENTS.md`)の自動読込ファイルとは別物で、非対話スクリプト実行の文脈では明確に区別する
- **結果の取得**: `-o` / `--output-last-message <path>`で最終応答をファイルに書き出すのが定番(標準出力にも出るが、パースするならこちらが確実)
- **Claude Code CLIとの対比**: Claude Codeは`--system-prompt-file`という専用フラグを持つが、Codexは汎用の`-c model_instructions_file=`経由という設計の違いがある
