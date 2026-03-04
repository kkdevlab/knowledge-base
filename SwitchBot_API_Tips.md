# SwitchBot API Tips

SwitchBot Web API v1.1 に関する情報まとめ。

---

## API 概要

- **ベースURL**: `https://api.switch-bot.com/v1.1/`
- **認証方式**: HMAC-SHA256 署名（Open Token + Secret Key）
- **レート制限**: 10,000 回/日
- **利用規約**: 個人用途のみ（商用利用禁止）

---

## 認証ヘッダーの生成

### 必要な情報

- **Open Token**: SwitchBot アプリの開発者オプションから取得
- **Secret Key**: 同上

### トークン取得手順

1. SwitchBot アプリ v6.24 以降を起動
2. プロフィール → 設定 →「アプリバージョン」を **10〜15 回タップ**
3. 「開発者向けオプション」が表示される
4. トークンとシークレットキーをコピー

### 署名生成アルゴリズム

```
timestamp = 現在時刻（ミリ秒・13桁の整数）
nonce     = UUID（任意のランダム文字列）
message   = token + timestamp + nonce
signature = Base64(HMAC-SHA256(secret, message))
```

### HTTPリクエストヘッダー

```
Authorization: {token}
sign: {signature}
t: {timestamp}
nonce: {nonce}
Content-Type: application/json; charset=utf8
```

---

## 主なエンドポイント

### デバイス一覧

```
GET /v1.1/devices
```

レスポンス: `deviceList`（物理デバイス）と `infraredRemoteList`（赤外線仮想デバイス）の配列

### デバイス状態取得

```
GET /v1.1/devices/{deviceId}/status
```

### コマンド送信

```
POST /v1.1/devices/{deviceId}/commands
Body: {"command": "turnOn"}
```

成功時: `statusCode: 100`

---

## デバイス別コマンド

### Plug / Plug Mini

| コマンド | 動作 |
|---------|------|
| `turnOn` | 電源 ON |
| `turnOff` | 電源 OFF |
| `toggle` | 電源切替 |

ステータス取得時の主フィールド:

| フィールド | 内容 |
|----------|------|
| `power` | `"on"` または `"off"` |
| `voltage` | 電圧 (V) |
| `weight` | 消費電力 (W) |
| `electricCurrent` | 電流 (A) |
| `electricityOfDay` | 当日消費電力量 (Wh) |

### Bot

| コマンド | 動作 |
|---------|------|
| `press` | ボタンを押す（プッシュモード） |
| `turnOn` | ON（スイッチモード） |
| `turnOff` | OFF（スイッチモード） |

---

## できること・できないこと

### できること

- Hub 経由でのリモートデバイス制御（インターネット越し可）
- Plug の電源 ON/OFF と電力監視
- Bot の物理ボタン押し操作
- デバイスの状態リアルタイム取得
- 赤外線家電の操作（Hub が IR 対応の場合）

### できないこと

- Hub なしのリモート制御
- 商用利用（利用規約禁止）
- 10,000 回/日を超えるリクエスト
- WebhookによるリアルタイムPush通知（別途設定が必要）
- 複数デバイスへの一括コマンド送信（1リクエスト1デバイス）

---

## Tasker からの利用（HMAC-SHA256 生成）

Tasker 標準には HMAC 機能がないため、**Java Code アクション（Tasker v6.6 以降・BeanShell）** を使う。

> **JavaScriptlet は使えない**: Tasker の JS エンジンは完全にサンドボックス化されており、
> `java.*` / `javax.*` / `android.*` / `Packages.*` / `Java.type()` にはアクセス不可（ReferenceError）。

```java
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import android.util.Base64;
import java.util.UUID;

String token  = tasker.getVariable("SB_TOKEN");
String secret = tasker.getVariable("SB_SECRET");
String timestamp = String.valueOf(System.currentTimeMillis());
String nonce  = UUID.randomUUID().toString();

Mac mac = Mac.getInstance("HmacSHA256");
SecretKeySpec keySpec = new SecretKeySpec(secret.getBytes("UTF-8"), "HmacSHA256");
mac.init(keySpec);
byte[] sigBytes = mac.doFinal((token + timestamp + nonce).getBytes("UTF-8"));
String sig = Base64.encodeToString(sigBytes, Base64.NO_WRAP);

tasker.setVariable("auth_json",
    "{\"timestamp\":\"" + timestamp
    + "\",\"nonce\":\"" + nonce
    + "\",\"signature\":\"" + sig + "\"}");
```

### Java Code での変数 API

| API | 説明 |
| --- | --- |
| `tasker.getVariable("SB_TOKEN")` | グローバル変数取得（大文字名 = グローバル） |
| `tasker.setVariable("auth_json", value)` | ローカル変数にセット（小文字名 = ローカル） |

- Java Code 後に `Return: %auth_json` で呼び出し元に返す
- 呼び出し元では `Perform Task → Return Variable: %sb_auth` で受け取り、`%sb_auth.timestamp` 等のドット記法でアクセス可

---

## 参考リンク

- [公式 GitHub](https://github.com/OpenWonderLabs/SwitchBotAPI)
- [トークン取得方法](https://support.switch-bot.com/hc/en-us/articles/12822710195351-How-to-Obtain-a-Token)
