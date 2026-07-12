# Join（joaoapps）PC⇄携帯 連携と off-LAN コマンド実行の調査記録

Join（joaoapps）で **PC と Android(Tasker) を双方向に連携**させる際の仕組み・実証結果・**失敗した手順／誤っていた前提とその訂正**をまとめる。
特に「**携帯が自宅Wi-Fi外（off-LAN, モバイルデータ）から PC にコマンドを実行させる**」用途（例: 外出時に PC アプリを kill）を主題とする。

> このドキュメントは「成功した方法」だけでなく「**やってみて間違いだった手順・誤った思い込み**」も意図的に残している。再調査時に同じ轍を踏まないため。

調査対象バージョン（重要・挙動はバージョン依存）:
- Join Desktop（Electron, `joaomgcd/JoinDesktop`）: **v1.1.3**（2024-07で更新停止＝薄い実行役）
- Join Chrome 拡張（`joaomgcd/JoinChrome`）: **1.9.3**（store版・現役メンテ）
- 公式: joaoapps.com/join/desktop, joaoapps.com/join/faq

---

## 0. 重要更新（2026-06-29）：off-LAN 携帯→PC コマンド実行は実機成功した。push-receiver-v2 は不要

**結論が変わった。** 携帯=モバイルデータ（Wi-Fiオフ）で Chrome拡張デバイス宛に commandLine push を送ったところ、拡張が受信→`localhost:9876/push` へ転送→Join Desktop が実行→PCのファイルが書き換わった。**off-LAN 上りは出荷版の拡張だけで成立する。** 以前の本ドキュメント（および「不可能・push-receiver-v2 必要」とした結論）は誤りで、以下が真因だった。

### 真因＝設定キーの取り違え（出荷版が読むキーが違った）

- 出荷版1.9.3 の受信コード `js/gcm.js` は、転送先として **`eventghostport` + `redirectionfullpush`** だけを読む（= v1 `options.html` の **Advanced → 「EventGhost, Node-RED」** セクション）。`redirectionfullpush` が ON のとき `POST http://localhost:<port>/push`（完全な push JSON）を送る＝**Join Desktop の `/push` が期待する形式と一致**。
- 一方、過去に手で設定した **`SettingCompanionAppPortToConnect`（v2「Join Desktop Port」）は出荷版が一切読まないキー**。v2設定アプリ(`v2/app.js`)はどのHTMLにも配線されておらず到達不可（`devices.html` が読む `appv2.js` は別物の死にコード、設定歯車は v1 `options.html?tab=1` を開くだけ）。`?connectoport` 自動ペアリングも OAuth も**出荷版では動かない**。`join.html` は設定画面ではなく**オフスクリーン（裏方処理）ページ**＝真っ白で正常。

### 正しい設定（PC・Chrome、すべて出荷版UIで到達可能）

1. 拡張の `options.html` → **ADVANCED タブ → 「EventGhost, Node-RED」**
2. **Port = `9876`**（Join Desktop の受信ポート `SettingCompanionAppPortToReceive` の既定）／ Server IP/Host = 空欄（localhost）
3. **「Send Full Push (enable if you use Node-RED)」を ON**（← これが欠けていた唯一のピース）

### 「統一」の核心

**コマンドは「Chrome（拡張）デバイス」宛に送る。** 拡張は常に FCM クラウド経由で受信し localhost へ中継するため、**携帯のネットワーク（自宅Wi-Fi/外出モバイルデータ）を問わず、同一 push が同一に動く**。自宅LAN直送と外出時の橋渡しを使い分ける必要が消え、経路が1本化される。

### 残る前提

- ⚠️ **Chrome が起動している必要がある**（拡張が受信→中継するため。公式仕様「別ネットワークではブラウザを開いたまま」どおり。離宅中も PC の Chrome 常駐が条件）。
- 転送は **text を持つ全 push を 9876 に流す**（コマンド以外も Desktop に届くが、実行されるのは commandLine push のみ）。
- RCE級リスクが off-LAN でも有効化された＝`JOIN_API_KEY` 秘匿の重要度が上がった（§5 参照）。

> 以下 §1〜§6 は2024-07の初回調査時の記録。**§0 が最新の確定事実**。§1・§4.4・§6 の「不可能／push-receiver-v2推奨」は §0 で覆っている点に注意して読むこと。

---

