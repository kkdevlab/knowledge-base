## 2026-07-14: 認証なしでGoogle Driveファイルを直接ダウンロードするURL形式

- **内容**: ファイルを「リンクを知っている全員が閲覧可能」に共有設定した上で、以下のURLへGETすると認証（OAuth）なしで生のファイル内容を取得できる

  ```text
  https://drive.google.com/uc?export=download&id=<FILE_ID>
  ```

- **挙動**: 上記URLは303 See Otherでリダイレクトを返す（`Location: https://drive.usercontent.google.com/download?id=<FILE_ID>&export=download`）。リダイレクトを追従すると、対象ファイルの生バイナリ/生テキストが`Content-Disposition: attachment`として返る
- **確認済みレスポンス**: `Content-Length`がDrive上の実ファイルサイズと完全一致（77667バイトのHTMLファイルで検証）。`Access-Control-Allow-Origin: *`も付与される
- **用途**: OAuthフローを実装できない/したくないクライアント（例: TaskerのHTTP Requestアクションのような素のGETしかできないツール）から、Google Drive上のファイルを直接取得したい場合に使える
- **前提・注意**:
  - リダイレクト（303）に自動追従するHTTPクライアントであること（`curl -L`、多くのモバイル自動化ツールは既定で追従する）
  - ファイルID単位で共有設定するため、フォルダ全体を公開する必要はない（フォルダは非公開のまま、特定ファイルだけ「リンクを知っている全員」にできる）
  - `rclone copyto`等で同一パスへ上書きアップロードする運用の場合、Google Drive側は同一ファイルID（既存ファイルの更新）を維持するため、上記の直接ダウンロードURLは毎回変わらず固定で使い回せる
  - ファイルサイズが大きい場合（Googleの基準でおおよそ25MB超、または実行可能ファイル等）は、ウイルススキャン警告の中間ページが挟まる場合がある（未検証。数十〜百KB程度のテキスト/HTMLでは発生しなかった）

## 2026-07-14（追記）: Taskerなど303を自動フォローしないクライアントでは新形式URLを直接使う

- **判明した事実**: 前述の`drive.google.com/uc?export=download&id=...`が返す303リダイレクトについて、「多くのモバイル自動化ツールは既定で追従する」としていたが、**Tasker（6.7.6-beta）のHTTP Requestアクションは303を自動フォローしない**ことを実機で確認した（`%http_response_code=303`・`%http_data`が空のまま返る）
- **回避策**: リダイレクト先URL`https://drive.usercontent.google.com/download?id=<FILE_ID>&export=download`を最初から直接指定する。curlで検証したところ、こちらは303を経由せず200 OKで直接ファイル内容が返る（`Content-Disposition: attachment`、`Content-Length`も一致）
- **教訓**: HTTPクライアントの303自動フォロー可否は個別に確認が必要。フォローしないクライアントを使う場合は、旧URL形式ではなく最初からリダイレクト先の新URL形式を使うべき

## 2026-08-15: search_filesはexcludeContentSnippets: trueを付けないと大量ファイルの一覧取得でトークン上限を超える

- **内容**: フォルダ内の全ファイル一覧を取得する目的（メタデータのみ必要）で`search_files`を呼ぶ際、`excludeContentSnippets`を指定しないと各ファイルの内容スニペットが結果に含まれ、ファイル数が多いフォルダでは応答が数万文字を超えてツール呼び出しがエラーになる
- **解決方法**: メタデータのみ必要な場合は`excludeContentSnippets: true`を付ける
- **備考**: タイトルの部分一致検索（`title contains '2026'`等）と組み合わせて対象を絞り込むのも有効
