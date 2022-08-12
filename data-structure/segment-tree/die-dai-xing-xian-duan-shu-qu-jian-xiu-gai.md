---
description: 遞迴型的線段樹能做到區間修改，迭代型的當然也可以！
---

# 迭代型線段樹 區間修改

![](../../.gitbook/assets/zkw\_query.png)

### 區間修改(區間加值、詢問區間最小值)

區間修改的操作與區間詢問十分相似，會發現我們實際上需要改的節點與詢問時是一模一樣的

學過遞迴型的線段樹後，應該都知道，我們會在那些節點上打上懶標之後

等到我們真的需要詢問時，才會去下推標記

因此，如同區間詢問，我們可以寫出類似的程式碼

```cpp
for(l += n-1, r += n-1; l < r; l >>= 1, r >>= 1){
    //Change函式是幫助我們拿來更新節點答案的函式
    if(l&1) change(l++, val);
    if(r&1) change(--r, val);
}
```

不過，我們應該如何處理懶惰標記下推的問題呢？

我們將圖畫出來再進行觀察

![從 l 和 r各自畫一條鉛直線通過的節點](../../.gitbook/assets/zkw\_range\_update.png)

會發現，當我們在遞迴向下時，這些節點的標記都會被下推。

那麼，我們模仿遞迴型線段樹所做的，從最上面開始依序去下推這些節點的懶標

因此，我們枚舉的方式就要從最上面開始

為了方便起見，這裡我們以區間加值、詢問區間最小值的線段樹為例

```cpp
void push(int i){
    for(int h = __lg(i); h >= 1; h--){
        int idx = i >> h;
        if(!tag[idx]) continue;
        change(idx<<1, tag[idx]);
        change(idx<<1|1, tag[idx]);
        tag[idx] = 0;
    }
}
```

在遞迴型的線段樹中，我們會將修改完的祖先的值，合併成左右孩子的答案

而這件事情在遞迴時，是由下而上的，因此我們也要模擬這樣子的方式寫出一個 pull

```cpp
void pull(int i){
    for(i>>=1 ; i >= 1; i >>= 1){
        //要記得加上標記，因為標記還沒下推
        tr[i] = combine(tr[i<<1],tr[i<<1|1])+tag[i];
    }
}
```

與遞迴型的線段樹相同，在修改前與詢問前要先下推標記，並且在修改後，要讓每個節點都正確的變為左右孩子合併後的答案。因此整份區間加值、詢問區間最小值的線段樹就會變的如下

```cpp
const int N = 1e5+5;
int tr[N<<1], tag[N], arr[N];
int n;

int combine(int a, int b){
    return min(a,b);
}

void init(){
    for(int i = 1;i <= n;i++){
        tr[i+n-1] = arr[i];
    }
    for(int i = n-1;i >= 1;i--){
        tr[i] = combine(tr[i<<1], tr[i<<1|1]);
    }
}

void change(int idx, int val){
    tr[idx] += val;
    if(idx < n) tag[idx] += val;
}

void push(int i){
    for(int h = __lg(i); h >= 1; h--){
        int idx = i >> h;
        if(!tag[idx]) continue;
        change(idx<<1, tag[idx]);
        change(idx<<1|1, tag[idx]);
        tag[idx] = 0;
    }
}

void pull(int i){
    for(i>>=1 ; i >= 1; i >>= 1){
        tr[i] = combine(tr[i<<1],tr[i<<1|1])+tag[i];
    }
}

void modify(int l, int r, int val){
    int tl = l, tr = r; //紀錄原本的 l, r
    push(l+n-1), push(r-1+n-1);
    for(l += n-1, r += n-1; l < r; l >>= 1, r >>= 1){
        if(l&1) change(l++, val);
        if(r&1) change(--r, val);
    }
    pull(tl+n-1), pull(tr-1+n-1);
}

int query(int l, int r){
    push(l+n-1), push(r-1+n-1);
    int res = 1e18;
    for(l+=n-1, r+=n-1;l < r;l>>=1,r>>=1){
        if(l&1) res = combine(res, tr[l++]);
        if(r&1) res = combine(res, tr[--r]);
    }
    return res;
}
```

### 需要用到區間長度的區間修改？

有很多的區間修改，例如：區間加值、區間求和，都會需要在懶標下推時，用到區間的長度

遇到這種狀況時，我們要利用 zkw 線段樹的節點長度皆為 $$2^k$$ 的性質來幫助我們

其實只要將 change, pull和 modify增加上高度的維護即可

這裡以區間加值、區間求和為例

```cpp
void change(int idx, int val, int h){
    tr[idx] += val * (1<<h);
    if(idx < n) tag[idx] += val;
}

void pull(int i){
    int h;
    for(h = 1, i>>=1 ; i >= 1; i >>= 1){
        tr[i] = combine(tr[i<<1],tr[i<<1|1]) + tag[i] * (1<<h);
    }
}

void modify(int l, int r, int val){
    int tl = l, tr = r, h = 0; 
    push(l+n-1), push(r-1+n-1);
    for(l += n-1, r += n-1; l < r; l >>= 1, r >>= 1, h++){
        if(l&1) change(l++, val, h);
        if(r&1) change(--r, val, h);
    }
    pull(tl+n-1), pull(tr-1+n-1);
}
```

### ZKW的主要概念（？

事實上看到這裡，你應該會發現，雖然說是迭代型的線段樹

不過有許多的操作也只是模仿遞迴型的寫法，一步一步將他們用迴圈的方式模擬出來而已

遞迴型的線段樹也有著許多無法被 zkw 取代的應用，因此，還是建議要將遞迴型的線段樹學好
