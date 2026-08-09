# Readability / Turndown Tips

Mozilla Readability・Turndownを使ったWebページ→Markdown変換で遭遇した挙動・回避策のまとめ。

## 2026-07-12: Readabilityが`<pre>`/コードブロックを削除する

- **エラー内容**: Readability.parse()の結果から、コードブロック(`<pre><code>`)の中身が完全に欠落する。周囲の文章は残るのに、短いコード片・箇条書き的なコード行だけが消える
- **原因**: Readabilityの`_cleanConditionally`が、記事本文っぽくない要素（句読点が少ない・リンク密度が高い等）を「広告/ボイラープレート」とみなして削除するヒューリスティックを持つ。コードブロックは句読点が少なく短いため、この判定に引っかかりやすい
- **解決方法**: Readability実行前に`<pre>`要素をプレースホルダー(`<p>MARKER</p>`)に置き換えて退避し、`article.content`生成後にプレースホルダーを元の`outerHTML`に戻す
- **備考**: Turndown自体は`<pre><code>`をfenced code blockに正しく変換できるので、Readabilityの手前で保護すれば問題ない

## 2026-07-12: Readabilityは`<nav>`タグを自動除去しない

- **エラー内容**: サイドバーのナビゲーション（チャット履歴一覧など）が本文と一緒に抽出される
- **原因**: Readabilityは記事抽出後に`<footer>`と`<aside>`は自動除去するが、`<nav>`は対象外。className/idに"sidebar"等のキーワードが含まれない場合（Tailwind等の汎用クラス名のみのSPA）、除外条件にも引っかからずそのまま残る
- **解決方法**: Readability実行前に`document.querySelectorAll('nav, [role="navigation"]')`で除去しておく

## 2026-07-12: Turndown標準にはGFMテーブル変換が含まれない

- **エラー内容**: `<table>`をturndownで変換すると、セルの中身が1つずつ改行区切りの段落として羅列されてしまい、表の体裁にならない
- **原因**: Turndownのコア(`turndown.js`)は`TABLE`/`TR`/`TD`/`TH`を単なるブロック要素としてしか扱わず、パイプテーブル記法(`| a | b |`)への変換ルールを持たない。これは公式プラグイン`turndown-plugin-gfm`が担う機能
- **解決方法**: `turndown-plugin-gfm`の`tables`ルール相当を実装して`turndownService.use()`で追加する（見出し行(`<thead>`または先頭行が全て`<th>`)を持たないテーブルは`keep()`でHTMLのまま保持するのが安全）

## 2026-07-12: File System Access APIの`getFileHandle`が「Name is not allowed」で失敗する

- **エラー内容**: `FileSystemDirectoryHandle.getFileHandle(name, {create:true})`が`TypeError: Failed to execute 'getFileHandle' ... Name is not allowed.`を投げる。名前を目視確認しても禁止文字（`\ / : * ? " < > |`）は含まれておらず、`JSON.stringify()`で確認しても異常が見えない
- **原因**: ファイル名の元になった文字列（Webページの`document.title`等）に、**目に見えない Unicode 制御文字・書式文字**（例: U+200E LEFT-TO-RIGHT MARK、ゼロ幅スペース等）が混入していた。これらはJSの`.trim()`（空白文字のみ対象）でも除去されず、`JSON.stringify()`でもエスケープされずそのまま出力されるため画面上は「正常」に見える
- **解決方法**: ファイル名生成時に、Unicodeカテゴリ`\p{Cc}`（制御文字）・`\p{Cf}`（書式文字）を正規表現で除去する。例: `name.replace(/[\p{Cc}\p{Cf}]/gu, "")`
- **備考（不可視文字の調べ方）**: `JSON.stringify(str)`では見つからない場合、文字コード単位でダンプすると発見できる。
  ```js
  Array.from(str).map(c => c.codePointAt(0).toString(16)).join(' ')
  ```
  例えば`200e 47 6f 6f ...`のように出力されれば、先頭にU+200Eが混入していると分かる

## 2026-08-09: pre要素保護用プレースホルダーが短すぎると、周辺ブロックごとReadabilityに削除される

- **エラー内容**: `<pre>`要素を短いプレースホルダー（IDの文字列のみ）に置き換えて保護したはずが、Markdown化後にその箇所が丸ごと欠落する
- **原因**: プレースホルダーの文字数が元のコード内容より極端に少ないと、文章密度ヒューリスティックにより周辺ブロックごと「価値の低いボイラープレート」と誤判定され削除される。保護のつもりの操作が逆に削除を誘発する
- **解決方法**: プレースホルダーのテキスト長を、元の`pre.textContent.length`以上（下限目安80文字）のダミー文字列で埋めてから復元用の一意な属性を付与し、復元時に元のHTMLへ置換する
- **備考**: note.comの記事ページで実機発見。保護なしでそのままReadabilityに渡せば通常サイズのpreは自然に保持されることも確認済み
