---
title: "Baltic OI 18   Valley"
date: 2022-09-24T14:22:53+08:00
tags: [Baltic OI]
---

<body class="line-numbers"> <!-- enabled for the whole page -->

</body>

## 題敘
有一棵大小為\\(N\\)的帶權樹，有\\(S\\)個點是商店，且點\\(E\\)為出口。給你\\(Q\\)個詢問，每個詢問有兩個數字\\(I\\)和\\(R\\)，問你刪除第\\(I\\)個邊後可否由\\(R\\)走到出口\\(E\\)，可以的話則輸出"escaped"，不行的話則輸出最近的商店離\\(R\\)的距離。

## 想法
對於每一個詢問，可以分成「能不能到出口」和「最近商店」兩個部分。\
\
第一個部分我們先把邊\\(I\\)的比較低的節點叫做\\(w\\)。如果我們把\\(E\\)定為整棵樹的根節點，問題就變為\\(w\\)是不是\\(R\\)的祖先，因為如果是的話\\(R\\)到\\(E\\)的過程中一定會要經過邊\\(I\\)，這個問題可以\\(O(n)\\)預處理完後
\\(O(1)\\)判斷。\
\
第二個部分我們可以先知道\\(R\\)一定在\\(W\\)的子樹內。假設我們已經知道離\\(R\\)最近的商店為\\(u\\)，並且定義\\(dis[v]\\)為點\\(E\\)到點\\(v\\)的距離，則有答案為$$ dis[R]+dis[u]-2dis[lca(R,u)] $$現在我們把\\(lca(R,u)\\)叫做\\(k\\)，則上式又可表示為$$ dis[R]-2dis[k]+min_v(dis[v]) $$
現在我們定義一個陣列$$ magic[k]=-2dis[k]+min_v (dis[v]) $$則對於所有在\\(R\\)到\\(w\\)的路徑的\\(k\\)，這個詢問的答案即為$$ dis[R]+min_k(magic[k]) $$
我們可以用一次dfs跑出\\(magic\\)，接著再用倍增法處理路徑上的最小值。

## Code:
 ```cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
#define f first
#define s second
#define all(x) x.begin(),x.end()
#define _ ios::sync_with_stdio(0); cin.tie(0); cout.tie(0);

void setIO(string s) {
    freopen((s + ".in").c_str(), "r", stdin);
    freopen((s + ".out").c_str(), "w", stdout);
}

const int mxn=1e5+5;
vector<pair<int,int>> adj[mxn];
pair<int,int> edgeid[mxn];
bool isshop[mxn]={false};
ll depth[mxn]={0},length[mxn]={0};
ll magic[mxn]={(ll)1e18};
ll jumping[20][mxn],jumpingans[20][mxn];
int st[mxn],en[mxn];
int parent[mxn];
int timer=0;

ll query(int top,int node){
    int x=node;
    int pathlength=depth[node]-depth[top];
    ll ans=magic[node];
    for(int i=0;i<20;i++){
        if(pathlength & (1<<i)){
            ans=min(ans,jumpingans[i][node]);
            node=jumping[i][node];
        }
    }
    return ans+length[x];
}

void dfs(int v,int p){
    st[v]=timer;
    timer++;
    parent[v]=p;
    for(auto u:adj[v]){
        if(u.f==p) continue;
        depth[u.f]=depth[v]+1;
        //cout<<v<<' '<<u.f<<' '<<u.s<<'\n';
        length[u.f]=length[v]+u.s;
        dfs(u.f,v);
    }
    en[v]=timer;
    timer++;
}

void dfs2(int v,int p){
    if(isshop[v]) magic[v]=length[v];
    else magic[v]=(ll)1e18;
    for(auto u:adj[v]){
        if(u.f==p) continue;
        dfs2(u.f,v);
        magic[v]=min(magic[v],magic[u.f]);
    }
}

int main() {_
    //setIO("wayne");
    int n,s,q,e;
    cin>>n>>s>>q>>e;
    for(int i=1;i<=n-1;i++){
        int a,b,w;
        cin>>a>>b>>w;
        adj[a].push_back({b,w});
        adj[b].push_back({a,w});
        edgeid[i]={a,b};
    }
    for(int i=0;i<s;i++){
        int x;
        cin>>x;
        isshop[x]=true;
    }
    dfs(e,0);
    dfs2(e,0);
    for(int i=1;i<=n;i++){
        magic[i]-=2*length[i];
    }
    for(int i=1;i<=n;i++){
        jumping[0][i]=parent[i];
        jumpingans[0][i]=magic[parent[i]];
    }
    for(int i=1;i<20;i++){
        for(int j=1;j<=n;j++){
            jumping[i][j]=jumping[i-1][jumping[i-1][j]];
            jumpingans[i][j]=min(jumpingans[i-1][j],jumpingans[i-1][jumping[i-1][j]]);
        }
    }
    for(int i=0;i<q;i++){
        int edge,node;
        cin>>edge>>node;
        int top=(depth[edgeid[edge].f]>depth[edgeid[edge].s]?edgeid[edge].f:edgeid[edge].s);
        if(st[top]<=st[node] and en[node]<=en[top]){
            if(query(top,node)<=(ll)1e14) cout<<query(top,node)<<'\n';
            else cout<<"oo"<<'\n';
        }
        else{
            cout<<"escaped"<<'\n';
        }
    }
    return 0;
}
```
