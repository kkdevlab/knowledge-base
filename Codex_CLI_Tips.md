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