## 1. 結論（※§0 で更新済み・初回調査時の記録）

- **off-LAN の「携帯→PC コマンド実行」は Join 公式の "Chrome 中継" で技術的に成立可能**。→ ✅ §0 で実機実証。
  - 経路: 携帯(モバイルデータ) → FCM クラウド → **PC の Chrome 拡張が受信** → 拡張が **同一PC内の localhost:ポート** へ転送 → **Join Desktop がコマンド実行**。
  - 公式も明記: 「別ネットワークではブラウザが push を受けてアプリへ転送する（**ブラウザを開いたままにする**）。同一LANなら直接。**送信のみブラウザ無しでも可**」。
- ただし **Join Desktop 単体には push 受信手段が無い**（クラウド受信を実装していない）。off-LAN 受信はブラウザ(拡張)頼み。
- ~~「実 push を受けた時の自動転送だけが発火しない」~~ → **§0 で解決**。原因は転送先設定を v2 キー(`SettingCompanionAppPortToConnect`)に書いていたこと。出荷版が読む v1 キー(`eventghostport`+`redirectionfullpush`)に正しく設定すれば発火する。
- ~~push-receiver-v2 推奨~~ → **不要**（§0）。出荷版の拡張中継で off-LAN が成立した。

---

## 2. アーキテクチャ（ソースで確認した実体）

### 2.1 Join Desktop（Electron アプリ）

- **クラウド push 受信を実装していない**。`package.json`（v1.1.3）に `push-receiver`/`electron-push-receiver`/GCM/mtalk 系の依存は**無い**。app.asar の grep でも `mtalk.google`/`push-receiver`/`androidId` ヒット0。外向き確立接続ゼロ（GCM 5228 を張らない）。**送信のみ**クラウド（FCM v1）を使う。
- 受信は **ローカル HTTP サーバ**のみ（`server/server.js`）。ポートは設定 `SettingCompanionAppPortToReceive`（既定 9876、全IF待受）。
  - 任意パスへの POST を受け、body を JSON parse → `sendToPage(body)`。
  - body 形式は `{ "type": "GCMPush", "json": "<GCMPushをJSON文字列化したもの>" }`（json は**文字列**である点に注意。中身に `push:{...}` が入る）。
  - `GET /任意` は `{"success":true}` を返すだけの ACK（push 注入の正式経路は上記 POST）。
- **コマンド実行経路**: 受信 push を renderer(`appdashboard.js`)が処理し、`push.commandLine` の時:
  - `SettingRequireEncryptionForCommandLine`（要暗号化）が ON かつ未暗号化なら**拒否**。OFF なら実行。
  - `push.text` を **`=:=`** で分割（左=command, 右=args。`=:=` 無しなら text 全体が command）。
  - `child_process.spawn(command, args, { shell:true })` で実行（Windows は cmd 経由）。
  - `push.commandResponse` があれば `<commandResponse>=:=<stdout>` を送信元へ返信（双方向）。
- ⚠️ 注意: ローカルサーバの汎用受信（`sendToPage`→`GCMPush.execute()`）は **clipboard/url/files は処理するが commandLine は実行しない**。commandLine 実行は renderer 側の別経路（`RequestRunCommandLineCommand`）。

### 2.2 Join Chrome 拡張（受信の主役）

- `manifest.json`: `default_popup = devices.html`、`options_ui.page = options.html`、SW = `service_worker.js`、permissions に `gcm`/`offscreen` 等。
- **Service Worker は薄い**: `service_worker.js` は `cross_context.js` を importScripts し、**オフスクリーン文書 `join.html` を生成**、通知クリックを処理するだけ。**SW 自体に Dexie も GCM 処理クラスも無い**。
- **GCM 受信は SW の `chrome.gcm.onMessage`** で受けるが、SW は処理せず **cross_context 経由でオフスクリーン/フォアグラウンドのリスナーへ中継**する。
- **実処理（push の execute → 転送）はオフスクリーン文書 `join.html` 側**で動く（`join.html` は `js/base/dexie.js` をグローバルロード＝Dexie が使える）。
- **デスクトップへの転送**: `v2/gcm/gcmbase.js` の `sendToCompanionAppIfNeeded()`:
  - 設定 `SettingCompanionAppPortToConnect`（拡張UIの **"Join Desktop Port"**, v2設定の **General** タブ）を読む。
  - 値があれば `sendToLocalPort({port})` → `fetch("http://localhost:<port>/push", { method:"POST", mode:"no-cors", body: JSON.stringify(gcmRaw) })`。
  - **送信先は localhost 固定**（＝拡張とデスクトップが同一PC前提。だから携帯のネットワークに依存せず off-LAN でも橋渡しできる）。
  - **push 全体（commandLine 含む）をフィルタせず転送**する。
