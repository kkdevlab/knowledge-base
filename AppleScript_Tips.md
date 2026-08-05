# AppleScript Tips

AppleScriptでの自動化スクリプト開発で発生したエラーと対処法をまとめる。

---

## 1. do shell script内でコマンドをバックグラウンド化する際、disownを使うとハングする

### 症状

```applescript
do shell script "caffeinate -di & disown"
```

このように`&`でバックグラウンド化した後`disown`を呼ぶと、`do shell script`が制御を返さず、呼び出し元のAppleScriptアプリ全体がフリーズする（120秒待っても応答が返らない）。GUIアプリの場合、Dockの「終了」メニューも効かなくなり、強制終了しかできなくなる。

### 原因

`disown`は対話シェルのジョブ制御機能に依存するコマンドで、`do shell script`が使う非対話シェルではジョブ制御が無効なため機能しない。`disown`自体がエラーになり、シェルコマンド全体が正常に終了しないため、`do shell script`が制御を返せなくなる。

### 解決方法

`disown`を使わず、標準入出力（stdin/stdout/stderr）を全て`/dev/null`にリダイレクトしてからバックグラウンド化する。

```applescript
do shell script "caffeinate -di > /dev/null 2>&1 < /dev/null &"
```

これなら`do shell script`は即座に制御を返し、バックグラウンドプロセスは親プロセスの入出力に依存せず動き続ける。

### 備考

AppleScriptから任意のコマンドをバックグラウンド実行したい場合は、常にこの「標準入出力の全リダイレクト＋`&`」の形にするとよい。

---

## 2. System Eventsのshut down/restartコマンドはstate saving preferenceを省略すると常に状態保存される

### 症状

macOSのシステム設定で「ログイン時にウィンドウを再度開く」を常にオフにしていても、`tell application "System Events" to shut down`でシャットダウンすると、次回起動時に直前のウィンドウ（開いていたアプリ）が復元されてしまう。

### 原因

`sdef`コマンドで`System Events`のスクリプティング定義を確認すると、`shut down`コマンドに`state saving preference`という明示的なbool型オプションパラメータが存在する。

```
$ sdef /System/Library/CoreServices/System\ Events.app | grep -A10 '"shut down"'

<command name="shut down" code="fndrshut" description="Shut Down the computer">
    <parameter name="state saving preference" code="stsv" type="boolean" optional="yes"
               description="Is the user defined state saving preference followed?">
    <documentation>
        If "state saving preference" is omitted or false, state is always saved.
    </documentation>
</command>
```

つまり、このパラメータを省略（またはfalseに）すると、ユーザーの実際のシステム設定を無視して**常に状態保存が行われる**、という仕様になっている。

### 解決方法

パラメータを明示的に`true`にすることで、ユーザーの実際のシステム設定（状態保存する/しない）に従うようになる。

```applescript
tell application "System Events" to shut down state saving preference true
```

### 備考

- 同様の仕組みは`restart`・`log out`コマンドにも存在する可能性がある（未検証）
- AppleScriptで意図しない挙動に遭遇した際は、推測で原因を決めつけず、`sdef <アプリのパス>`でそのアプリの正式なコマンド仕様を確認するとよい
