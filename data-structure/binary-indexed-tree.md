---
description: 能操作前綴又能快速詢問，程式碼不到十行，好樹不刻嗎？
---

# BIT (Binary Indexed Tree)

### 什麼是 BIT 呢？

BIT (Binary Index Tree) 是一個在1994年，由 **Peter M. Fenwick **所提出的一種資料結構，在國外也常直接叫這個資料結構 「**Fenwick Tree**」，而對岸會叫他「樹狀樹組」，他能夠很有效率的解決需要前綴操作的問題與詢問，並將其降至 $$O(\log n)$$&#x20;

### 數字的lowbit?

在介紹 BIT 的原理與實作前，需要先知道一個數字的「**lowbit**」是什麼，因為他是這個資料結構的核心，而他的定義為「**數字轉成二進位後，最後一個 1 的數值**」。就只有這句話可能有點難懂，讓我們實際來看一下

**範例：**

1\. $$5 = (101)_{binary}$$ ，那麼 $$lowbit(5) = (1)_{binary} = 1$$

2\. $$10 = (1010)_{binary}$$ ，那麼 $$lowbit(10) = (10)_{binary} = 2$$&#x20;

### 程式上要怎麼去找到數字的lowbit呢？

找lowbit的的結論很簡單，就是短短一行 $$x\&(-x)$$ ，至於這行是什麼意思呢

$$-x$$ 的位元運算，做的事情是將 NOT x + 1，也就是將 $$x$$ 的位元反轉後加上 $$1$$&#x20;

**範例**：

&#x20;1\. $$5 = (101)_{binary}$$ ，那麼 $$-5 = (010)_{binary} + 1 = (011)_{binary}$$&#x20;

2\. $$10 = (1010)_{binary}$$ ，那麼 $$-10 = (0101)_{binary} + 1 = (0110)_{binary}$$&#x20;

而 $$\&$$ 就是位元運算 AND 的操作，由上面的舉例應該能很清楚的觀察到這是正確的

**範例：**

1\. $$5\&(-5) = (101)_{binary} \& (011)_{binary} = (001)_{binary} = 1$$&#x20;

2\. $$10\&(-10)=(1010)_{binary} \& (0110)_{binary} = (0010)_{binary} = 2$$&#x20;

對於lowbit的操作，可以自己練習看看找數字的lowbit

### BIT的運作原理

![BIT 的長相](../.gitbook/assets/fenwick\_tree\_binary\_index\_tree.jpg)

這棵樹長的就像上圖，我們用一個陣列來表示所有節點，而可以每個節點儲存的是一段區間的總和，至於是哪一段區間呢？（以下用 $$BIT[i]$$ 表示為樹上的節點， $$Sum(l,r)$$ 表示為 $$[l,r]$$ 的和）

{% hint style="info" %}
**BIT **節點儲存的資料為： $$BIT[i] = Sum(i-lowbit(i)+1,i)$$&#x20;
{% endhint %}

### 前綴詢問&#x20;

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

時間複雜度呢？很明顯最多執行 $$log_2(i)$$ 次！因此詢問的複雜度是 $$O(\log n)$$&#x20;

那區間詢問呢？拿前綴和相減不就好了？ $$query(r)-query(l-1) = Sum(l,r)$$&#x20;

### 單點修改

那如果我們要進行單點修改呢？只要更新會用到這個點的區間不就好了！

仔細想想 BIT 的儲存形式，這裡用陣列大小為 $$8$$ 做範例：

$$
BIT[1] = Sum(1,1), \ BIT[2] = Sum(1,2) \\
BIT[3] = Sum(3,3), \ BIT[4] = Sum(1,4) \\
BIT[5] = Sum(5,5), \ BIT[6] = Sum(5,6) \\
BIT[7] = Sum(7,7), \ BIT[8] = Sum(1,8)
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

例如： $$[3,1,2]$$ 的逆序對有兩個， $$(3,1)$$ 與 $$(3,2)$$&#x20;

因此，我們每次加入數字時，就可以去找有幾個數字比當前的數字大

如下：

```cpp
int ans = 0;
for(int i = 0; i < n; i++){
    //在答案增加在這個元素前有幾個更大的數字
    ans += query(MAXN-1)-query(arr[i]); 
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

這是 dp 的經典題之一，一般能用二分搜的方式做到 $$O(n \log n)$$&#x20;

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
    dp[i] = max(query(arr[i])+1, dp[i]);
    update(arr[i], dp[i]); //更新答案
}
```

**BIT 上二分搜**

要在 BIT 上二分搜，有兩種方法，而這裡我們用一個例題來講述

> 在陣列中找到第 $$k$$ 大的元素為何？

而寫法分為兩種，一種是**正常二分搜**，第二種是**倍增法二分搜**

（這裡的樹存的是出現頻率，如 $$[1,2,2,3]$$ ，則頻率陣列為 $$[1,2,1]$$  ）

```cpp
//正常二分搜
int find(int k){
    int l = 0, r = n;
    while(l < r){
        int m = (l+r)/2;
        if(query(m) >= k) r = m;
        else l = m+1;  
    }
}
//l即為第k大的元素
```

