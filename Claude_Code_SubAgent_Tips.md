# Claude Code SubAgent Tips

Claude Codeのカスタムサブエージェント（.claude/agents/*.md）を書く際の知見。

---

## 2026-08-18: カスタムサブエージェント(.claude/agents/*.md)を新規作成しても同一セッション内で認識されない

- **エラー内容**: 新規作成した`.claude/agents/<name>.md`をAgentツールで`subagent_type`指定して呼び出すと `Agent type '<name>' not found` エラーになる
- **原因**: Claude Codeはサブエージェント定義ファイルの変更を数秒以内に検知して次回の委譲から反映するが、**`.claude/agents/`ディレクトリを初めて作成した場合は例外で、セッション再起動（新規チャット開始）が必要**（公式ドキュメント code.claude.com/docs/en/sub-agents に明記。他に `--add-dir` で追加したディレクトリ配下のケース、`--disable-slash-commands` 起動時も同様に再起動が必要）
- **解決方法**: `.claude/agents/`初回作成後は、新しいチャット（新セッション）を開始してからサブエージェントを呼び出す
- **備考**:
  - frontmatterの必須項目は`name`と`description`のみ。`tools`はカンマ区切り文字列（YAML配列ではない）。`model`はエイリアス指定（`sonnet`/`opus`/`haiku`/`fable`）で常に各ファミリー最新版に自動追従し、フルモデルID指定で特定バージョンに固定できる
  - `claude-code-guide`サブエージェントに同じ質問をした際、「toolsはYAML配列必須」「scopeフィールドが必須」という誤情報を返した実例がある。サブエージェントの回答も鵜呑みにせず、重要な仕様は公式ドキュメント(code.claude.com)で裏取りすること
