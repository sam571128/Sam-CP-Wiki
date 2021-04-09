---
description: 能操作前綴又能快速詢問，程式碼不到十行，好樹不刻嗎？
---

# BIT \(Binary Indexed Tree\)

### 什麼是 BIT 呢？

BIT \(Binary Index Tree\) 是一個在1994年，由 **Peter M. Fenwick** 所提出的一種資料結構，在國外也常直接叫這個資料結構 「**Fenwick Tree**」，而對岸會叫他「樹狀樹組」，他能夠很有效率的解決需要前綴操作的問題與詢問，跟「線段樹」一樣都能將操作與詢問降至 $$O(\log n)$$ ，而 BIT 能解決的問題都能使用線段樹完成，那我們為什麼需要 BIT 呢？繼續看下去就知道了。

