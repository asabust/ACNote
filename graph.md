# 第三章 搜索与图论

## DFS 与 BFS

| -   | 数据结构 | 空间     | 特点               |
| --- | -------- | -------- | ------------------ |
| DFS | stack    | $O(h)$   | 执着，没有“最短性” |
| BFS | queue    | $O(2^h)$ | 稳重，“最短路”     |

### DFS 回溯和剪枝

## 图的存储

- 树是一种特殊的图，与图的存储方式相同。
- 对于无向图中的边 ab，存储两条有向边 a->b, b->a。
- 因此我们可以只考虑有向图的存储。

(1) 邻接矩阵：g[a][b] 存储边 a->b

(2) 邻接表：

```c++
// 对于每个点 k，开一个单链表，存储 k 所有可以走到的点。h[k]存储这个单链表的头结点
int h[N], e[N], ne[N], idx;

// 添加一条边 a->b
void add(int a, int b)
{
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++ ;
}

// 初始化
idx = 0;
memset(h, -1, sizeof h);
```

## 树与图的遍历

时间复杂度 $O(v+e)$, `v` 表示点数，`e` 表示边数

> 无向图是一种特殊的有向图，每条边都是两向。
> 无向图建图的时候要建两条边。

### 深度优先遍历

[846. 树的重心](/3.搜索与图论/5-846.%20树的重心.md)

> 深度优先遍历可以求子树的大小。

```c++
int dfs(int u)
{
st[u] = true; // st[u] 表示点 u 已经被遍历过

    for (int i = h[u]; i != -1; i = ne[i])
    {
        int j = e[i];
        if (!st[j]) dfs(j);
    }
}
```

### 宽度优先遍历

模板题 [AcWing 847. 图中点的层次](/3.搜索与图论/6-847.%20图中点的层次.md)

```c++
queue<int> q;
st[1] = true; // 表示1号点已经被遍历过
q.push(1);

while (q.size())
{
    int t = q.front();
    q.pop();

    for (int i = h[t]; i != -1; i = ne[i])
    {
        int j = e[i];
        if (!st[j])
        {
            st[j] = true; // 表示点j已经被遍历过
            q.push(j);
        }
    }
}
```

## 拓扑排序

[848. 有向图的拓扑序列](/3.搜索与图论/7-848.%20有向图的拓扑序列.md)

时间复杂度 $O(V+E)$, $V$ 表示点数，$E$表示边数

```c++
bool topsort()
{
    int hh = 0, tt = -1;

    // d[i] 存储点i的入度
    for (int i = 1; i <= n; i ++ )
        if (!d[i])
            q[ ++ tt] = i;

    while (hh <= tt)
    {
        int t = q[hh ++ ];

        for (int i = h[t]; i != -1; i = ne[i])
        {
            int j = e[i];
            if (-- d[j] == 0)
                q[ ++ tt] = j;
        }
    }

    // 如果所有点都入队了，说明存在拓扑序列；否则不存在拓扑序列。
    return tt == n - 1;
}
```

## 最短路问题