- 受信時の `execute()`（`v2/gcm/gcmapp.js` GCMBaseApp）:
  ```js
  async execute(){
    const sent = await this.sendToCompanionAppIfNeeded();
    if(sent){ return; }            // 転送したら通知は出さず終了
    this.sendToLocalAutomationPortIfNeeded(); // EventGhost/Node-RED（別機能）
    await EventBus.post(this);     // → 通知表示など
  }
  ```
  - 別実装 `v2/gcm/gcmserviceworker.js` GCMBaseServiceWorker は転送成功時に `[{title:"Join Push",text:"Sent to Companion App"}]` を返す（**この版だと転送成功でも通知が出る**点に注意）。

### 2.3 設定の保存先（Dexie / IndexedDB）

- 設定は IndexedDB の **`join_settings` DB / `settings` ストア**（Dexie）に `{ id, value }` で保存。
- Dexie は `db.version(1).stores({ settings: 'id,value' })`。`'id,value'` = 主キー `id` ＋ `value` のインデックス。
- **Dexie のバージョン採番**: `indexedDB.open(name, Math.round(verno*10))` ＝ **version(1) は IDB バージョン 10**。手動で IDB を作る場合は version 10・主キー`id`・index`value` で作らないと Dexie と噛み合わない。

---

## 3. 実証できたこと（個別には全部 OK）

| 検証 | 方法 | 結果 |
|---|---|---|
| デスクトップが commandLine 実行 | PowerShell から `http://localhost:9876/push` に上記形式で POST | ✅ ファイルが書き換わる |
| 拡張SWから localhost へ届く | SW コンソールで同じ `fetch`（no-cors） | ✅ デスクトップ実行 |
| オフスクリーンから localhost へ届く | join.html コンソールで同じ `fetch` | ✅ デスクトップ実行 |
| off-LAN 受信 | 携帯(モバイルデータ)→**Chrome デバイス宛**に push | ✅ PCのChromeに通知が出る |
| 設定値の読取 | join.html / devices.html で `new Dexie("join_settings")...get("SettingCompanionAppPortToConnect")` | ✅ `value: 9876` を返す |

→ **受信・設定読取・転送fetch・デスクトップ実行、全ての脚が単体では成立**することを確認した。

---

## 4. 失敗・誤った前提・ハマりどころ（重要）

### 4.1 結論レベルの誤りと訂正

- ❌（旧）「off-LAN では携帯→PC コマンド実行は Join では不可能」
  - → ✅（訂正）**公式の Chrome 中継で可能**。Desktop 単体が受信できないのは事実だが、**ブラウザ(拡張)が FCM 受信→localhost 転送**で橋渡しする設計。「設定で直らない/不可能」は誤りで、正しくは「**正規設定が要る**」。
- ❌「JoinDesktop は electron-push-receiver で GCM 直受信しているかも」
  - → ✅ **していない**。`package.json` に依存なし。Desktop は受信手段を持たない（送信のみクラウド）。
- ❌「公式の "ブラウザ中継" は本環境では機能しない（送信先を変えても不発）」と早合点
  - → 実際は **拡張の "Join Desktop Port" 設定が未設定**だと `sendToCompanionAppIfNeeded()` が何もしない。中継の前提設定が入っていなかっただけ、という可能性が高い。

### 4.2 拡張UI（設定画面）に関する誤り

- ❌ `options.html`（`options_ui` の画面）に「Join Desktop Port」がある、と探した。
  - → **`options.html` は旧(v1)設定**（タブ: Shortcuts/Options/Clipboard/Actions/Encryption/Diagnostics/Advanced/Help）。**"Join Desktop Port" は無い**。`?tab=数字` 指定も無効（タブは名前管理）。
