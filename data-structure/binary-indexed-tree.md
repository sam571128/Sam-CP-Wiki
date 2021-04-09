---
description: 能操作前綴又能快速詢問，程式碼不到十行，好樹不刻嗎？
---

# BIT \(Binary Indexed Tree\)

### 什麼是 BIT 呢？

BIT \(Binary Index Tree\) 是一個在1994年，由 **Peter M. Fenwick** 所提出的一種資料結構，在國外也常直接叫這個資料結構 「**Fenwick Tree**」，而對岸會叫他「樹狀樹組」，他能夠很有效率的解決需要前綴操作的問題與詢問，並將其降至 $$O(\log n) $$ 

### 數字的lowbit?

在介紹 BIT 的原理與實作前，需要先知道一個數字的「**lowbit**」是什麼，因為他是這個資料結構的核心





