---
description: 如果你認為他只能做到單點修改，區間詢問，那麼就太少了！
---

# 線段樹（二）

在前一個章節，我們學會了單點修改、區間詢問的線段樹

{% page-ref page="segment-tree-1.md" %}

而現在呢？如果我們有一道問題如下

> 在一個陣列中尋找第一個出現的大於 $$x$$ 的位置，同時也會有修改元素的操作

這樣的問題，線段樹也辦得到！如果要尋找第一個數字，該怎麼做呢？

我們可以試著用一棵區間詢問最大值的線段樹來幫我們做到這一點

![&#x4E00;&#x500B;&#x7DAD;&#x8B77;&#x5340;&#x9593;&#x6700;&#x5927;&#x503C;&#x7684;&#x7DDA;&#x6BB5;&#x6A39;](../.gitbook/assets/segment-tree-bs-1-.jpg)

當我們建立完一棵區間最大值線段樹之後，我們可以在樹上去尋找我們要的位置，如下圖

![&#x7DDA;&#x6BB5;&#x6A39;&#x4E0A;&#x4E8C;&#x5206;&#x641C;&#xFF08;&#x5716;&#x70BA;&#x641C;&#x5C0B;4&#x7684;&#x60C5;&#x6CC1;&#xFF09;](../.gitbook/assets/segment-tree-2-1-.jpg)

這樣的動作一般稱為「線段樹上二分搜」，如同圖上，我們先從最上面那個點開始找

大致操作如下

{% tabs %}
{% tab title="C++" %}
```cpp
int find(int x, int idx, int l, int r){
    if(l==r) return l; //找到了就回傳
    int m = (l+r)/2;
    
    //往兩邊找，如果左邊的最大值>=x，往左繼續找
    //否則往右找
    if(tr[idx] >= x) find(x, idx*2, l, m); 
    else find(x, idx*2+1, m+1, r);
}
```
{% endtab %}
{% endtabs %}

