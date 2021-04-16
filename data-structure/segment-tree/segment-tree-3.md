---
description: 遇到修改連續區間時，紀錄下來之後再做不就好了嗎？
---

# 區間修改與懶惰標記

在看這章節前，若尚未學過單點修改的線段樹，請點選以下連結

{% page-ref page="segment-tree-1.md" %}

### 區間加值 單點詢問

我們已經學過單點修改的線段樹可以在 $$O(\log n)$$的時間內完成，那麼如果我們今天只是要在修改後，詢問某一個位置的值，那麼我們其實可以使用一種技巧，也就是「差分」！

設一個陣列 $$A$$ 的元素為以下

$$
A[0], \ A[1], \ A[2], \ ..., \ A[n]
$$

在進行區間修改 $$[l,r]$$ 時，如果我們每一個點都去修改他的值，我們要花的時間為 $$O((r-l+1)\log n) \approx O(n \log n)$$ ，實在不是一個非常好的作法

如果我們用另外一個陣列 $$B$$ 存陣列 $$A$$ 兩兩元素之間的差距呢？ 也就是如下

$$
A[1] - A[0], \ A[2] - A[1], \ A[3] - A[2], ..., \ A[n] - A[n-1]
$$

那其實只需要讓 $$B[l]$$ 去加上區間所要加的值，以及 $$B[r+1]$$ 去減掉區間加的值

當我們在詢問 $$A[i]$$ 時，只要詢問陣列$$B$$ 的前 $$i$$ 項之和，即可得到答案

時間複雜度：區間加值 $$O(\log n)$$ ，單點求值 $$O(\log n)$$ 

### 區間修改 區間詢問

我們可以看一下線段樹的運作方式，如下圖