- ✅ 正しくは **v2 アプリ `join.html` の Settings → General タブ**にある（`SettingCompanionAppPortToConnect`、ラベル "Join Desktop Port"）。
- ❌（本環境の問題）`join.html` をタブで直接開くと**真っ白で描画されない**（offscreen 用途のページをタブで開くと初期化が噛み合わない）。このため UI から設定できなかった。
  - 正規 UI で設定すると、ポート保存に加えて **接続テスト＋OAuth の companion ペアリング**（`redirect_uri=http://127.0.0.1:<port>`、Desktop 側が `code` を受けて認証）まで行う。手動設定ではこのペアリングが抜ける。

### 4.3 DevTools / IndexedDB 直書きのハマりどころ

- ❌ `indexedDB.open("join_settings")`（バージョン無し）で**様子見** → Dexie 未作成だと**空の v1 DB を新規作成してしまう**副作用。Dexie 管理DBを生 open で覗くのは避ける。
- ❌ Service Worker のコンソールで `new Dexie(...)` → **`Dexie is not defined`**。SW には Dexie が無い（`service_worker.js` は `cross_context.js` しか読まない）。Dexie はオフスクリーン/ページ側。
- ❌ SW で `await import('/v2/settings/setting.js')` → **`import() is disallowed on ServiceWorkerGlobalScope`**（SW では動的 import 不可）。
- ❌ ページで `setting.js` を import → `Dexie is not defined`（`setting.js` は**グローバル Dexie 前提**。Dexie を `<script>` で持つページ＝`devices.html`/`join.html` でのみ動く）。
- ❌ 生 IDB で `settings` ストアを作るとき **version とスキーマを間違える**と Dexie が開けない/読めない。
  - → ✅ **IDB version 10・`createObjectStore("settings",{keyPath:"id"})`・`createIndex("value","value")`** で作れば Dexie 側 `version(1)` と一致して読める（実機で `value:9876` の読取成功を確認）。
- ハマり: **DevTools の貼り付けガード**。`貼り付けを許可`（英語なら `allow pasting`）は**手入力必須**（貼り付け不可）。
- ハマり: **コンソールのログレベルフィルタ**で `console.log`(Info/Verbose) が隠れて「何も出ない」ように見える。→ **トップレベル await で式の評価値として返す**とフィルタの影響を受けず `<` 行に出る。
- ハマり: **MV3 の SW/オフスクリーンは短命**（アイドルで停止→push で再起動）。コンソールで仕掛けた `fetch` フック等は再起動で消え、**ライブ観測が不安定**。fetch をフックして「push 時に転送 fetch が出るか」を見る方法は当てにならなかった。

### 4.4 「最後まで残った謎」は解決済み（§0 参照）

- 当時の症状: 設定値 `SettingCompanionAppPortToConnect=9876` は読めるし、オフスクリーンから localhost への手動 `fetch` も通る。なのに実 push では転送が発火しない。
- **解決（2026-06-29）**: 出荷版の受信コード `js/gcm.js` は `SettingCompanionAppPortToConnect`（v2キー）を**そもそも読まない**。読むのは `eventghostport` + `redirectionfullpush`（v1キー）。`sendToCompanionAppIfNeeded()`（v2/gcm/gcmbase.js）は出荷版の live 受信経路では呼ばれていなかった。
- 正しく v1 キー（options.html → Advanced → EventGhost/Node-RED の Port=9876 ＋ Send Full Push=ON）に設定したら、**off-LAN で一発で発火**した。
- 教訓: **「リポジトリにあるコード」と「出荷ビルドで配線され実行されるコード」は別**。v2 一式は repo にあるが未配線の死にコードだった。設定キーは**実際に走る受信経路のコードから逆算**して特定すること。

---

## 5. セキュリティ上の注意（重要）

- **commandLine 付き push ＝ PC で任意シェル実行**。Join Desktop には**実行ON/OFFトグルもコマンド・ホワイトリストも無い**。唯一の安全装置が `SettingRequireEncryptionForCommandLine`（要暗号化）だけ。
  - つまり **API キーを握る者は PC で任意コマンド実行可能（RCE 級）**。
  - ただし**暗号化 ON は携帯→PC のインライン画像送信を壊す**（画像は `files` 配列で届き、復号ループが文字列専用のため）。画像連携を使うなら暗号化 OFF＝ノーガードになるトレードオフ。
