---
description: 單點修改，區間詢問，一切都難不倒這棵樹！
---

# 單點修改 區間詢問

![線段樹畫成圖的樣子(?](<../../.gitbook/assets/Segment Tree (1).jpg>)

### 什麼是線段樹？

線段樹是一種將陣列變成一棵樹來進行區間操作及詢問的一種資料結構，最經典會拿來解決的問題像是 RMQ (Range Minimum Query)。不過各種區間操作都會用到這樣的一種資料結構。

而線段樹主要是將一個陣列，用像上圖那樣的方式分成很多不同的節點，建立成一棵二元樹，而每個節點所對應到的都是一個區間，如圖上最上面的點就是維護區間 $$[1,8]$$ 的資訊，而每個節點都分別對應到 $$2^k$$個位置，而在詢問時，如詢問 $$[2,5]$$ 這個區間時，就會像下圖所示，分別詢問三個區間 $$[2], [3,4], [5]$$，並將答案合併，即為我們所要詢問的答案

![線段樹詢問](<../../.gitbook/assets/Segment Tree (1) (1).jpg>)

### 建立線段樹

要建立一棵線段樹，我們要先了解我們該如何去建出這棵樹，一般來說，我們會使用另外一個陣列來存整棵樹的節點（當然也有指標的寫法，但這裡暫時不提）

而由於我們會希望建出來的這棵樹是一棵「完美二元樹」，因為完美二元樹放在陣列中，可以很簡單的將節點的編號放進陣列中，而他的總共元素為 $$2^n-1$$ ，如下圖

![線段樹的編號](<../../.gitbook/assets/Segment Tree Number (1).jpg>)

但線段樹只能用在節點數量為二的冪次的陣列嗎？答案是否定的，我們同樣可以將一個大小並非二的冪次的陣列建成完美二元樹，但是有些節點所儲存的資料會是空的。那陣列大小需要多大呢？我們可以來實際計算看看。

若一個陣列的大小 $$n$$ 為二的冪次，我們可以很簡單的觀察出儲存線段樹的陣列大小為 $$2n-1$$，那當陣列大小 $$n$$ 並非二的冪次時呢？例如： $$n=2^k+x (0 \le x \le 2^k)$$ ，我們可以將其補到二的冪次，而 $$n < 2^{k+1}$$ ，因此線段樹的大小為 $$2^{k+2}-1$$ ，而這個數字我們可以發現，一定會小於 $$n \times 2^2$$ 也就是 $$4n$$ ，因此我們都會將節點的陣列大小開成 $$4n$$

因為我們所建立的節點數為 $$4n$$ ，建立的時間複雜度為 $$O(n)$$&#x20;

{% tabs %}
{% tab title="C++" %}
```cpp
const int MAXN = 1e5+5;
ll tr[MAXN*4], arr[MAXN]; //線段樹的節點數量一般會開成 4n
 
void build(int idx, int l, int r){
    if(l==r){
        //在建立線段樹時，若 l,r 相同，此節點的值為 arr[l]
        tr[idx] = arr[l]; 
    }else{
        //在建立線段樹時，會從中間分兩邊去建立
        int m = (l+r)/2;
        build(idx*2,l,m); //一個節點的左子樹一般會用 idx*2 表示
        build(idx*2+1,m+1,r); //一個節點的左子樹一般會用 idx*2+1 表示
        tr[idx] = combine(tr[idx*2],tr[idx*2+1]); //將左右子樹的資料合併
    }
}
```
{% endtab %}
{% endtabs %}

### 區間詢問

在建立好一棵線段樹之後，最重要的就是區間的詢問了吧？

讓我們回到這張圖

![](<../../.gitbook/assets/Segment Tree (1) (1).jpg>)

在區間詢問時，若我們詢問的區間，如 $$[2,5]$$ ，將這個節點上的區間全部包住了（$$[3,4]$$），那麼我們就可以直接回傳這個節點的答案，否則如 $$[1,2]$$ ，我們就要繼續往下尋找答案。

由於整個樹有 $$\log_2(n)$$ 層，因此我們每次詢問最多 $$C \cdot \log_2(n)$$（C為一常數），時間複雜度為 $$O(\log n)$$&#x20;

{% tabs %}
{% tab title="C++" %}
```cpp
int query(int ql, int qr, int idx, int l, int r){
    //l,r為這個節點的儲存區間的左右界
    if(ql <= l && r <= qr){
        //若這個區間被詢問完全包住，直接回傳答案
        return tr[idx];
    }
    int m = (l+r)/2;
    if(ql > m){
        //若詢問的區間完全在右邊，我們只須往右找答案
        return query(ql, qr, idx*2+1, m+1, r);
    }
    if(qr <= m){
        //若詢問的區間完全在左邊，我們只須往左找答案
        return query(ql, qr, idx*2, l, m);
    }
    //往左右都尋找答案，並回傳合併後的答案
    return combine(query(ql, qr, idx*2, l, m), query(ql, qr, idx*2+1, m+1, r));
}
```
{% endtab %}
{% endtabs %}

### 單點修改

有時候，題目並不會只是單單的只有詢問這個動作，通常還會搭配著修改這樣的操作，而如果我們只是要修改陣列上的某一個位置，那很簡單，我們只要一路往下去找我們要修改的位置即可。

{% tabs %}
{% tab title="C++" %}
```cpp
void update(int pos, int val, int idx, int l, int r){
    if(l==r){
        tr[idx] = val;
        return;
    }
    int m = (l+r)/2;
    //依據要更新的位置去更新左或右子樹的資訊
    if(pos <= m) update(pos, val, idx*2, l, m);
    else update(pos, val, idx*2+1, m+1, r);
    tr[idx] = combine(tr[idx*2],tr[idx*2+1]); //將左右子樹的資料合併
}
```
{% endtab %}
{% endtabs %}

### 基本的線段樹建立完了！

若以上都學會了，那麼恭喜你！你已經學會了基礎的線段樹

以下有一些線段樹單點修改區間詢問的練習題，可以去練習看看

{% tabs %}
{% tab title="CF Edu Segment Tree Part 1 Step 1" %}
[點我前往題目](https://codeforces.com/edu/course/2/lesson/4/1/practice)

第一、二題：區間求（和/最小值）線段樹

> 給一個 $$n$$ 項的陣列，一共有 $$m$$ 次操作及詢問
>
> 一、將陣列第 $$i$$ 個位置的元素改成 $$v$$&#x20;
>
> 二、詢問陣列在區間 $$[l,r)$$ 之間的總和/最小值

第三題：區間最小值出現次數

> 一個 $$n$$ 項的陣列，一共有 $$m$$ 次操作及詢問
>
> 一、將陣列第 $$i$$ 個位置的元素改成 $$v$$&#x20;
>
> 二、詢問陣列在區間 $$[l,r)$$ 之間的最小值以及最小值出現次數


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
        tr[idx] = val;
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
    for(int i = 0; i < n; i++){
        cin >> arr[i];
    }
    build(1, 0, n-1);
    for(int i = 0;i < m;i++){
        int op;
        cin >> op;
        if(op==1){
            int i, v;
            cin >> i >> v;
            update(i, v, 1, 0, n-1);
        }else{
            int l, r;
            cin >> l >> r;
            cout << query(l, r-1, 1, 0, n-1) << "\n";
        }
    }
}
```
{% endtab %}

{% tab title="參考程式碼（二）" %}
```cpp
#include <bits/stdc++.h>

#define ll long long
#define fastio ios_base::sync_with_stdio(0); cin.tie(0); cout.tie(0);

using namespace std;

const int MAXN = 1e5+5;
ll tr[MAXN*4], arr[MAXN]; 

ll combine(ll a, ll b){
    return min(a,b);
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
        tr[idx] = val;
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
    for(int i = 0; i < n; i++){
        cin >> arr[i];
    }
    build(1, 0, n-1);
    for(int i = 0;i < m;i++){
        int op;
        cin >> op;
        if(op==1){
            int i, v;
            cin >> i >> v;
            update(i, v, 1, 0, n-1);
        }else{
            int l, r;
            cin >> l >> r;
            cout << query(l, r-1, 1, 0, n-1) << "\n";
        }
    }
}
```
{% endtab %}

{% tab title="參考程式碼（三）" %}
```cpp
#include <bits/stdc++.h>

#define ll long long
#define fastio ios_base::sync_with_stdio(0); cin.tie(0); cout.tie(0);

using namespace std;

const int MAXN = 1e5+5;
ll arr[MAXN];
pair<ll,ll> tr[MAXN*4]; //第一個存最小值，第二個存出現次數

pair<ll,ll> combine(pair<ll,ll> a, pair<ll,ll> b){
    //回傳左右子樹中較小的答案
    if(a.first < b.first) return a; 
    else if(a.first > b.first) return b;
    
    //若左右子樹的最小值相同，將次數累加
    return {a.first, a.second + b.second};
}

void build(int idx, int l, int r){
    if(l==r){
        tr[idx] = {arr[l],1}; 
    }else{
        int m = (l+r)/2;
        build(idx*2,l,m); 
        build(idx*2+1,m+1,r); 
        tr[idx] = combine(tr[idx*2],tr[idx*2+1]); 
    }
}

void update(int pos, int val, int idx, int l, int r){
    if(l==r){
        tr[idx] = {val,1};
        return;
    }
    int m = (l+r)/2;
    if(pos <= m) update(pos, val, idx*2, l, m);
    else update(pos, val, idx*2+1, m+1, r);
    tr[idx] = combine(tr[idx*2],tr[idx*2+1]); 
}

pair<ll,ll> query(int ql, int qr, int idx, int l, int r){
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
    for(int i = 0; i < n; i++){
        cin >> arr[i];
    }
    build(1, 0, n-1);
    for(int i = 0;i < m;i++){
        int op;
        cin >> op;
        if(op==1){
            int i, v;
            cin >> i >> v;
            update(i, v, 1, 0, n-1);
        }else{
            int l, r;
            cin >> l >> r;
            pair<ll,ll> ans = query(l, r-1, 1, 0, n-1);
            cout << ans.first << " " << ans.second << "\n";
        }
    }
}
```
{% endtab %}
{% endtabs %}