![&#x5340;&#x9593;&#x7684;&#x8A62;&#x554F; or &#x5340;&#x9593;&#x7684;&#x4FEE;&#x6539;](../../.gitbook/assets/segment-tree-.jpg)

在我們要修改區間 $$[2,5]$$ 時，我們其實只要像區間詢問時，只修改上面塗成紅色的三個區間就好了

但是！這樣會出現一個問題，當我們修改完 $$[3,4]$$ 之後，如果詢問 $$[3]$$ 的值呢？會發現如果我們僅僅是修改 $$[3,4]$$ ，那麼在詢問 $$[3]$$ 的時候，就會是還沒修改過的值了

你可能會想，那我們就認命的一個一個元素慢慢修改吧！

不過！這一點，線段樹可以輕鬆解決！ 我們可以在每一個節點去紀錄已經修改過的值，當我們在詢問他的子節點時，再下推答案就好了！

這時候我們就會用到我們稱為「懶惰標記（Lazy Propagation）」的東西了

### 懶惰標記

要使用這種方式進行區間修改，則對於線段樹中的值必須具有「可預測性」

什麼是「可預測性」呢？

例如：對整個區間進行 $$+$$ 的操作，如果每個點都增加 $$x$$ ，那麼整個區間 $$[l,r]$$ 的總和則會增加 $$(r-l+1)x$$ 。

代表說，我們不必去修改他的子節點，即可得知該區間的值

之後，我們可以在該節點上，打上一個標記，紀錄目前這個區間已經增加了 $$x$$ ，那麼當我們要詢問樹上該節點的子節點時，我們就可以順便將 $$x$$ 的值下推

詳細作法如下：

```cpp
const int MAXN = 1e5+5;
int tr[MAXN*4], arr[MAXN], tag[MAXN*4]; //線段樹的節點數量一般會開成 4n

void push(int idx){
    //下推節點的標記（這裡以區間加值 區間最大值為例）
    if(tag[idx]){
        tr[idx<<1] += tag[idx];
        tr[idx<<1|1] += tag[idx];
        tag[idx<<1] += tag[idx];
        tag[idx<<1|1] += tag[idx];
        tag[idx] = 0;
    }
}

int query(int ql, int qr, int idx, int l, int r){
    if(l!=r) push(idx); //當節點並非葉節點時，下推標記
    if(ql <= l && r <= qr){
        return tr[idx];
    }
    int m = (l+r)/2;
    if(ql > m){
        return query(ql, qr, idx*2+1, m+1, r);
    }
    if(qr <= m){
        return query(ql, qr, idx*2, l, m);
    }
    return combine(query(ql, qr, idx*2, l, m), query(ql, qr, idx*2+1, m+1, r));
}

//這裡最特別的地方，區間修改，寫法與區間詢問大致相同
void modify(int ql, int qr, int val, int idx, int l, int r){
    if(l!=r) push(idx); //當節點並非葉節點時，下推標記
    if(ql <= l && r <= qr){
        tr[idx] += val;
        return;
    }
    int m = (l+r)/2;
    if(qr > m) modify(ql, qr, val, idx*2+1, m+1, r);
    if(ql <= m) modify(ql, qr, val, idx*2, l, m);
    tr[idx] = combine(tr[idx<<1],tr[idx<<1|1]);
}
```

### 練習題

{% tabs %}
{% tab title="CF Edu Segment tree part 2 step 1" %}
[點我前往題目](https://codeforces.ml/edu/course/2/lesson/5/1/practice)

第一題：區間加值 單點求值

第二題：區間最大值 單點求值

第三題：區間賦值 單點求
{% endtab %}

{% tab title="參考程式碼（一）" %}
```cpp
#include <bits/stdc++.h>

#define ll long long
#define fastio ios_base::sync_with_stdio(0); cin.tie(0); cout.tie(0);

using namespace std;

const int MAXN = 1e5+5;
ll tr[MAXN*4], arr[MAXN]; 

ll combine(ll a, ll b){
    return a+b;
}

void build(int idx, int l, int r){
    if(l==r){
        tr[idx] = arr[l]; 
    }else{
        int m = (l+r)/2;
        build(idx*2,l,m); 
        build(idx*2+1,m+1,r); 
        tr[idx] = combine(tr[idx*2],tr[idx*2+1]); 
    }
}

void update(int pos, int val, int idx, int l, int r){
    if(l==r){
        tr[idx] += val;
        return;
    }
    int m = (l+r)/2;
    if(pos <= m) update(pos, val, idx*2, l, m);
    else update(pos, val, idx*2+1, m+1, r);
    tr[idx] = combine(tr[idx*2],tr[idx*2+1]); 
}

ll query(int ql, int qr, int idx, int l, int r){
    if(ql <= l && r <= qr){
        return tr[idx];
    }
    int m = (l+r)/2;
    if(ql > m){
        return query(ql, qr, idx*2+1, m+1, r);
    }
    if(qr <= m){
        return query(ql, qr, idx*2, l, m);
    }
    return combine(query(ql, qr, idx*2, l, m), query(ql, qr, idx*2+1, m+1, r));
}

signed main(){
    fastio
    int n, m;
    cin >> n >> m;
    for(int i = 0;i < m;i++){
        int op;
        cin >> op;
        if(op==1){
            int l, r, v;
            cin >> l >> r >> v;
            update(l, v, 1, 0, n-1);
            if(r != n) update(r, -v, 1, 0, n-1);
        }else{
            int i;
            cin >> i;
            cout << query(0, i, 1, 0, n-1) << "\n";
        }
        for(int j = 0;j < n;j++){
            cout << query(0, j, 1, 0, n-1) << " ";
        }
        cout << "\n";
    }
}
```
{% endtab %}

{% tab title="參考程式碼（二）" %}
```cpp
#include <bits/stdc++.h>

#define fastio ios_base::sync_with_stdio(0); cin.tie(0); cout.tie(0);

using namespace std;

const int MAXN = 1e5+5;
int tr[MAXN*4], arr[MAXN], tag[MAXN*4]; 

int combine(int a, int b){
    return max(a,b);
}

void build(int idx, int l, int r){
    if(l==r){
        tr[idx] = arr[l]; 
    }else{
        int m = (l+r)/2;
        build(idx*2,l,m); 
        build(idx*2+1,m+1,r); 
        tr[idx] = combine(tr[idx*2],tr[idx*2+1]); 
    }
}

void push(int idx){
    if(tag[idx]){
        tr[idx<<1] = max(tr[idx<<1], tag[idx]);
        tr[idx<<1|1] = max(tr[idx<<1|1], tag[idx]);
        tag[idx<<1] = max(tag[idx<<1], tag[idx]);
        tag[idx<<1|1] = max(tag[idx<<1|1], tag[idx]);
        tag[idx] = 0;
    }
}

void modify(int ql, int qr, int val, int idx, int l, int r){
    if(l!=r) push(idx); //當節點並非葉節點時，下推標記
    if(ql <= l && r <= qr){
        tr[idx] = max(tr[idx],val);
        tag[idx] = max(tag[idx],val);
        return;
    }
    int m = (l+r)/2;
    if(qr > m) modify(ql, qr, val, idx*2+1, m+1, r);
    if(ql <= m) modify(ql, qr, val, idx*2, l, m);
    tr[idx] = combine(tr[idx<<1],tr[idx<<1|1]);
}

int query(int ql, int qr, int idx, int l, int r){
    if(l!=r) push(idx); //當節點並非葉節點時，下推標記
    if(ql <= l && r <= qr){
        return tr[idx];
    }
    int m = (l+r)/2;
    if(ql > m){
        return query(ql, qr, idx*2+1, m+1, r);
    }
    if(qr <= m){
        return query(ql, qr, idx*2, l, m);
    }
    return combine(query(ql, qr, idx*2, l, m), query(ql, qr, idx*2+1, m+1, r));
}

signed main(){
    fastio
    int n, m;
    cin >> n >> m;
    for(int i = 0;i < m;i++){
        int op;
        cin >> op;
        if(op==1){
            int l, r, v;
            cin >> l >> r >> v;
            modify(l, r-1, v, 1, 0, n-1);
        }else{
            int i;
            cin >> i;
            cout << query(i, i, 1, 0, n-1) << "\n";
        }
    }
}#include <bits/stdc++.h>

#define fastio ios_base::sync_with_stdio(0); cin.tie(0); cout.tie(0);

using namespace std;

const int MAXN = 1e5+5;
int tr[MAXN*4], arr[MAXN], tag[MAXN*4]; 

int combine(int a, int b){
    return max(a,b);
}

void build(int idx, int l, int r){
    if(l==r){
        tr[idx] = arr[l]; 
    }else{
        int m = (l+r)/2;
        build(idx*2,l,m); 
        build(idx*2+1,m+1,r); 
        tr[idx] = combine(tr[idx*2],tr[idx*2+1]); 
    }
}

void push(int idx){
    if(tag[idx]){
        tr[idx<<1] = max(tr[idx<<1], tag[idx]);
        tr[idx<<1|1] = max(tr[idx<<1|1], tag[idx]);
        tag[idx<<1] = max(tag[idx<<1], tag[idx]);
        tag[idx<<1|1] = max(tag[idx<<1|1], tag[idx]);
        tag[idx] = 0;
    }
}

void modify(int ql, int qr, int val, int idx, int l, int r){
    if(l!=r) push(idx); //當節點並非葉節點時，下推標記
    if(ql <= l && r <= qr){
        tr[idx] = max(tr[idx],val);
        tag[idx] = max(tag[idx],val);
        return;
    }
    int m = (l+r)/2;
    if(qr > m) modify(ql, qr, val, idx*2+1, m+1, r);
    if(ql <= m) modify(ql, qr, val, idx*2, l, m);
    tr[idx] = combine(tr[idx<<1],tr[idx<<1|1]);
}

int query(int ql, int qr, int idx, int l, int r){
    if(l!=r) push(idx); //當節點並非葉節點時，下推標記
    if(ql <= l && r <= qr){
        return tr[idx];
    }
    int m = (l+r)/2;
    if(ql > m){
        return query(ql, qr, idx*2+1, m+1, r);
    }
    if(qr <= m){
        return query(ql, qr, idx*2, l, m);
    }
    return combine(query(ql, qr, idx*2, l, m), query(ql, qr, idx*2+1, m+1, r));
}

signed main(){
    fastio
    int n, m;
    cin >> n >> m;
    for(int i = 0;i < m;i++){
        int op;
        cin >> op;
        if(op==1){
            int l, r, v;
            cin >> l >> r >> v;
            modify(l, r-1, v, 1, 0, n-1);
        }else{
            int i;
            cin >> i;
            cout << query(i, i, 1, 0, n-1) << "\n";
        }
    }
}
```
{% endtab %}

{% tab title="參考程式碼（三）" %}
```cpp
#include <bits/stdc++.h>

#define fastio ios_base::sync_with_stdio(0); cin.tie(0); cout.tie(0);

using namespace std;

const int MAXN = 1e5+5;
int tr[MAXN*4], arr[MAXN], tag[MAXN*4]; 

int combine(int a, int b){
    return a+b;
}

void build(int idx, int l, int r){
    if(l==r){
        tr[idx] = arr[l]; 
    }else{
        int m = (l+r)/2;
        build(idx*2,l,m); 
        build(idx*2+1,m+1,r); 
        tr[idx] = combine(tr[idx*2],tr[idx*2+1]); 
    }
}

void push(int idx){
    if(tag[idx] != -1){
        tr[idx<<1] = tag[idx];
        tr[idx<<1|1] = tag[idx];
        tag[idx<<1] = tag[idx];
        tag[idx<<1|1] = tag[idx];
        tag[idx] = -1;
    }
}

void modify(int ql, int qr, int val, int idx, int l, int r){
    if(l!=r) push(idx); //當節點並非葉節點時，下推標記
    if(ql <= l && r <= qr){
        tr[idx] = val;
        tag[idx] = val;
        return;
    }
    int m = (l+r)/2;
    if(qr > m) modify(ql, qr, val, idx*2+1, m+1, r);
    if(ql <= m) modify(ql, qr, val, idx*2, l, m);
    tr[idx] = combine(tr[idx<<1],tr[idx<<1|1]);
}

int query(int ql, int qr, int idx, int l, int r){
    if(l!=r) push(idx); //當節點並非葉節點時，下推標記
    if(ql <= l && r <= qr){
        return tr[idx];
    }
    int m = (l+r)/2;
    if(ql > m){
        return query(ql, qr, idx*2+1, m+1, r);
    }
    if(qr <= m){
        return query(ql, qr, idx*2, l, m);
    }
    return combine(query(ql, qr, idx*2, l, m), query(ql, qr, idx*2+1, m+1, r));
}

signed main(){
    fastio
    int n, m;
    cin >> n >> m;
    fill(tag,tag+MAXN*4,-1);
    for(int i = 0;i < m;i++){
        int op;
        cin >> op;
        if(op==1){
            int l, r, v;
            cin >> l >> r >> v;
            modify(l, r-1, v, 1, 0, n-1);
        }else{
            int i;
            cin >> i;
            cout << query(i, i, 1, 0, n-1) << "\n";
        }
    }
}
```
{% endtab %}
{% endtabs %}