1. 单源路最短路径：从某一个点到其他点的最短路。

   1. 所有边都是正权边。
      - [朴素-dijkstra](#朴素-dijkstra-算法) $O(V^2 + E)$ 适用于稠密图，基于贪心
      - [堆优化 dijkstra](#堆优化版-dijkstra) $O(ElogV)$ 稠密图反而更低效$O(V^2logV)$
   1. 存在负权边。
      - [Bellman-Ford](#bellman-ford-算法) $O(EV)$
      - [SPFA](#spfa-判断图中是否存在负环) 平均 $O(E)$ 最坏 $O(EV)$ Bellman-Ford 的优化

2. 多源汇最短路：一个图多个查询，有多个起点终点
   - [Floyd](#floyd-算法) $O(V^3)$ 基于动态规划

**最短路的难点是建图，把普通问题抽象成图的问题，转化成最短路问题来解决。**

### 朴素 dijkstra 算法

模板题 [849. Dijkstra 求最短路 I](/3.搜索与图论/8-849.%20Dijkstra求最短路%20I%20.md)

时间复杂是 $O(n^2+m)$, $n$ 表示点数，$m$ 表示边数

```c++
int g[N][n]; // 存储每条边
int dist[N]; // 存储 1 号点到每个点的最短距离
bool st[N]; // 存储每个点的最短路是否已经确定

// 求 1 号点到 n 号点的最短路，如果不存在则返回-1
int dijkstra()
{
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;

    for (int i = 0; i < n - 1; i ++ )
    {
        int t = -1;     // 在还未确定最短路的点中，寻找距离最小的点
        for (int j = 1; j <= n; j ++ )
            if (!st[j] && (t == -1 || dist[t] > dist[j]))
                t = j;

        // 用t更新其他点的距离
        for (int j = 1; j <= n; j ++ )
            dist[j] = min(dist[j], dist[t] + g[t][j]);

        st[t] = true;
    }

    if (dist[n] == 0x3f3f3f3f) return -1;
    return dist[n];
}
```

### 堆优化版 dijkstra

[850. Dijkstra 求最短路 II](/3.搜索与图论/9-850.%20Dijkstra求最短路%20II%20.md)

时间复杂度 $O(mlogn)$, `n` 表示点数，`m` 表示边数

```c++
typedef pair<int, int> PII;

int n; // 点的数量
int h[N], w[N], e[N], ne[N], idx; // 邻接表存储所有边
int dist[N]; // 存储所有点到 1 号点的距离
bool st[N]; // 存储每个点的最短距离是否已确定

// 求 1 号点到 n 号点的最短距离，如果不存在，则返回-1
int dijkstra()
{
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;
    priority_queue<PII, vector<PII>, greater<PII>> heap;
    heap.push({0, 1}); // first 存储距离，second 存储节点编号

    while (heap.size())
    {
        auto t = heap.top();
        heap.pop();

        int ver = t.second, distance = t.first;

        if (st[ver]) continue;
        st[ver] = true;

        for (int i = h[ver]; i != -1; i = ne[i])
        {
            int j = e[i];
            if (dist[j] > distance + w[i])
            {
                dist[j] = distance + w[i];
                heap.push({dist[j], j});
            }
        }
    }

    if (dist[n] == 0x3f3f3f3f) return -1;
    return dist[n];

}
```

### Bellman-Ford 算法

[853. 有边数限制的最短路](/3.搜索与图论/10-853.%20有边数限制的最短路.md)

时间复杂度 $O(nm)$, $n$ 表示点数，$m$ 表示边数
注意在模板题中需要对下面的模板稍作修改，加上备份数组，详情见模板题。

```c++
int n, m; // n 表示点数，m 表示边数
int dist[N]; // dist[x]存储 1 到 x 的最短路距离

struct Edge // 边，a 表示出点，b 表示入点，w 表示边的权重
{
    int a, b, w;
}edges[M];

// 求 1 到 n 的最短路距离，如果无法从 1 走到 n，则返回-1。
int bellman_ford()
{
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;

    // 如果第n次迭代仍然会松弛三角不等式，就说明存在一条长度是n+1的最短路径，由抽屉原理，路径中至少存在两个相同的点，说明图中存在负权回路。
    for (int i = 0; i < n; i ++ )
    {
        for (int j = 0; j < m; j ++ )
        {
            int a = edges[j].a, b = edges[j].b, w = edges[j].w;
            if (dist[b] > dist[a] + w)
                dist[b] = dist[a] + w;
        }
    }

    if (dist[n] > 0x3f3f3f3f / 2) return -1;
    return dist[n];

}
```

### spfa 算法（队列优化的 Bellman-Ford 算法

[851. spfa 求最短路](/3.搜索与图论/11-851.%20spfa求最短路.md)

时间复杂度 平均情况下 $O(m)$，最坏情况下 $O(nm)$, `n` 表示点数，`m` 表示边数

```c++
int n; // 总点数
int h[N], w[N], e[N], ne[N], idx; // 邻接表存储所有边
int dist[N]; // 存储每个点到 1 号点的最短距离
bool st[N]; // 存储每个点是否在队列中

// 求 1 号点到 n 号点的最短路距离，如果从 1 号点无法走到 n 号点则返回-1
int spfa()
{
memset(dist, 0x3f, sizeof dist);
dist[1] = 0;

    queue<int> q;
    q.push(1);
    st[1] = true;

    while (q.size())
    {
        auto t = q.front();
        q.pop();

        st[t] = false;

        for (int i = h[t]; i != -1; i = ne[i])
        {
            int j = e[i];
            if (dist[j] > dist[t] + w[i])
            {
                dist[j] = dist[t] + w[i];
                if (!st[j])     // 如果队列中已存在j，则不需要将j重复插入
                {
                    q.push(j);
                    st[j] = true;
                }
            }
        }
    }

    if (dist[n] == 0x3f3f3f3f) return -1;
    return dist[n];

}
```

### spfa 判断图中是否存在负环

[852. spfa 判断负环](/3.搜索与图论/12-852.%20spfa判断负环.md)

时间复杂度是 $O(nm)$, `n` 表示点数，`m` 表示边数

```c++
int n; // 总点数
int h[N], w[N], e[N], ne[N], idx; // 邻接表存储所有边
int dist[N], cnt[N]; // dist[x]存储 1 号点到 x 的最短距离，cnt[x]存储 1 到 x 的最短路中经过的点数
bool st[N]; // 存储每个点是否在队列中

// 如果存在负环，则返回 true，否则返回 false。
bool spfa()
{
// 不需要初始化 dist 数组
// 原理：如果某条最短路径上有 n 个点（除了自己），那么加上自己之后一共有 n+1 个点，由抽屉原理一定有两个点相同，所以存在环。

    queue<int> q;
    for (int i = 1; i <= n; i ++ )
    {
        q.push(i);
        st[i] = true;
    }

    while (q.size())
    {
        auto t = q.front();
        q.pop();

        st[t] = false;

        for (int i = h[t]; i != -1; i = ne[i])
        {
            int j = e[i];
            if (dist[j] > dist[t] + w[i])
            {
                dist[j] = dist[t] + w[i];
                cnt[j] = cnt[t] + 1;
                if (cnt[j] >= n) return true;       // 如果从1号点到x的最短路中包含至少n个点（不包括自己），则说明存在环
                if (!st[j])
                {
                    q.push(j);
                    st[j] = true;
                }
            }
        }
    }

    return false;
}
```

### floyd 算法

[854. Floyd 求最短路](/3.搜索与图论/13-854.%20Floyd求最短路.md)

时间复杂度是 $O(n^3)$, n 表示点数

```c++
初始化：
for (int i = 1; i <= n; i ++ )
    for (int j = 1; j <= n; j ++ )
        if (i == j) d[i][j] = 0;
            else d[i][j] = INF;

// 算法结束后，d[a][b]表示 a 到 b 的最短距离
void floyd()
{
    for (int k = 1; k <= n; k ++ )
        for (int i = 1; i <= n; i ++ )
            for (int j = 1; j <= n; j ++ )
                d[i][j] = min(d[i][j], d[i][k] + d[k][j]);
}
```

## 最小生成树

- 最小生成树针对无向图而言。

### 朴素版 prim 算法

- 稠密图：朴素 $O(n^2)$
- 堆优化：稀疏图 不常用 $O(mlogn)$

[858. Prim 算法求最小生成树](/3.搜索与图论/14-858.%20Prim算法求最小生成树.md)

时间复杂度是 $O(n^2+m)$, `n` 表示点数，`m` 表示边数

```c++
int n; // n 表示点数
int g[N][n]; // 邻接矩阵，存储所有边
int dist[N]; // 存储其他点到当前最小生成树的距离
bool st[N]; // 存储每个点是否已经在生成树中

// 如果图不连通，则返回 INF(值是 0x3f3f3f3f), 否则返回最小生成树的树边权重之和
int prim()
{
    memset(dist, 0x3f, sizeof dist);

    int res = 0;
    for (int i = 0; i < n; i ++ )
    {
        int t = -1;
        for (int j = 1; j <= n; j ++ )
            if (!st[j] && (t == -1 || dist[t] > dist[j]))
                t = j;

        if (i && dist[t] == INF) return INF;

        if (i) res += dist[t];
        st[t] = true;

        for (int j = 1; j <= n; j ++ ) dist[j] = min(dist[j], g[t][j]);
    }

    return res;

}
```

### Kruskal 算法

[859. Kruskal 算法求最小生成树](/3.搜索与图论/15-859.%20Kruskal算法求最小生成树.md)

时间复杂度是 `O(mlogm)`, `n` 表示点数，`m` 表示边数

```c++
int n, m; // n 是点数，m 是边数
int p[N]; // 并查集的父节点数组

struct Edge // 存储边
{
int a, b, w;

    bool operator< (const Edge &W)const
    {
        return w < W.w;
    }

}edges[M];

int find(int x) // 并查集核心操作
{
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

int kruskal()
{
sort(edges, edges + m);

    for (int i = 1; i <= n; i ++ ) p[i] = i;    // 初始化并查集

    int res = 0, cnt = 0;
    for (int i = 0; i < m; i ++ )
    {
        int a = edges[i].a, b = edges[i].b, w = edges[i].w;

        a = find(a), b = find(b);
        if (a != b)     // 如果两个连通块不连通，则将这两个连通块合并
        {
            p[a] = b;
            res += w;
            cnt ++ ;
        }
    }

    if (cnt < n - 1) return INF;
    return res;
}
```

## 二分图

### 染色法判别二分图

[860. 染色法判定二分图](/3.搜索与图论/16-860.%20染色法判定二分图.md)

时间复杂度是 $O(n+m)$, n 表示点数，m 表示边数

```c++
int n;      // n表示点数
int h[N], e[M], ne[M], idx;     // 邻接表存储图
int color[N];       // 表示每个点的颜色，-1表示未染色，0表示白色，1表示黑色

// 参数：u表示当前节点，c表示当前点的颜色
bool dfs(int u, int c)
{
    color[u] = c;
    for (int i = h[u]; i != -1; i = ne[i])
    {
        int j = e[i];
        if (color[j] == -1)
        {
            if (!dfs(j, !c)) return false;
        }
        else if (color[j] == c) return false;
    }

    return true;
}

bool check()
{
    memset(color, -1, sizeof color);
    bool flag = true;
    for (int i = 1; i <= n; i ++ )
        if (color[i] == -1)
            if (!dfs(i, 0))
            {
                flag = false;
                break;
            }
    return flag;
}
```

### 匈牙利算法

[861. 二分图的最大匹配](/3.搜索与图论/17-861.%20二分图的最大匹配.md)

时间复杂度是 $O(nm)$, n 表示点数，m 表示边数。
实际运行时间远小于$O(nm)$

```c++
int n1, n2;     // n1表示第一个集合中的点数，n2表示第二个集合中的点数
int h[N], e[M], ne[M], idx;     // 邻接表存储所有边，匈牙利算法中只会用到从第一个集合指向第二个集合的边，所以这里只用存一个方向的边
int match[N];       // 存储第二个集合中的每个点当前匹配的第一个集合中的点是哪个
bool st[N];     // 表示第二个集合中的每个点是否已经被遍历过

bool find(int x)
{
    for (int i = h[x]; i != -1; i = ne[i])
    {
        int j = e[i];
        if (!st[j])
        {
            st[j] = true;
            if (match[j] == 0 || find(match[j]))
            {
                match[j] = x;
                return true;
            }
        }
    }

    return false;
}

// 求最大匹配数，依次枚举第一个集合中的每个点能否匹配第二个集合中的点
int res = 0;
for (int i = 1; i <= n1; i ++ )
{
    memset(st, false, sizeof st);
    if (find(i)) res ++ ;
}
```