這個就只是正常的二分搜搭配 BIT 的詢問，時間複雜度 $$O(\log^2 n)$$&#x20;

```cpp
//倍增法二分搜
int find(int k){
    int sum = 0, pos = 0;
    for(int i = LOGN; i >= 0; i--)
        if(pos + (1<<i) < n && sum + bit[pos + (1<<i)] < k){
            sum += bit[pos + (1<<i)];
            pos += (1<<i);
        }
    }
    return pos;
}
```

至於為什麼能這樣做，就當做是給讀者的一個小練習，時間複雜度 $$O(\log n)$$&#x20;

### 練習題

#### 單點修改、區間詢問

{% tabs %}
{% tab title=" CF Edu Segment Tree Part 1 Step 1" %}
[點我前往題目](https://codeforces.com/edu/course/2/lesson/4/1/practice/contest/273169/problem/A)

第一、二題：區間求和線段樹

> 給一個 $$n$$ 項的陣列，一共有 $$m$$ 次操作及詢問
>
> 一、將陣列第 $$i$$ 個位置的元素改成 $$v$$&#x20;
>
> 二、詢問陣列在區間 $$[l,r)$$ 之間的總和
{% endtab %}

{% tab title="參考程式碼" %}
```cpp
#include <bits/stdc++.h>

#define ll long long
#define fastio ios_base::sync_with_stdio(0); cin.tie(0); cout.tie(0);

using namespace std;

const int MAXN = 1e5+5;
ll bit[MAXN];

void update(int i, ll val){
	while(i < MAXN){
		bit[i] += val;
		i += i&(-i);
	}
}

ll query(int i){
	ll res = 0;
	while(i){
		res += bit[i];
		i -= i&(-i);
	}
	return res;
}

signed main(){
	fastio
	int n, m;
	cin >> n >> m;
	ll arr[n+1];
	for(int i = 1; i <= n; i++) cin >> arr[i];
	for(int i = 1; i <= n; i++) update(i,arr[i]);
	for(int i = 0; i < m; i++){
		int op;
		cin >> op;
		if(op == 1){
			ll i,v;
			cin >> i >> v;
			i++;
			update(i,-arr[i]);
			arr[i] = v;
			update(i,arr[i]);
		}else if(op == 2){
			int l, r;
			cin >> l >> r;
			l++;
			cout << query(r) - query(l-1) << "\n";
		}
	}
}
```
{% endtab %}
{% endtabs %}

**逆序對**

{% tabs %}
{% tab title=" CF Edu Segment Tree Part 1 Step 3" %}
[點我前往題目](https://codeforces.com/edu/course/2/lesson/4/3/practice/contest/274545/problem/A)

給一個陣列，問在每個點前面有幾個比他大的數字
{% endtab %}

{% tab title="參考程式碼" %}
```cpp
#include <bits/stdc++.h>

#define ll long long
#define fastio ios_base::sync_with_stdio(0); cin.tie(0); cout.tie(0);

using namespace std;

const int N = 1e5+5;
int bit[N];

void update(int pos, int val){
	while(pos<N){
		bit[pos] += val;
		pos += pos&-pos;
	}
}

int query(int pos){
	int res = 0;
	while(pos){
		res += bit[pos];
		pos -= pos&-pos;
	}
	return res;
}

signed main(){
	fastio
	int n;
	cin >> n;
	while(n--){
		int x;
		cin >> x;
		cout << query(N-1) - query(x) << " ";
		update(x,1);
	}
}
```
{% endtab %}
{% endtabs %}

**LIS(DP)**

{% tabs %}
{% tab title="Atcoder Educational DP contest (Q) Flowers " %}
[點我前往題目](https://atcoder.jp/contests/dp/tasks/dp\_q)

> 有$$n$$朵花，每一朵花都有各自的高度 $$h_i$$ 與漂亮程度 $$a_i$$
>
> 今天你希望由左到右選取一些高度遞增的花
>
> 問最大的漂亮程度總和為？&#x20;
{% endtab %}

{% tab title="參考程式碼" %}
```cpp
#include <bits/stdc++.h>

#define ll long long
#define fastio ios_base::sync_with_stdio(0); cin.tie(0); cout.tie(0);

using namespace std;


const int N = 2e5+5;

ll h[N], a[N], bit[N];

void update(int i, int v){
	while(i < N){
		bit[i] = max(bit[i],v);
		i += i&-i;
	}
}

int query(int i){
	int ans = 0;
	while(i){
		ans = max(ans,bit[i]);
		i -= i&-i;
	}
	return ans;
}

signed main(){
	fastio
	int n;
	cin >> n;
	for(int i = 0;i < n;i++) cin >> h[i];
	for(int i = 0;i < n;i++) cin >> a[i];
	for(int i = 0;i < n;i++){
		update(h[i],query(h[i])+a[i]);
	}
	cout << query(n) << "\n";
}
```
{% endtab %}
{% endtabs %}

