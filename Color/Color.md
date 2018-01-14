## 透明度转换

 > 1. 颜色透明度区间（0-255）, 0 代表全透明, 255 代表不透明
 > 2. 255 X 0.4 = 102（示例：40%透明度，这里得到的 102 是10进制）
 > 3. 102 转 16进制 得到：66
 > 4. 将不透明度和颜色拼接成ARGB格式，如：#66000000
 
 ```
 100% — FF
 95% — F2
 90% — E6
 85% — D9
 80% — CC
 75% — BF
 70% — B3
 65% — A6
 60% — 99
 55% — 8C
 50% — 80
 45% — 73
 40% — 66
 35% — 59
 30% — 4D
 25% — 40
 20% — 33
 15% — 26
 10% — 1A
 5% — 0D
 0% — 00
 ```