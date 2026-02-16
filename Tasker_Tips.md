# Tasker Tips

Tasker の仕様・ノウハウをまとめたリファレンス。

---

## 変数の型と比較

### 変数に型はない

Tasker の変数は厳密な型（Integer や String など）を持たない。
値が `10` の場合、使用する演算子によって「数値の10」にも「文字列の"10"」にもなる。

- **数値として比較**: `%count` `Maths: Greater Than` `5` → 数値比較
- **文字列として比較**: `%count` `Matches` `10` → パターンマッチ（文字列比較）

### 演算子による挙動の違い

| 種類 | 演算子例 | 解説 |
|------|---------|------|
| **数値比較** | Maths: Equals (=), Maths: Greater Than (>) | 両方の値を数値として評価。`01` と `1` は等しい |
| **パターンマッチ** | Matches (~), Doesn't Match (!~) | 文字列として比較。ワイルドカード `*` が使える。`01` と `1` は一致しない |
| **正規表現** | Regex Matches (~R) | 高度な文字列検索 |

### 条件演算子一覧

| 演算子 | 略称 |
|--------|------|
| Equals | EQ |
| Doesn't Equal | NEQ |
| Matches | ~ |
| Doesn't Match | !~ |
| Matches Regex | ~R |
| Doesn't Match Regex | !~R |
| Maths: Less Than | < |
| Maths: Greater Than | > |
| Maths: Equals | = |
| Maths: Isn't Equal To | != |
| Maths: Is Even | EVEN |
| Maths: Is Odd | ODD |
| Is Set | SET |
| Isn't Set | !SET |

### 注意事項

- **If 内で直接計算はできない**: `If [ %value + 1 > 10 ]` は不可
- **解決策**: 先に Variable Set（Do Maths チェック）で計算し、結果を比較に使う
- **未定義変数の数値比較**: 変数が空の状態で数値比較するとエラーになる場合がある。事前に初期値を設定すること

### ベストプラクティス

- 数値比較（「〜より大きい」「〜以下」など）→ **Maths: ...** 系を使う
- 特定の文字と一致するか → **Matches** を使う
