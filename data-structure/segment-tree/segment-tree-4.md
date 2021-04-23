---
description: 就算有很多種操作，線段樹一樣能搞定！
---

# 多種操作的懶惰標記

這裡需要用到區間修改與懶惰標記的概念，若尚未學過，請點選以下連結

{% page-ref page="segment-tree-3.md" %}

### 區間加值、區間賦值

之前已經學過只有單個操作的線段樹了，那麼如果今天題目同時要求兩種操作呢？

聰明的你一定想到了！就是開兩種「懶惰標記」，並且在下推標記時，同時去維護這兩種操作。

不過這個問題難的地方在，究竟該如何實作呢？

### 操作的順序？

在實作的時候，正常來說會遇到的最大問題在於「**操作順序**」亂了

當我們「**先加值、後賦值**」時，在下推時，我們必須要先執行「**加值**」的動作，再去執行「**賦值**」的動作，亦即加值的動作會被賦值蓋過，因此，當我們要進行「**賦值**」的動作時，我們可以直接將「**加值**」的懶標改為 $$0$$ 。而在相反的情況「**先賦值、後加值**」時，整個區間必須是先修改完答案之後才做加值的動作。

大致寫法如下

{% tabs %}
{% tab title="C++" %}
```cpp
void push(int idx, int l, int r){
    int m = (l+r)/ 2;
    if(tag_assign[idx]!=-1){
        tr[idx<<1] = tag_assign[idx] * (m-l+1);
        tr[idx<<1|1] = tag_assign[idx] * (r-m);
        tag_assign[idx<<1] = tag_assign[idx];
        tag_assign[idx<<1|1] = tag_assign[idx];
        tag_add[idx<<1] = tag_add[idx<<1|1] = 0;
        tag_assign[idx] = -1;
    }

    if(tag_add[idx]){
        tr[idx<<1] += tag_add[idx] * (m-l+1);
        tr[idx<<1|1] += tag_add[idx] * (r-m);
        tag_add[idx<<1] += tag_add[idx];
        tag_add[idx<<1|1] += tag_add[idx];
        tag_add[idx] = 0;
    }
}

void add(int ql, int qr, int val, int idx, int l, int r){
    if(l!=r) push(idx, l, r);
    if(ql <= l && r <= qr){
        tr[idx] += val * (r-l+1);
        tag_add[idx] += val;
        return;
    }
    int m = (l+r)/2;
    if(qr > m) add(ql, qr, val, idx*2+1, m+1, r);
    if(ql <= m) add(ql, qr, val, idx*2, l, m);
    tr[idx] = combine(tr[idx<<1],tr[idx<<1|1]);
}

void assign(int ql, int qr, int val, int idx, int l, int r){
    if(l!=r) push(idx, l, r);
    if(ql <= l && r <= qr){
        tr[idx] = val * (r-l+1);
        tag_assign[idx] = val;
        tag_add[idx] = 0; //如果修改了值，那就不必再下推上一個加值的操作了
        return;
    }
    int m = (l+r)/2;
    if(qr > m) assign(ql, qr, val, idx*2+1, m+1, r);
    if(ql <= m) assign(ql, qr, val, idx*2, l, m);
    tr[idx] = combine(tr[idx<<1],tr[idx<<1|1]);
}
```
{% endtab %}
{% endtabs %}

將操作以「**優先下推賦值的標記之後再下推加值的標記**」就不會有問題了

{% tabs %}
{% tab title="Codeforces EDU Segment Tree Part 2 Step 4" %}
[點我前往題目](https://codeforces.ml/edu/course/2/lesson/5/4/practice/contest/280801/problem/A)

> 區間加值，區間賦值，區間詢問總和的線段樹
{% endtab %}

{% tab title="參考程式碼" %}
```cpp
#include <bits/stdc++.h>

#define int long long
#define fastio ios_base::sync_with_stdio(0); cin.tie(0); cout.tie(0);

using namespace std;

const int MAXN = 1e5+5;
int tr[MAXN*4], arr[MAXN], tag_add[MAXN*4], tag_assign[MAXN*4]; 

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

void push(int idx, int l, int r){
    int m = (l+r)/ 2;
    if(tag_assign[idx]!=-1){
        tr[idx<<1] = tag_assign[idx] * (m-l+1);
        tr[idx<<1|1] = tag_assign[idx] * (r-m);
        tag_assign[idx<<1] = tag_assign[idx];
        tag_assign[idx<<1|1] = tag_assign[idx];
        tag_add[idx<<1] = tag_add[idx<<1|1] = 0;
        tag_assign[idx] = -1;
    }

    if(tag_add[idx]){
        tr[idx<<1] += tag_add[idx] * (m-l+1);
        tr[idx<<1|1] += tag_add[idx] * (r-m);
        tag_add[idx<<1] += tag_add[idx];
        tag_add[idx<<1|1] += tag_add[idx];
        tag_add[idx] = 0;
    }
}

void add(int ql, int qr, int val, int idx, int l, int r){
    if(l!=r) push(idx, l, r);
    if(ql <= l && r <= qr){
        tr[idx] += val * (r-l+1);
        tag_add[idx] += val;
        return;
    }
    int m = (l+r)/2;
    if(qr > m) add(ql, qr, val, idx*2+1, m+1, r);
    if(ql <= m) add(ql, qr, val, idx*2, l, m);
    tr[idx] = combine(tr[idx<<1],tr[idx<<1|1]);
}

void assign(int ql, int qr, int val, int idx, int l, int r){
    if(l!=r) push(idx, l, r);
    if(ql <= l && r <= qr){
        tr[idx] = val * (r-l+1);
        tag_assign[idx] = val;
        tag_add[idx] = 0; //如果修改了值，那就不必再下推上一個加值的操作了
        return;
    }
    int m = (l+r)/2;
    if(qr > m) assign(ql, qr, val, idx*2+1, m+1, r);
    if(ql <= m) assign(ql, qr, val, idx*2, l, m);
    tr[idx] = combine(tr[idx<<1],tr[idx<<1|1]);
}

int query(int ql, int qr, int idx, int l, int r){
    if(l!=r) push(idx, l, r);
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
    fill(tag_assign,tag_assign+MAXN*4,-1);
    int n, m;
    cin >> n >> m;
    for(int i = 0;i < m;i++){
        int op;
        cin >> op;
        if(op==1){
            int l, r, v;
            cin >> l >> r >> v;
            assign(l, r-1, v, 1, 0, n-1);
        }else if(op==2){
            int l, r, v;
            cin >> l >> r >> v;
            add(l, r-1, v, 1, 0, n-1);
        }else{
            int l, r;
            cin >> l >> r;
            cout << query(l, r-1, 1, 0, n-1) << "\n";
        }
    }
}
```
{% endtab %}
{% endtabs %}

### 區間加值、區間最大值

如果今天是加值與區間最大值呢？操作順序該如何維護？

當我們「**先加值、後區間最大值**」時，會發生什麼事呢？

詳細實作會在以下章節描述

{% page-ref page="segment-tree-beats.md" %}



