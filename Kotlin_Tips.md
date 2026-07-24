# Kotlin Tips

Kotlin開発で発生したエラーと対処法をまとめる。

---

## 1. InputStream.read()の戻り値(Int)とByte定数を直接==比較できない

### 症状

`InputStream.read()`は1バイトを`Int`型（0〜255、EOF時は-1）で返すが、これを`Byte`型の定数と直接`==`比較しようとするとコンパイルエラーになる。

```kotlin
private const val LF = '\n'.code.toByte()  // Byte型

val b = input.read()  // Int型
if (b == LF) { ... }  // コンパイルエラー
```

```
Operator '==' cannot be applied to 'Int' and 'Byte'.
```

### 原因

Kotlinは`Int`と`Byte`の間の暗黙の型変換を行わないため、異なる数値型同士の`==`比較は許可されない。

### 解決方法

比較する定数側を`Int`型で持つ（`.code`のみでByteへの変換をしない）。

```kotlin
private const val LF = '\n'.code  // Int型のまま保持

val b = input.read()
if (b == LF) { ... }  // OK
```

`ByteArray`の要素（`Byte`型）と比較したい場合は、逆にその場で`.toByte()`変換する。

```kotlin
if (bytes.last() == CR.toByte()) { ... }
```

### 備考

ソケット・ストリームをバイト単位で自前パースする実装（IMAP等の独自プロトコルクライアント等）でよく遭遇する。`read()`の戻り値が`Int`である設計はJavaのIO APIに由来する（EOFを`-1`で表現するため）。

## 2. Kotlinで定義したinterfaceをラムダで生成するにはfun interfaceにする

### 症状

自作の単一抽象メソッドinterfaceに対して`MyInterface { ... }`のようなSAM変換構文を使うとコンパイルエラーになる。

### 原因

Kotlin(1.4+)のSAM変換は、Java由来のinterfaceか、`fun interface`として宣言されたKotlin interfaceにしか適用されない。ただの`interface`ではラムダから直接インスタンス化できない。

### 解決方法

単一抽象メソッドのinterfaceは`fun interface`として宣言する。

```kotlin
fun interface AiJudge {
    fun judgeBatch(headers: List<MailHeader>): List<JudgmentResult>
}
```

これにより`AiJudge { headers -> ... }`の形でテスト用スタブ等を簡潔に書けるようになる。既存の`class Foo : AiJudge { ... }`形式の実装には影響しない。
