---
description: 區間建邊？那就用線段樹啊！
---

# 線段樹優化建圖

考慮[這個問題](https://codeforces.com/problemset/problem/786/B)（Codeforces 786B)

> 給定一張 $$n$$ 個點的有向圖，接下來有以 $$q$$ 次加邊的操作
>
> 每次操作會是以下三種
>
> * $$1\ u \ v \ w$$ :從 $$u$$ 到 $$v$$ 建一條權重為 $$w$$ 的邊。
> * $$2\ u \ l \ r \ w$$ :從 $$u$$ 到 $$[l,r]$$ 區間內所有點建一條權重為 $$w$$ 的邊。
> * $$3\ v \ l \ r \ w$$ :從  $$[l,r]$$ 區間內所有點到 $$u$$ 建一條權重為 $$w$$ 的邊。
>
> 輸出從原點 $$s$$ 到所有點的最短路徑長
>
> * $$1 \le n, \ q  \le 10^5$$&#x20;





