# Tasker アクションコード一覧

参照元: [Taskomater/Tasker-XML-Info](https://github.com/Taskomater/Tasker-XML-Info/blob/master/Tasker_XML_Codes.md)
※ 非公式だが、実際の動作と一致することを確認済み

---

## よく使用するTask Actions

### 制御フロー
| Code | アクション名 |
|------|-------------|
| 37 | If |
| 38 | End If |
| 39 | For |
| 40 | End For |
| 43 | Else |
| 126 | Return（戻り値あり） |
| 135 | Goto |
| 137 | Stop Task（Return・戻り値なし） |
| 300 | Anchor（GoTo先・タスク先頭ヘッダー用。テキストは label 要素／引数なし） |

### 待機
| Code | アクション名 |
|------|-------------|
| 30 | Wait |
| 35 | Wait Until |

### ディスプレイ
| Code | アクション名 | 備考 |
|------|-------------|------|
| 348 | Test Display | arg0=Type（`3`=Available Resolution）。結果を `arg2` の変数に格納。Available Resolution は `幅x高さ` 文字列を返す（例: `1080x2400`）→ `x` で Variable Split して使う。`Sub_JSON_Viewer`(V1) で確認（2026-06-19） |
| 806 | Turn On | ディスプレイがオフの場合にオンにする。Block Time (ms) = 画面点灯後に操作をロックする時間。公式: ah_poke_display.html |
| 812 | Display Timeout | arg0=Secs, arg1=Mins, arg2=Hours |

### 変数操作
| Code | アクション名 | 備考 |
|------|-------------|------|
| 488 | Format JSON | arg1=JSON文字列, arg2=Format（"Pretty"等）, arg4=Overwrite Source Variable(1=On)。`Weather (Google).prj.xml` task119/149 で確認（2026-07-04） |
| 545 | Variable Randomize | |
| 547 | Variable Set | |
| 548 | **Flash** | |
| 549 | Variable Clear | |
| 590 | Variable Split | |
| 592 | Variable Join | |
| 595 | Variable Query | |
| 596 | Variable Convert | |
| 597 | Variable Section | |
| 598 | Variable Search Replace | |

### タスク呼び出し
| Code | アクション名 |
|------|-------------|
| 130 | Perform Task |
| 129 | JavaScript |

### 通知・表示・アラート

> Tasker の「Alert」アクションカテゴリ。音・バイブ・ライト・通知・ポップアップ・読み上げを含む。
> 61/171/172/334/511/697/699/779/941 は All_Action_Alert.tsk.xml + Description で確認（2026-06-19）。

| Code | アクション名 | 備考 |
|------|-------------|------|
| 61 | Vibrate | arg0=Time(ms)。単発バイブ（パターンは 62 Vibrate Pattern） |
| 171 | Beep | arg0=Frequency, arg1=Duration, arg2=Amplitude, arg3=Stream |
| 172 | Morse | arg0=Text, arg1=Frequency, arg2=Speed, arg3=Amplitude, arg4=Stream |
| 334 | Say WaveNet | arg0=Text/SSML, arg1=Voice, arg2=Stream, arg3=Pitch, arg4=Speed |
| 511 | Torch | arg0: 0=Off, 1=On, 2=Toggle |
| 523 | Notify | LED・Vibration・Sound パラメータを内蔵。下記 525/536/538 の統合先 |
| 525 | Notify LED | **非推奨**（Notify に統合）。実機で開くと "This type of Notify action is now deprecated" 警告。今後は base Notify(523) を使う |
| 536 | Notify Vibrate | **非推奨**（Notify に統合）。同上 |
| 538 | Notify Sound | **非推奨**（Notify に統合）。同上 |
| 548 | Flash | |
| 550 | Popup | |
| 551 | Menu | |
| 552 | Popup Task Buttons | |
| 559 | Say | arg0=Text, arg1=Engine:Voice, arg2=Stream, arg3=Pitch, arg4=Speed |
| 697 | Shut Up | パラメータなし。読み上げ（TTS）を停止 |
| 699 | Say To File | arg0=Text, arg1=Engine:Voice, arg2=File(.wav), arg3=Pitch, arg4=Speed |
| 779 | Notify Cancel | arg0=Title, arg1=Number |
| 941 | HTML Popup | arg0=Code, arg1=Layout, arg2=Timeout(Seconds), arg3=Show Over Keyguard |

### プロファイル制御
| Code | アクション名 |
|------|-------------|
| 159 | Profile Status |

### アプリ操作
| Code | アクション名 |
|------|-------------|
| 18 | Kill App |
| 20 | Launch App |

### 位置情報
| Code | アクション名 | 備考 |
|------|-------------|------|
| 366 | Get Location v2 | Timeout(Seconds)・Last Location If Timeout 等。出力変数: `%gl_latitude`/`%gl_longitude`/`%gl_altitude`/`%gl_bearing`/`%gl_satellites`/`%gl_speed`/`%gl_coordinates`/`%gl_map_url` ほか。`Weather (Google).prj.xml` task153/154/155 で確認（2026-07-04） |

### 通信
| Code | アクション名 |
|------|-------------|
| 41 | Send SMS |
| 90 | Call |
| 116 | HTTP Post |
| 118 | HTTP Get |
| 339 | HTTP Request |

### クリップボード

| Code | アクション名 |
|------|-------------|
| 105 | Set Clipboard |

### ダイアログ・入力

| Code | アクション名 | 備考 |
|------|-------------|------|
| 360 | Input Dialog | 戻り値: `%input`（固定） |
| 390 | Pick Input Dialog | 戻り値: プレフィックス指定のみ（例: `input` → `%input`）。Type に Color / Directory / File 等を指定 |

#### Pick Input Dialog (390) — XMLパラメータ対応

`Dropbox.prj.xml` / `GV_Manager.prj.xml`（Type: Color/Directory/File）にて確認（2026-07-06）。

| arg | 内容 | 備考 |
| --- | ---- | ---- |
| arg0 | Bundle（RELEVANT_VARIABLES） | 出力変数の説明メタ情報。空でも動作する |
| arg1 | Type | `"File"` / `"Color"` / `"Directory"` / `"FileSystemPicker"` など |
| arg2 | Title | 自由文字列 |
| arg3 | Message | 自由文字列 |
| arg4 | （未確認・空で動作） | |
| arg5 | Timeout（秒） | `30` |

- **Type: File は拡張子フィルタなし**（どのファイルでも選択可）。実機確認済み（2026-07-06、ユーザー実機テスト）
- 出力は常に `%input`（プレフィックス変更時の格納先arg位置は未確認）
- キャンセル時は `%input` が未設定になる。**`If %input Isn't Set`（op13）で判定**する（`GV_Manager.prj.xml` act2 で確認済みの実パターン）

### ファイル操作
| Code | アクション名 | 備考 |
|------|-------------|------|
| 404 | Copy File | |
| 410 | Write File | arg0=ファイルパス, arg1=書き込むテキスト, arg2=Int（Add=追記/1でON。0だと上書きと推測・未確認）, arg3=Int（Add Newline=末尾改行/1でON）。`DocomoMail_Trigger`/`Join_Router`のデバッグログ出力で使用実績あり |
| 417 | Read File | arg0=ファイルパス, arg1=出力変数（`%var`形式）, arg2=Int（常に0、用途未確認）。`AN LOG表示`等で確認 |

### システム設定
| Code | アクション名 | 備考 |
|------|-------------|------|
| 62 | Vibrate Pattern | arg0=パターン文字列（例: "500,1000,500"）, arg1=空 |
| 113 | WiFi Tether (Hotspot) | arg0: 0=Off, 1=On |
| 192 | Play Ringtone | arg0=Type(1=Ringtone), arg1=Sound（例: "Notification"）, arg2=Stream（5=Notification） |
| 235 | Custom Setting | |
| 294 | Bluetooth | arg0: 0=Off, 1=On |
| 303 | Alarm Volume | arg0=音量レベル（0〜15）|
| 304 | Ringer Volume | arg0=音量レベル（0〜15）|
| 305 | Notification Volume | arg0=音量レベル（0〜15）|
| 306 | Call Volume | arg0=音量レベル（0〜15）|
| 307 | Media Volume | arg0=音量レベル（0〜15）|
| 310 | Vibrate Mode | arg0: 0=Off, 1=Vibrate（2=Normal は未確認）|
| 331 | Auto-Sync | arg0: 0=Off, 1=On |
| 332 | GPS | |
| 340 | Bluetooth Connection | arg1=Action(0=Connect?), arg2=デバイス名, arg3=Timeout秒, `<se>false</se>`=Continue on Error |
| 341 | Test Net | arg0=Type, arg2=結果格納変数。`arg0=5`=**Wifi SSID**（接続中SSIDを返す）。**能動プローブで即時取得**＝monitored変数 `%WIFII`(間隔更新) と違いリアルタイム。比較は `~R`。`wifi If Test`(id273) で確認（2026-06-16） |
| 425 | WiFi | |
| 443 | Media Control | arg0=コマンド番号（5=Play Simulated?）, arg1=Simulate Media Button(1=On), arg2=App |
| 512 | Status Bar | |

### データベース（SQLite）

| Code | アクション名 |
|------|-------------|
| 667 | SQL Query |

#### SQL Query (667) パラメータ詳細

```text
arg0  Int  Mode: 0=Raw, 他=Table URI/Formatted
arg1  Str  File: DBファイルパス
arg2  Str  (未使用 or Table URI)
arg3  Str  (未使用 or Selection)
arg4  Str  Query: SQL文（Raw モード時）
arg5  Str  (未使用 or Selection Args)
arg6  Str  (未使用)
arg7  Str  Output Column Divider: 列区切り文字（例: |）
arg8  Str  Variable Array: 結果格納変数（%付きで指定）
arg9  Int  Use Root: 0=Off
arg10 Int  Use Global Namespace: 1=On
```

#### SQL Query 動作メモ

- Continue Task After Error = ON → `<se>false</se>` を追加
- 成功時: `%err` は未設定（空文字）
- 失敗時: `%err` に整数が設定される（通常 "1"）
- `%err` は次のアクション実行時にリセットされるため、**直後の Variable Set で保存すること**
- SELECT 結果: `%varname(1)`, `%varname(2)`, ... に格納（各行、列は Column Divider で区切り）
- 書き込み操作（INSERT/UPDATE/DELETE）: 結果変数は空（不要なら `%dummy` を指定）

### カレンダー
| Code | アクション名 |
|------|-------------|
| 567 | Calendar Insert |

### 配列操作
| Code | アクション名 | 備考 |
|------|-------------|------|
| 354 | Array Set | |
| 393 | Arrays Merge | arg1=Names（対象配列）, Merge Type（"Format"等）, Joiner, Format, Output。`Weather (Google).prj.xml` task151 で確認（2026-07-04） |

---

## タスクレベル属性（アクションコード以外の Task 直下タグ）

| タグ | 意味 | 値 |
| ---- | ---- | --- |
| `<pri>` | タスク優先度 | 整数 |
| `<rty>` | Collision Handling（実行中に再発火した時の挙動） | なし/`0`=Abort New Task（既定）, `1`=Abort Existing Task, `2`=Run Both Together |

> `<rty>` は `<pri>` の直後に置く。既定（Abort New Task）ではタグ自体が出力されない。値は [Taskomater/Tasker-XML-Info](https://github.com/Taskomater/Tasker-XML-Info) で確認（2026-07-02、`DocomoMail_Trigger` の二重発火調査で使用）。

---

## Profile States

| Code | 状態名 | 備考 |
|------|--------|------|
| 2 | BT Status | Bluetooth ON/OFF状態 |
| 3 | BT Connected | arg0=デバイス名, arg1=アドレス（空=""はすべてのデバイス）。Description: "BT Connected [Name:* Address:*]" |
| 5 | Calendar Entry | arg0=Title, arg4=Calendar（例: "Google:仕事用"）。Description: "Calendar Entry [Title:... Calendar:...]" |
| 10 | Power | arg0: 0=Any, 1=AC, 2=USB（推定） |
| 103 | Light Level | |
| 123 | Display State | |
| 140 | Battery Level | arg0=Min, arg1=Max（Min=Max で「ちょうどN%」） |
| 150 | USB Connected | |
| 160 | WiFi Connected | |
| 165 | Variable Value | `<ConditionList>` 内の `<Condition>` で lhs/op/rhs を指定。複数条件を AND で結合可能 |
| 180 | Temperature | |

---

## Profile Events

| Code | イベント名 |
|------|-----------|
| 2 | Phone Offhook |
| 7 | Received Text |
| 208 | Display On |
| 210 | Display Off |
| 411 | Device Boot |
| 413 | Device Shutdown |
| 461 | Notification |
| 1000 | Display Unlocked（※Tasker-XML-Info では "Plugin" とも記載あり。実機XMLとDescriptionより "Display Unlocked" と確認） |

---

## HTTP Request メソッド番号（code 339）

| arg1 | メソッド |
|------|-------------|
| 0 | GET |
| 1 | POST |
| 4 | PATCH |

※ PUT / DELETE / HEAD 等は未確認。確認でき次第追記。

---

## Condition 演算子（UI表示名）

Tasker UI の「Select Condition Operator」に表示される名称と疑似コード表記の対応。
**文字列用と数値（Maths）用は別物** — 意味が同じでも必ず正しい種別を選ぶこと。

### 文字列系

| UI表示名 | 疑似コード記号 | 意味 |
| -------- | ------------- | ---- |
| Equals | `Equals` | 文字列として等しい |
| Doesn't Equal | `Doesn't Equal` | 文字列として等しくない |
| Matches | `~` | グロブパターン一致（`*` `+` が使える） |
| Doesn't Match | `!~` | グロブパターン不一致 |
| Matches Regex | `~R` | 正規表現一致 |
| Doesn't Match Regex | `!~R` | 正規表現不一致 |

### 数値系（Maths）

| UI表示名 | 疑似コード記号 | 意味 |
| -------- | ------------- | ---- |
| Maths: Equals | `Maths:=` | 数値として等しい |
| Maths: Isn't Equal To | `Maths:≠` | 数値として等しくない |
| Maths: Less Than | `Maths:<` | より小さい |
| Maths: Greater Than | `Maths:>` | より大きい |
| Maths: Is Even | `Maths:Even` | 偶数 |
| Maths: Is Odd | `Maths:Odd` | 奇数 |

### 変数状態

| UI表示名 | 疑似コード記号 | 意味 |
| -------- | ------------- | ---- |
| Is Set | `Is Set` | 変数に値がある（空でない） |
| Isn't Set | `Isn't Set` | 変数が空または未設定 |

### 使い分けの原則

- 文字列との比較（ラベル名・フラグ値・コマンド名など）→ `Equals` / `~` / `!~`
- 数値との比較（HTTPコード・カウント・日付数値・曜日番号など）→ `Maths:=` / `Maths:>` / `Maths:<` など

### XML op番号対応表

ConditionList の `<op>` 値と演算子の対応。確認済みのもののみ記載。

| op値 | 演算子 | Description表記 | 確認方法 |
| ---- | ------ | --------------- | -------- |
| 0 | Matches（グロブ） | `~` | 実機XMLとDescription対照 |
| 1 | Doesn't Equal（文字列） | （未確認） | XML文脈から推定 |
| 2 | Equals（文字列） | （未確認） | 実機XMLより |
| 7 | Maths: Greater Than | `>` | 実機XMLとDescription対照（`%gps_ago > 180`） |
| 8 | Maths: Equals | `=` | 実機XMLとDescription対照（`%DEBUG_MODE = 0`） |
| 9 | Maths: Isn't Equal To | `!=` | 実機XMLとDescription対照（`%LOC_HOME != 0`）、Tasker_Tips.md確認 |
| 12 | Is Set | `Set` | 実機XMLより |
| 13 | Is Not Set | `!Set` | 実機XMLより |

### 複数条件（AND / OR）の XML 構造

If 等の `<ConditionList>` に複数の `<Condition>` を入れる場合、先頭に `<bool0>` で結合子を指定する。
`<bool0>` は c0 と c1 の間の結合子（n番目は `<boolN>`）。

```xml
<ConditionList sr="if">
    <bool0>And</bool0>            <!-- And または Or（先頭大文字） -->
    <Condition sr="c0" ve="3">
        <lhs>%LOC_HOME</lhs>
        <op>9</op>                <!-- != -->
        <rhs>0</rhs>
    </Condition>
    <Condition sr="c1" ve="3">
        <lhs>%WK_HEADSET</lhs>
        <op>8</op>                <!-- = -->
        <rhs>0</rhs>
    </Condition>
</ConditionList>
```

- 値は `And` / `Or`（先頭大文字）。実機XML（backup.xml）で両方確認済み（2026-06-09）
- 単一条件のときは `<bool0>` 不要

---

## 疑似コード表記規則：数値 vs 文字列

### Variable Set (code 547) — XML パラメータ対応

| arg | 内容 | 値 |
| --- | ---- | -- |
| arg0 | 変数名 | `%var` |
| arg1 | To（代入値） | 文字列 / 変数 |
| arg2 | 不明（常に 0） | `0` |
| arg3 | Do Maths | `0`=Off, `1`=On |
| arg4 | Append | `0`=Off, `1`=On |
| arg5 | Structure Output | `0`=Off, `3`=On |
| arg6 | 不明（常に 1） | `1` |

※ arg4=1 (Append: On) は "To" の値を変数の現在値に**追記**する（Flash.tsk.xml にて確認済み 2026-03-31）

### JavaScriptlet (code 129) — XML パラメータ対応

| arg | 内容 | 値 |
| --- | ---- | -- |
| arg0 | Code（JSコード本体） | 文字列 |
| arg1 | 不明（実例では常に空） | `""` |
| arg2 | Auto Exit | `0`=Off, `1`=On |
| arg3 | Timeout (Seconds) | 数値（例 `45`） |

※ `DocomoMailGuard_Recv.tsk.xml`・`Sub_Notify_Ring.tsk.xml`・`Join_Router.tsk.xml` の複数実例で `arg2=1, arg3=45` を確認（2026-07-10）。JS内では `local('name')`/`setLocal('name', value)` で同一タスク内のローカル変数を読み書きできる（`%name` として後続アクションから参照可能）。

### Variable Set（疑似コード表記）

| パターン | 表記 | 備考 |
| -------- | ---- | ---- |
| 文字列リテラル | `Variable Set %x = "text"` | ダブルクォートで囲む |
| 数値リテラル | `Variable Set %x = 42` | クォートなし |
| 変数を含む文字列 | `Variable Set %x = %DIR/file_%TIME.txt` | クォートなし（変数が含まれると明白なため） |
| 数式（Do Maths ON） | `Variable Set %x = (%a + %b * 2)` | 括弧で数式と明示、Do Maths ON を意味する |

### Variable Search Replace (598) — XML パラメータ対応

backup.xml 内の複数実例（7件以上）で **arg2/arg3/arg4=0固定, arg6=1固定** を確認（2026-07-06）。
これらは実機での標準的なデフォルト設定と考えられる。

| arg | 内容 | 例 |
| --- | ---- | -- |
| arg0 | 変数名 | `%var` |
| arg1 | Search（検索パターン） | `"&"` |
| arg2 | 不明（実例では常に0） | `0` |
| arg3 | 不明（実例では常に0） | `0` |
| arg4 | 不明（実例では常に0） | `0` |
| arg5 | 不明（実例では常に空） | `""` |
| arg6 | 不明（実例では常に1） | `1` |
| arg7 | Replace（置換文字列） | `"&amp;"` |

### Variable Split / Join

```text
Variable Split %labels, Splitter: ";"      ← 区切り文字はクォートあり
Variable Join  %arr, Joiner: ","
```

### Variable Add / Subtract

常に数値演算のため、クォートなし。

```text
Variable Add %count, Value: 1
```

### Variable Section

`From` と `Length` は常に数値（位置・長さ）。

```text
Variable Section %fname, From: %bkup_fatepos, Length: 8, Store: %filedate
                                ↑ 数値変数（クォートなし）
```

### Perform Task パラメータ

| パラメータ値の種類 | 表記例 |
| -------- | ---- |
| 文字列リテラル | `par1: "autoconf"` |
| 数値 | `par1: -7` |
| 変数 | `par1: %labels(%idx)` |

### For ループ

```text
For %i, 1:10            ← 数値範囲（クォートなし）
For %item, %array()     ← 配列要素
```

---

## Scene V1 アクション（旧シーン）

> Tasker の旧シーン（V1）操作アクション。`Sub_JSON_Viewer`(V1) XML + Description で確認（2026-06-19）。
> 6.7.x では Scene V2（479〜484）への移行が進んでおり、V1 アクションは旧タスクの読解用。

| Code | アクション名 | 主なパラメータ |
| ---- | ----------- | ------------ |
| 47 | Show Scene | arg0=Name, arg1=Display As, arg2/arg3=Horizontal/Vertical Position |
| 49 | Destroy Scene | arg0=Name。`se`=Continue Task After Error |
| 51 | Element Text | arg0=Scene Name, arg1=Element, arg3=Text（要素のテキストを設定） |
| 53 | Element Web Control | arg0=Scene Name, arg1=Element, arg2=Mode（6=Load URL）, arg3=Value（例 `data:text/html;charset=utf-8,...`） |

---

## Scene V2 アクション（6.7.0-beta 以降）

> Tasker 6.7.0-beta で追加。コード 479〜484 が Scene V2 専用。
> 実機 XML（Scene_V2_Action_Code.tsk.xml）にて確認済み（2026-03-16）。

| Code | アクション名 | 主なパラメータ |
| ---- | ----------- | ------------ |
| 479 | Show Scene v2 | Screen ID, JSON, Display Mode, Blocking Overlay |
| 480 | Dismiss Scene v2 | Screen ID |
| 481 | Update Scene v2 | Screen ID, Element ID（1要素ずつ更新） |
| 482 | Update Scene v2 Overlay | Screen ID, Blocking Overlay, Transition Duration (ms) |
| 483 | Get Scene v2 Element Value | Screen ID, Element ID → `%sv2_value()`, `%sv2_element_id()`, `%sv2_result_count` |
| 484 | Wait For Scene v2 Result | Screen ID → `%sv2_value()`, `%sv2_element_id()`, `%sv2_result_count` |

### Show Scene v2 (479) — XMLパラメータ対応

| arg | 内容 | 例 |
| --- | ---- | -- |
| arg0 | Bundle（RELEVANT_VARIABLES） | 空でも動作 |
| arg1 | シーン名 or JSON 文字列 | `"SV2_Webview"` / `{"root":{...},"name":"銭湯設定"}` |
| arg2 | Screen ID | `"%screen_id"`（空=シーン自身のID） |
| arg3 | Display Mode | `"Fullscreen"` / `"Dialog"` |
| arg4 | Overlay Width（Overlay時のみ） | `"50%"` |
| arg5 | Overlay Height（Overlay時のみ） | `"50%"` |
| arg6, arg7 | 不明（Fullscreen/Dialogでは空） | `""` |
| arg8 | Blocking Overlay（1=On） | `1` |
| arg9〜arg11 | 不明（実例では常に空） | `""` |
| arg12 | 不明（実例では0） | `0` |
| arg13 | 不明（実例では0または1） | `0` |
| arg14 | 不明（実例では0または1） | `1` |
| arg15 | 不明（実例では常に空） | `""` |
| arg16 | 不明（tv 6.7.5-beta以降で追加？tv 6.7.3-betaの実例には無し） | `1` |

実機確認元: `Sento_Open.tsk.xml`（tv 6.7.3-beta, arg15まで）、`Sub_Webview.tsk.xml` / `Scene_V2_Action_Code.tsk.xml`（tv 6.7.5-beta, arg16あり）（2026-07-06）

### Get Scene v2 Element Value (483) — 戻り値

- `%sv2_value()` — 取得した値（配列。単一取得時は `%sv2_value(1)` で参照）
- `%sv2_element_id()` — 取得した要素のID
- `%sv2_result_count` — 結果件数
- 読み取り可能な要素: TextInput（入力テキスト）、Switch（on/off）など

### Update Scene v2 (481) — 注意事項

- **Element ID を指定して1要素ずつ更新**（Variables のバッチ更新ではない）
- グローバル変数はシーンに自動反映されるため Update Scene v2 不要
- ボタン色等のローカル変数更新のみ必要な場合に使用

#### Update Scene v2 (481) — XMLパラメータ対応（単一要素プロパティ更新モード）

`Sub_Webview.tsk.xml` act8/act9 にて確認（2026-07-06）。WebView の `content` プロパティを
動的に流し込む用途で実際に使われている（本パターンを MD_Preview でも踏襲）。

| arg | 内容 | 例 |
| --- | ---- | -- |
| arg0 | Screen ID | `%scene_id` |
| arg1 | Element ID | `"webview1"` |
| arg2 | Property名 | `"content"` |
| arg3 | 値 | `%final_html` |
| arg4 | 不明（実例では0） | `0` |
| arg5 | 不明（実例では空） | `""` |

---

## プラグインコード

> ※ コードはTaskerバージョン・プラグインバージョンで変わる可能性あり。Test_Plugin.tsk.xml（6.7.1-beta）で確認済み（2026-04-05）。

### AutoAppsHub（com.joaomgcd.autoappshub）

| Code | アクション名 | plugintypeid |
| ---- | ----------- | ------------ |
| 502807688 | AutoAppsHub SendCommand | IntentSendCommand |

### AutoContacts（com.joaomgcd.autocontacts）

| Code | アクション名 | plugintypeid |
| ---- | ----------- | ------------ |
| 211905330 | AutoContacts | IntentQueryContacts |
| 429032033 | AutoContacts Details | IntentGetContactDetails |
| 1452528931 | AutoContacts Query 2.0 | IntentQuery2 |

### AutoInput（com.joaomgcd.autoinput）

| Code | アクション名 | plugintypeid |
| ---- | ----------- | ------------ |
| 107361459 | AutoInput Actions v2 | IntentActionv2 |
| 1732635924 | AutoInput Action | IntentPerformAction |
| 778682267 | AutoInput Gestures | IntentGestures |
| 811079103 | AutoInput Global Action | IntentPerformGlobalAction |
| 921575593 | AutoInput Keyguard | IntentKeyguardAction |
| 1687767515 | AutoInput Modes | IntentModes |
| 2110921373 | AutoInput Root | IntentIssueCommand |
| 2146575103 | AutoInput Screen Capture | IntentScreenCapture |
| 1250249549 | AutoInput Screen Off/On | IntentTurnOffScreen |
| 1311526850 | AutoInput Settings | IntentSettings |
| 1040876951 | AutoInput UI Query | IntentUIQuery |
| 234244923 | AutoInput Unlock Screen | IntentUnlockScreen |

### AutoLaunch（com.joaomgcd.autoapps）

| Code | アクション名 | plugintypeid |
| ---- | ----------- | ------------ |
| 1795842217 | AutoLaunch | IntentLaunchApp |
| 2114100406 | AutoLaunch Query | IntentRequestQueryApps |

### AutoLocation（com.joaomgcd.autolocation）

| Code | アクション名 | plugintypeid |
| ---- | ----------- | ------------ |
| 1469986059 | AutoLocation Activities | IntentRequestActivityReport |
| 1879487834 | AutoLocation Geofences | IntentRequestGeofenceReport |
| 96135575 | AutoLocation Info | IntentGetInfo |
| 725543659 | AutoLocation Location | IntentRequestLocationReport |
| 501524174 | AutoLocation Manage | IntentManageGeofence |
| 1331972225 | AutoLocation Map | IntentMap |
| 411134516 | AutoLocation Mock Location | IntentMockLocation |
| 1670648462 | AutoLocation Orientation | IntentOrientationService |

### AutoNotification（com.joaomgcd.autonotification）

#### Task Actions

| Code | アクション名 | plugintypeid |
| ---- | ----------- | ------------ |
| 166160670 | AutoNotification Create | IntentNotification |
| 1677547919 | AutoNotification Actions | IntentNotificationInterceptActions |
| 1563355455 | AutoNotification Buttons Notification | IntentNotificationButtons |
| 2046367074 | AutoNotification Cancel | IntentCancelNotification |
| 565385068 | AutoNotification Query | IntentNotificationQuery |
| 563213414 | AutoNotification Table | IntentNotificationTable |

#### Profile Event（プロファイルのイベントトリガーとして使用）

| Code | イベント名 | plugintypeid | 変数 |
| ---- | --------- | ------------ | ---- |
| 1825107102 | AutoNotification Command Event | IntentCommandEvent | `%ancomm`（コマンド）, `%anpar`（パラメータ）, `%anmessage`（メッセージ） |
| 1520257414 | AutoNotification (Notification Intercept) Event | IntentInterceptNotificationEvent | `%antitle`（タイトル）, `%antext`（本文）ほか。通知ID/キー系変数（`%anid`/`%ankey` 等）は実機イベント設定の「Variables」タブで要確認。`DocomoMail_Notify`(prof402) で使用 |

### AutoRemote（com.joaomgcd.autoremote）

| Code | アクション名 | plugintypeid |
| ---- | ----------- | ------------ |
| 1137141958 | AutoRemote Bluetooth | IntentBluetoothService |
| 837429111 | AutoRemote Query | IntentQuery |

### AutoShare（com.joaomgcd.autoshare）

| Code | アクション名 | plugintypeid |
| ---- | ----------- | ------------ |
| 940160580 | AutoShare | IntentShare |
| 319692633 | AutoShare Process Text | IntentProcessTextAction |

### AutoTools（com.joaomgcd.autotools）

| Code | アクション名 | plugintypeid |
| ---- | ----------- | ------------ |
| 1165325195 | AutoTools Web Screen | IntentWebScreen |
| 1508929357 | AutoTools Arrays | IntentSortArrays |
| 1304982781 | AutoTools Dialog | IntentDialog |
| 1912522764 | AutoTools Toast | IntentToast |

### AutoVoice（com.joaomgcd.autovoice）

| Code | アクション名 | plugintypeid |
| ---- | ----------- | ------------ |
| 1754437993 | AutoVoice Recognize | IntentGetVoice |

### Join（com.joaomgcd.join）

| Code | アクション名 | plugintypeid |
| ---- | ----------- | ------------ |
| 864692752 | Join Query Devices | IntentQueryDevices |
| 1668911626 | Join Push Received（Event） | IntentReceivedPush |

### Tasker SQLite Plugin（com.jordanhotmann.taskersqliteplugin）

| Code | アクション名 |
| ---- | ----------- |
| 1572765422 | Tasker SQLite Plugin |

### Termux:Tasker（com.termux.tasker）

| Code | アクション名 |
| ---- | ----------- |
| 1256900802 | Termux:Tasker（スクリプト実行） |

### Enhanced SMS & Caller ID — Gmail（com.enhancedsmscallerid.setting.gmail）

| Code | アクション名 |
| ---- | ----------- |
| 1254573230 | Gmail 送信（Run SL4A Script 系） |

### Kustom（org.kustom.wallpaper）

| Code | アクション名 |
| ---- | ----------- |
| 703953103 | Kustom Variable Name Set |

---

### AutoNotification Create の主要 Bundle キー

| キー | 型 | 説明 |
| ---- | -- | ---- |
| `config_notification_title` | String | 通知タイトル |
| `config_notification_text` | String | 通知本文 |
| `notificaitionid` | String | 通知ID（スペルミス注意: `notification` ではなく `notificaition`） |
| `config_notification_icon` | String | アイコン（Drawable名 **または URL**）。URLを指定するとAutoNotificationが画像をDLして通知アイコンとして表示する |
| `config_notification_persistent` | Boolean | 永続通知 |
| `config_notification_vibration` | String | バイブパターン（カンマ区切りms） |
| `SoundPath` | String | 通知音ファイルパス |
| `BackgroundColor` | String | 背景色（#AARRGGBB 形式） |
| `UseHTML` | Boolean | HTML表示を使用するか |
| `StatusBarIconString` | String | ステータスバー小アイコン（Drawable名）。`null` のとき `preferenceScreen` の値が使われる |
| `preferenceScreen` | String | ステータスバーアイコン（Drawable名）。`config_notification_icon` にURLを指定した場合は別途ここでDrawable名を指定する必要がある |
| `NotificationChannelId` | String | 通知チャンネルID |

---

## Profile XML 要素マッピング

> アクションコード（`<code>`）とは別物。`<Profile>` 要素・`<State>`/`<Event>` 要素に付くフィールド。
> 充電管理 V2 プロファイルの実機エクスポートと Description の比較により確認（2026-06-04）。

### Profile 要素のフィールド

| XMLフィールド | Tasker UIでの表示 | 値 | 確認度 |
| ----------- | --------------- | -- | ------ |
| `<clp>true</clp>` | Settings: **Restore: yes** | true=復元する | ✓ 確実（ディスプレイ_Off.prf.xml + Description対照） |
| `<clp>`なし | Settings: **Restore: no** | 既定値 | ✓ 確実 |
| `<cldm>N</cldm>` | Settings: **Cooldown: N** (分) | N=分数 | ✓ 確実（ディスプレイ_Off.prf.xml + Description対照） |
| `<limit>true</limit>` | 不明（Description では "Restore: yes" と表示されるが `<clp>` も同様）| — | ? 要実機確認。旧バージョンの "Restore" タグ、または State の "Limit Entry Task" オプションの可能性。Geofences_LIFE は `<clp>` と `<limit>` を両方持つ |
| `<flags>8</flags>` | 不明（ビットフラグ）| bit3=8の意味は未確認 | ? 要調査（Clock_calendar プロファイルで確認） |

### State 要素のフィールド

| XMLフィールド | Tasker UIでの表示 | 意味 |
| ----------- | --------------- | ---- |
| `<pin>true</pin>` | 条件名の先頭に **Not** が付く | 条件を反転（NOT）する |
| `<pin>`なし | 通常の条件 | 反転なし |

> ⚠️ `<pin>` はあらゆる State 条件（Power・Battery Level・WiFi 等）に存在しうる。
> XML を手書きする際は反転したい条件に `<pin>true</pin>` を忘れずに追加すること。

### Power State (code=10) の arg0 マッピング

| arg0 | Tasker UIでの表示 |
| ---- | --------------- |
| `0` | Source: Any（いずれかの充電源） |
| `1` | Source: AC |
| `2` | Source: USB（推定・未確認） |

充電管理 開始（Power Any）: `arg0=0`、充電管理 中断（Not Power AC）: `arg0=1` + `<pin>true</pin>` で確認済み。

### Battery Level State (code=140) のパラメータ

| arg | 内容 |
| --- | ---- |
| `arg0` | Min（最小値） |
| `arg1` | Max（最大値） |

Min=Max にすると「ちょうどN%」の条件になる（例: Min=80 Max=80）。

### Calendar Entry State (code=5) のパラメータ

```xml
<State sr="con0" ve="2">
    <code>5</code>
    <Str sr="arg0" ve="3">Meeting:*</Str>   <!-- Title（グロブ可） -->
    <Str sr="arg1" ve="3"/>                  <!-- Location -->
    <Str sr="arg2" ve="3"/>                  <!-- Description -->
    <Int sr="arg3" val="0"/>                 <!-- Available（0=Any） -->
    <Str sr="arg4" ve="3">Google:仕事用</Str> <!-- Calendar名 -->
    <Str sr="arg5" ve="3"/>                  <!-- Start Early (Minutes) -->
    <Str sr="arg6" ve="3"/>                  <!-- End Later (Minutes) -->
</State>
```

### Variable Value State (code=165) の構造

```xml
<State sr="con0" ve="2">
    <code>165</code>
    <ConditionList sr="if">
        <Condition sr="c0" ve="3">
            <lhs>%LOC_HOME</lhs>
            <op>9</op>    <!-- Maths: Isn't Equal To -->
            <rhs>0</rhs>
        </Condition>
    </ConditionList>
</State>
```

複数条件は `<Condition sr="c1">` を追加し、AND で結合される。

### Time トリガーの XML 構造

```xml
<Time sr="con0">
    <fh>10</fh>      <!-- From Hour -->
    <fm>0</fm>       <!-- From Minute -->
    <rep>2</rep>     <!-- Repeat type（2=分間隔） -->
    <repval>30</repval> <!-- Repeat value（例: 30分ごと） -->
    <th>12</th>      <!-- To Hour -->
    <tm>0</tm>       <!-- To Minute -->
</Time>
```

### Location（Geofence）トリガーの XML 構造

```xml
<Loc sr="con0">
    <cname>場所名</cname>     <!-- Tasker内の場所名 -->
    <lat>35.69013</lat>       <!-- 緯度 -->
    <long>139.80212</long>    <!-- 経度 -->
    <rad>100.0</rad>          <!-- 半径（メートル） -->
</Loc>
```

### ProfileVariable（プロファイルローカル変数）の XML 構造

```xml
<ProfileVariable sr="pv0">
    <pvn>%prof_var</pvn>   <!-- 変数名 -->
    <pvt>t</pvt>           <!-- 型（t=text） -->
    <pvci>true</pvci>      <!-- Clear on entry？ -->
    <strout>true</strout>  <!-- Entry タスクに渡す -->
    <clearout>false</clearout>
</ProfileVariable>
```

Description 表示例: `Variables: [ %prof_var:has value ]`

---

## 注意事項

- 500番台には廃止済み（Deprecated）のコードも含まれる
- プラグイン（AutoNotification, AutoTools等）は別途長いコード番号を持つ
- コード番号はTaskerのバージョンによって追加される場合がある
- Profile XML 要素（`<clp>` `<cldm>` `<pin>` `<limit>` 等）はアクションコードとは別物（上のセクション参照）
- `<limit>true</limit>` の正確な意味は未確定（要実機確認）