- ⚠️ **機密ファイル（共有・公開・リポジトリ投入は厳禁）**:
  - Join API キー（PCで任意シェル実行できる＝実質 RCE の鍵）
  - `%APPDATA%\Join Desktop\auth.json`（Google の有効な refresh token）
  - `%APPDATA%\Join Desktop\devices.json`（各端末の FCM 登録ID）
  - `%APPDATA%\Join Desktop\notifications.json`（受信通知のローリング記録。本文が平文で残る）

---

## 6. 推奨（※§0 で更新）：まず出荷版の Chrome 拡張中継を使う。push-receiver-v2 は不要

- **第一選択＝出荷版 Chrome 拡張の EventGhost フルプッシュ転送**（§0 の設定）。実機で off-LAN 成立済み・追加実装ゼロ。
  - 条件: **PC の Chrome を常駐**させる（拡張が受信→中継するため。離宅中も起動が必要）。
- **push-receiver-v2（PC常駐の自前FCM受信＋ホワイトリスト）は当面不要**。次のどちらかが問題化したときだけ再検討する:
  - 離宅中に Chrome を起動し続けたくない（ブラウザ非依存にしたい）。
  - Join ネイティブのノーガード（任意シェル実行＝RCE級）を**コマンドのホワイトリスト**で塞ぎたい（暗号化ONは画像送信を壊すため、ホワイトリストが現実的な防御）。
- Join Desktop は2024-07で更新停止だが「薄い localhost 実行役」なので枯れていて問題ない。Chrome 拡張は現役メンテ。

---

## 8. 2026-07-12: Join Desktop の認証ループ（401 "Loaded user null"）の真因はブラウザの通知許可自動ブロック

- **エラー**: Join Desktopアプリのウィンドウが常に空白のまま。DevToolsコンソールに
  `Failed to load resource: www.googleapis.com/...userinfo?alt=json 401`、`Loaded user null`、
  `Showing dialog for N seconds DialogOk` → `Didn't get choice from dialog Timed out` が繰り返し出る。
  Googleアカウント側でJoinへのアクセスを一度revoke→再連携すると発生しやすい。
- **原因**: `https://joinjoaomgcd.appspot.com/?settings` のポート連携フロー（General → Join Desktop Port →
  Port入力 → SAVE）は、完了に**ブラウザの通知許可**を必要とする。Chromiumは同一サイトの許可プロンプトを
  ユーザーが何度も無視/拒否したと判断すると、**二度とプロンプトを出さず自動でBlock**する仕様がある
  （コンソールに `Notifications permission has been blocked as the user has ignored the permission
  prompt several times` と出る）。ブロックされると、アプリ側は許可待ちダイアログを出し続けたまま
  永遠にタイムアウト→再表示を繰り返し、外見上「Googleログインにループする」ように見える。
- **正解**: ブラウザのアドレスバー横のサイト情報アイコン（鍵/調整アイコン）→ このサイトの許可設定 →
  「通知」を Block → Ask/Allow に変更 → ページ再読み込み → 再度 Port(例:9876)を入力してSAVE →
  今度は通知許可プロンプトが出るので許可 → Googleサインインへ進む。認証成功後は
  `%APPDATA%\Join Desktop\auth.json` が生成され、アプリのダッシュボードが正常表示される。
- **注意**: 当初「Googleが廃止した `gapi.auth2` ライブラリのサーバー側実装が原因で修正不可能」と結論したが、
  これは**誤りだった**（ソースコード調査からの早合点）。ブラウザを変える（Chrome→Edge）と挙動が違ったのが
  ヒントで、実際は上記のブラウザ側の許可ブロック状態が原因だった。**「ソースを読んで詰んでいる」と結論する前に、
  別ブラウザで試すなど単純な環境要因を先に切り分けること。**

---

## 7. 参考

- ソース: `github.com/joaomgcd/JoinDesktop`（Electron Desktop）/ `github.com/joaomgcd/JoinChrome`（拡張）。
- 公式: `joaoapps.com/join/desktop`, `joaoapps.com/join/faq`。
- 送信API（送信は off-LAN でもブラウザ不要）: `https://joinjoaomgcd.appspot.com/_ah/api/messaging/v1/sendPush?apikey=...&deviceId=...&text=...&title=...`
  - デバイス一覧: `.../registration/v1/listDevices?apikey=...`。**送信に使う deviceId は long hex フィールド**（listDevices の数値 `id` は内部IDで送信不可）。
