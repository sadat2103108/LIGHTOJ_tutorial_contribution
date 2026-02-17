# LightOJ 1101 – A Secret Mission

## Problem Summary

We are given a connected weighted graph with (N) cities and (M) roads.
Each road has a *danger value*.

For each query (s, t), we must find a path from (s) to (t) such that:

> the maximum danger of any road on the path is minimized.

This is known as a **minimax path problem**.

---

## Key Observation

If we build a **Minimum Spanning Tree (MST)** of the graph, then:

> For any two nodes, the path between them in the MST minimizes the maximum edge weight among all possible paths in the original graph.

This is a well-known property of MSTs.
Therefore, once we construct the MST, we only need to answer queries on that tree.

---

## Strategy

1. Build the MST using **Prim’s algorithm**
2. Convert the graph into a tree
3. Preprocess the tree using **Binary Lifting**
4. Store:

   * `anc[u][i]` → (2^i)-th ancestor of node (u)
   * `mx[u][i]` → maximum edge weight from (u) to its (2^i)-th ancestor
5. For each query:

   * compute LCA of (u) and (v)
   * find maximum edge on path (u to LCA) and (v to LCA) using binary lifting
   * answer = max of the two above values.

---

## Complexity

* MST construction: (O(M log N))
* Binary lifting preprocessing: (O(N log N))
* Each query: (O(log N))

This easily fits within constraints.

---

## Implementation

```cpp
#include<bits/stdc++.h>
using namespace std;
#define br cout<<"\n";
#define ll long long
#define loop(n) for(int i=0; i<(n); i++)
#define fr(i,init, n) for(int i=(init); i<(n); i++)
#define revl(i,init) for(int i=(init-1); i>=0; i--)
#define pb push_back
#define all(v) v.begin(),v.end()
#define nl "\n"


const int N =  50005, LOG=18 ;
vector<pair<int,int>> adj2[N], adj[N];
bool vis[N];
int anc[N][LOG], mx[N][LOG], dep[N];


// clear global variables for each test case
void clear(){
    loop(N){
        vis[i] = false;
        adj2[i].clear();
        adj[i].clear();
    }
    memset(anc,0, sizeof(anc));
    memset(mx, 0, sizeof(mx));
    memset(dep, 0, sizeof(dep));
}

// create the MST from adj2[] and build the new one in adj[]
void createMST(){
    
    priority_queue<
        tuple<int,int,int>,
        vector<tuple<int,int,int>>,
        greater<tuple<int,int,int>>
    > pq;
    
    pq.push({-1, 1,0});
    
    while(! pq.empty()){
        int wt, node, parent;
        tie(wt, node, parent) = pq.top();
        pq.pop();
        
        if(vis[node]) continue;
        
        vis[node] = true;
        
        if(parent!=0){
            adj[parent].pb({node,wt});
            adj[node].pb({parent,wt});
        }
        
        
        for(auto child: adj2[node]){
            if(!vis[child.first]){
                pq.push({child.second, child.first, node});
            }
        }
    }
        
}
    
// dfs for binary-lifting precomputations
void dfs(int node, int par=0, int wt=0){

    dep[node] = dep[par]+1;
    anc[node][0] = par;

    mx[node][0]= wt;

    fr(i,1,LOG){
        anc[node][i] = anc[ anc[node][i-1] ][i-1];
        mx[node][i] = max( mx[ anc[node][i-1] ][i-1] , mx[node][i-1] );
    }

    for(auto child: adj[node]){
        if(child.first != par){
            dfs(child.first, node, child.second);
        }
    }
}

int lca(int u, int v){
    if(dep[u]< dep[v]) swap(u,v);

    revl(i,LOG){
        if( dep[ anc[u][i] ] >= dep[v] ){
            u = anc[u][i];
        }
    }

    if(u==v) return u;

    revl(i,LOG){
        if(anc[u][i] != anc[v][i]){
            u = anc[u][i];
            v = anc[v][i];
        }
    }

    return anc[u][0];

}

// maximum edge between a node and its ancestor
int maxedge(int node, int l){
    int dis = dep[node] - dep[l];

    int ans = 0;

    revl(i,LOG){
        if(dis & (1<<i)){
            ans = max( ans, mx[node][i]   );
            node= anc[node][i];
        }
    }

    return ans;

}

int query(int u, int v){

    int l = lca(u,v);

    return max( maxedge(u,l), maxedge(v,l) );

}



int tc = 1;
void SOLVE(){
    clear();
    cout<<"Case "<<tc++<<":"<<nl;

    int n,m; cin>>n>>m;
    while(m--){
        int a,b,c; cin>>a>>b>>c;
        adj2[a].pb({b,c});
        adj2[b].pb({a,c});
    }

    createMST();

    dfs(1);

    int q; cin>>q;
    while(q--){
        int u,v; cin>>u>>v;
        cout<<query(u,v)<<nl;
    }

}   

signed main(){
    ios_base::sync_with_stdio(0);
    cin.tie(0);
    int t=1;

    cin>>t;///////////////////// 

    while(t--) SOLVE();
    return 0;
}
```

---

## References

* https://cp-algorithms.com/graph/mst_prim.html
* https://cp-algorithms.com/graph/lca_binary_lifting.html

---

**Author:** Sadat
