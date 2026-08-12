# OpenCV ORB特徴点マッチング Tips

最終更新: 2026-08-12

## 反復パターンの多い画像で誤対応点が多発し、正しいペアが棄却される問題

### 症状
ORB特徴点 + BFMatcher + Loweの比率テストで「良いマッチ」を抽出した後、
全マッチの(dx, dy)オフセットの標準偏差だけで一致/不一致を判定すると、
経緯線・格子模様・類似形状のラベルなど反復パターンが多い画像では、
正しい対応点に大量の誤対応点が混入し、標準偏差が閾値を大きく超えて
ペア自体が「不一致」として棄却されることがある。中央値自体は正しい値を
示していても、判定に使う標準偏差だけが外れ値で暴れてしまう。

### 原因
Loweの比率テストは「1位と2位の候補の距離差」をチェックするだけなので、
反復パターンでは複数の類似候補が拮抗しやすく、誤対応点でも比率テストを
通過してしまう。結果、正しい対応点と誤対応点が「良いマッチ」に混在する。

### 対処法: ビン投票（多数決）による頑健な外れ値除去
1. 各マッチの (dx, dy) を一定サイズ（例: 6px）のビンに丸める
2. 最多得票のビン（+隣接1ビン程度）に属するマッチだけを「インライア」として抽出
3. インライアのみで中央値・標準偏差を計算し、棄却判定もインライアに対して行う

並進移動のみを仮定できる場合の簡易RANSAC/Hough変換的手法。
回転・スケールも考慮する一般的なケースでは `cv2.findHomography(RANSAC)` や
`cv2.estimateAffinePartial2D` が使える。

### 実装例（numpy）

```python
def select_dominant_offset_inliers(offsets: np.ndarray, bin_size: float, neighbor: int) -> np.ndarray:
    bins = np.round(offsets / bin_size).astype(int)
    unique_bins, counts = np.unique(bins, axis=0, return_counts=True)
    dominant_bin = unique_bins[np.argmax(counts)]
    inlier_mask = np.all(np.abs(bins - dominant_bin) <= neighbor, axis=1)
    return offsets[inlier_mask]
```

### 適用例
kklab Tools/src/stitch/stitch.py（複数画像をORBマッチングで自動合成するツール）。
世界地図画像（経緯線・類似国境が多数）でこの問題が顕在化し、本手法で解決した。
