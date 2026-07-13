## 2026-07-13: Google Cloud OAuthクライアントのリフレッシュトークンが7日で失効する

- **エラー内容**: rclone等でGoogle Drive APIをOAuth認証で使う場合、Google Cloud ConsoleでOAuth同意画面が「テスト」公開状態のままだと、リフレッシュトークンが7日で失効し、定期的な再認証が必要になる
- **原因**: OAuth同意画面のPublishing statusが「Testing」のプロジェクトでは、Googleがリフレッシュトークンの有効期限を7日に制限する仕様
- **解決方法**: OAuth同意画面のPublishing statusを「本番（In production）」に切り替える。個人利用の範囲（自分のGoogleアカウントのみ使用、機微なスコープを使わない）であれば、Google審査（verification）を経ずに即座に切り替え可能
- **備考**: サービスアカウント方式を使えばこの問題自体を回避できるが、rcloneの設定はOAuth方式の方が`rclone config`の対話ウィザードに沿っていて手軽
