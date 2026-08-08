# Claude Code Skill Authoring Tips

Claude CodeのSKILL.md（スキル定義ファイル）を書く際の知見。

---

## 2026-08-08: SKILL.mdのfrontmatter descriptionにコロンを含む文言を書くとYAML解析事故のリスクがある

- **エラー内容**: `description: 説明文...\`新規指摘: なし\`...`のように、バッククォート区切りでコロン+空白を含む文字列を、YAMLのplain scalar（引用符なし）としてそのまま書いていた
- **原因**: YAMLのplain scalarでは`key: value`の`: `（コロン+空白）がマッピングの区切りとして解釈されるため、説明文中に同じパターンが現れると意図しない構造として解析されるリスクがある。バッククォートはYAML上ただの文字であり引用符にはならない
- **解決方法**: frontmatterのdescriptionをblock scalar（`>-`で開始し、以降の行をインデントして続ける）に変更する
  ```yaml
  description: >-
    説明文の1行目...
    続きの行...
  ```
- **備考**: 変更後は`python -c "import yaml; yaml.safe_load(open(path,encoding='utf-8').read())"`等で実際にパースして確認するとよい
