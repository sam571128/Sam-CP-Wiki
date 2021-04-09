---
description: 能操作前綴又能快速詢問，程式碼不到十行，好樹不刻嗎？
---

# BIT \(Binary Indexed Tree\)

### 什麼是 BIT 呢？

BIT \(Binary Index Tree\) 是一個在1994年，由 **Peter M. Fenwick** 所提出的一種資料結構，在國外也常直接叫這個資料結構 「**Fenwick Tree**」，而對岸會叫他「樹狀樹組」，他能夠很有效率的解決需要前綴操作的問題與詢問，並將其降至 $$O(\log n) $$ 

### 數字的lowbit?

在介紹 BIT 的原理與實作前，需要先知道一個數字的「**lowbit**」是什麼，因為他是這個資料結構的核心，而他的定義為「**數字轉成二進位後，最後一個 1 的數值**」。就只有這句話可能有點難懂，讓我們實際來看一下

**範例：**

1. $$5 = (101)_{binary} $$ ，那麼 $$lowbit(5) = (1)_{binary} = 1$$

2. $$10 = (1010)_{binary}$$ ，那麼 $$lowbit(10) = (10)_{binary} = 2$$ 

### 程式上要怎麼去找到數字的lowbit呢？

找lowbit的的結論很簡單，就是短短一行 $$x\&(-x)$$ ，至於這行是什麼意思呢

$$-x$$ 的位元運算，做的事情是將 NOT x + 1，也就是將 $$x$$ 的位元反轉後加上 $$1$$ 

**範例**：

 1. $$5 = (101)_{binary}$$ ，那麼 $$-5 = (010)_{binary} + 1 = (011)_{binary}$$ 

2. $$10 = (1010)_{binary}$$ ，那麼 $$-10 = (0101)_{binary} + 1 = (0110)_{binary}$$ 

而 $$\&$$ 就是位元運算 AND 的操作，由上面的舉例應該能很清楚的觀察到這是正確的

**範例：**

1. $$5\&(-5) = (101)_{binary} \& (011)_{binary} = (001)_{binary} = 1$$ 

2. $$10\&(-10)=(1010)_{binary} \& (0110)_{binary} = (0010)_{binary} = 2 $$ 

對於lowbit的操作，可以自己練習看看找數字的lowbit

### BIT的運作原理

![BIT &#x7684;&#x9577;&#x76F8;](../.gitbook/assets/fenwick_tree_binary_index_tree.jpg)

這棵樹長的就像上圖，我們用一個陣列來表示所有節點，而可以每個節點儲存的是一段區間的總和，至於是哪一段區間呢？（以下用 $$BIT[i]$$ 表示為樹上的節點， $$Sum(l,r)$$ 表示為 $$[l,r]$$ 的和）

{% hint style="info" %}
**BIT** 節點儲存的資料為： $$BIT[i] = Sum(i-lowbit(i)+1,i)$$ 
{% endhint %}

### 前綴詢問 

而在詢問時，若要詢問到 $$i$$ 的前綴和，該怎麼做呢？

```cpp
int bit[MAXN]; //存BIT節點的陣列
int query(int i){
    int res = 0;
    while(i){ //當i不等於零時，去找答案
        res += bit[i]; //更新答案
        i -= i&(-i); //減掉lowbit，繼續找答案
    }
    return res;
}
```

這樣就完成前綴詢問了！

時間複雜度呢？很明顯最多執行 $$log_2(i)$$ 次！因此詢問的複雜度是 $$O(\log n)$$ 

那區間詢問呢？拿前綴和相減不就好了？ $$query(r)-query(l-1) = Sum(l,r)$$ 

### 單點修改

那如果我們要進行單點修改呢？只要更新會用到這個點的區間不就好了！

仔細想想 BIT 的儲存形式，你會發現有存到陣列第 $$i$$ 個位置的只有以下（這裡用陣列大小為 $$8$$ 做範例：

$$
BIT[1] = Sum(1,1), BIT[2] = Sum(1,2) \\
BIT[3] = Sum(3,3), BIT[4] = Sum(1,4) \\
BIT[5] = Sum(5,5), BIT[6] = Sum(5,6) \\
BIT[7] = Sum(7,7), BIT[8] = Sum(1,8)
$$

當我們要更新 $$arr[2]$$ 時，會更新到誰呢？

會發現$$BIT[2], BIT[4], BIT[8]$$ 都有儲存到 $$arr[2]$$ ，但他們的關係呢？

$$
2 + lowbit(2) = 4, 4+lowbit(4) = 8
$$

因此，要做單點修改時，我們持續去更新那些值就好了！

```cpp
void update(int i, int val){
    while(i < MAXN){ //當i小於陣列大小時，去更新答案
        bit[i] += val; //更新答案
        i += i&(-i); //加上lowbit，繼續更新答案
    }
}
```

### 應用

所以說，詢問與操作前綴有什麼用呢？這裡舉幾個例子

**逆序對**

逆序對的定義是在一個陣列 $$a$$ 中，有幾對 $$a[i] > a[j] \ (i < j)$$

例如： $$[3,1,2]$$ 的逆序對有兩個， $$(3,1)$$ 與 $$(1,2)$$ 

因此，我們每次加入數字時，就可以去找有幾個數字比當前的數字大

如下：

```cpp
int ans = 0;
for(int i = 0; i < n; i++){
    //在答案增加在這個元素前有幾個更大的數字
    ans += query(N)-query(arr[i]); 
    //更新樹上的資料
    update(arr[i]);
}
```

或可以將陣列倒過來做，就不用每次都做兩次詢問了

```cpp
int ans = 0;
for(int i = n-1; i >= 0; i--){
    //在答案增加在這個元素後面有幾個更小的數字
    ans += query(arr[i]-1); 
    //更新樹上的資料
    update(arr[i]);
}
```

**最長遞增子序列（LIS, Longest Increasing Subsequence）**

這是 dp 的經典題之一，一般能用二分搜的方式做到 $$O(n \log n)$$ 

但這裡也可以用 BIT 去做到同樣的事情，但要換成維護前綴最大值

```cpp
int query(int i){
    int res = 0;
    while(i){ //當i不等於零時，去找答案
        res = max(res, bit[i]); //更新答案
        i -= i&(-i); //減掉lowbit，繼續找答案
    }
    return res;
}

void update(int i, int val){
    while(i < MAXN){
        bit[i] = max(bit[i], val);
        i += i&(-i);
    }
}
```

而作法如下：

```cpp
int dp[n];
for(int i = 0; i < n; i++){
    //找前面比自己小的數字最長的子序列長度+1
    dp[i] = max(query(arr[i])+1, ans);
    update(i, dp[i]); //更新答案
}
```

>

