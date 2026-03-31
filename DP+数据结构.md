# ((★)) DP

看是不是**求最优、一步步决策** → 想 DP

抽 **最少信息** 描述当前局面 → 写出状态

想当前能**做哪些动作** → 写转移

确定**顺序**：从小状态推到大状态

写代码：初始化 → 循环 → 转移 → 答案

## 线性

```c++
#include<bits/stdc++.h>
using namespace std;
#define int long long
const int MAXN = 1e5 + 10;
const int MAXK = 30;
const long long INF = 1e18;

void solve() {
    int n, k, m;
    cin >> n >> k >> m; // 先读入n,k,m，再初始化数组！
    vector<int> a(n + 1);
    vector<int> sum(n + 1, 0);
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
        sum[i] = sum[i - 1] + a[i];
    }
    // DP状态定义：dp[i][j] = 处理完前i个节点，剩余j个回溯效果时的最大碎片数
    // j∈[0,k]，j=0表示无回溯影响
    vector<vector<int>> dp(n + 2, vector<int>(k + 1, -INF));
    dp[0][0] = 0; // 初始状态：处理完0个节点，无回溯效果，碎片数0

    for (int i = 0; i < n; i++) { // 处理完i个节点，接下来处理i+1号
        for (int j = 0; j <= k; j++) {
            if (dp[i][j] == -INF) continue; // 不可达状态跳过
            // 选择1：直接收集i+1号节点
            if (j == 0) {
                // 无回溯影响，拿完整a[i+1]，剩余效果还是0
                dp[i + 1][0] = max(dp[i + 1][0], dp[i][j] + a[i + 1]);
            } else {
                // 有回溯影响，拿a[i+1]/2，剩余效果减1
                dp[i + 1][j - 1] = max(dp[i + 1][j - 1], dp[i][j] + (a[i + 1] / 2));
            }
            // 选择2：触发时空回溯（仅当无回溯影响时才能触发）
            if (j == 0) {
                // 拿2倍a[i+1]，接下来k个节点被影响，剩余效果k
                dp[i + 1][k] = max(dp[i + 1][k], dp[i][j] + 2 * a[i + 1]);
            }
            // 选择3：时空跳跃
            int r = min(i + 1 + m, n); // 跳跃的终点（包含）
            int cnt = m + 1;
            int new_j = max(0LL, j - cnt); // 跳跃后剩余的回溯效果
            // 拿完整的前缀和，不受回溯影响，直接处理到r号节点
            dp[r][new_j] = max(dp[r][new_j], dp[i][j] + (sum[r] - sum[i]));
        }
    }

    // 答案是处理完n个节点的所有状态的最大值
    int ans = 0;
    for (int j = 0; j <= k; j++) {
        ans = max(ans, dp[n][j]);
    }
    cout << ans << endl;
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int t;
    cin >> t;
    while (t--) {
        solve();
    }
    return 0;
}
```



## 区间

​	**区间 dp 有一个性质——大区间包含小区间**

​	**★小区间递推大区间**  +**枚举分割点/左右端点的贡献**

```c++
//状态计算
	for(int len=2;len<=n;len++){		//1.阶段：枚举区间长度
		for(int l=1;l+len-1<=n;l++){    //2.状态：枚举区间起点
			int r=l+len-1;//区间终点
			for(int k=l;k<r;k++){		//3.决策：枚举分割点
				dp[l][r]=min(dp[l][r],dp[l][k]+dp[k+1][r]+pre[r]-pre[l-1]);
			}
		}
	}
状态，dp[l][r]     合并区间[l,r]需要的最小消耗
    dp[i][j][0]    表示的是第 i 人从左进来的方案数；
    dp[i][j][1]    表示的是第 j 人从右边进来的方案数。
```

## 背包

### 01背包：

​	要求**相乘**凑出价值：**内循环每次加**

```c++
 for (int x = 2; x <= n; x++) {
    if (have[x]) {
        for (int multiple = x; multiple <= n; multiple += x) {
            if (dp[multiple / x] != INF) {
                dp[multiple] = min(dp[multiple], dp[multiple / x] + 1);
            }
        }
    }
}
```

**背包内的物品不一定都是一样的，可以改变**

[P1941 [NOIP 2014 提高组\] 飞扬的小鸟 - 洛谷](https://www.luogu.com.cn/problem/P1941#submit)

## 树形

**注意初始化**

### 模板

```c++
void dfs(int u,int fa){
    f[u][1]=1,f[u][0]=0;
    for(int v:g[u]){
        if(v==fa)continue;
        dfs(v,u);
        f[u][0]+=f[v][1];
        f[u][1]+=min(f[v][0],f[v][1]);
    }   
}
```

### 基环树

并查集，把**唯一环去掉**，找到**分割点**；

两遍**BFS**

[P2016 [SEERC 2000\] 战略游戏 - 洛谷](https://www.luogu.com.cn/problem/P2016)

```c++
dfs(S,0);
ans = f[S][0];
dfs(T,0);
ans = max(ans, f[T][0]);
```



### 树型背包

#### dfs序做法

我们把原树叫做 A。

定义　 多叉树的**后序遍历**指的是：在搜索某个结点的过程中，先记录它的所有子树，再记录它本身。注意，如果有多个子树，则后序遍历的顺序不限。

定义　 多叉树的**后序遍历序列**指的是在上述过程中所记录下来的序列。

我们不妨在 DFS 后把结点按照后序遍历序列**重新编号**。下图就是一个例子，左图为原树 A，右图为重新编号的树 B。

![image-20260312162818037](typora-user-images\image-20260312162818037.png)

因此，设 dp(*i*,*j*) 表示将树 B 的结点 1…*i* 放入新图，背包容量为 *j* 时，所能取得的最大价值。设 size*i* 表示以 *i* 为根的子树的大小。

若取物品 i（前提是此时的背包容量放得下物品 i），则可以取它的子树，则
问题转化为「将结点 1…i−1 加入 C，且背包容量为 j−1 时，所能取到的最大价值」加上物品 i 的价值，
所以答案为 dp(i−1,j−1)+v i ；
若不取物品 i，则不可以取它的子树，则
问题转化为「将『结点 1…i−1 中不属于 i 的子树的结点』加入 C，背包容量不变时，所能取到的最大价值」
答案为$dp(i−size_i,j)$；

$ dp(i,j) =  \begin{cases} \max\bigl(dp(i-1,j-1)+v_i,\ dp(i-\text{size}_i,j)\bigr) & j \ge w_i \\ dp(i-\text{size}_i,j) & j < w_i \end{cases} $

[P1273 [CHCI 2002 Final Exam #2\] 有线电视网 - 洛谷](https://www.luogu.com.cn/problem/P1273)

```c++
vector<int> adj[N];
int n,m,c[N];
int idx[N],sz[N],tot;
// dfs序
void dfs(int u){
    sz[u]=1;
    for(int v : adj[u]){
        dfs(v);
        sz[u]+=sz[v];
    }
    idx[++tot]=u;
}  
........
    
 	dfs(1);// 从根节点开始DFS
    vector<vector<int>>f(tot+1,vector<int>(m+1,-INF));
    // dp[u][j]表示以u为根的子树选j个用户的最大收益（成本-收益）
    for(int i=0;i<=tot;i++){
        f[i][0]=0;
    }//all weight is 1,the 成本 is val;
    for(int i=1;i<=tot;i++){//遍历物品
        int u=idx[i];
        for(int j=1;j<=m;j++){//遍历容量
            if(u>=n-m+1){// 叶子节点
                f[i][j]=max(f[i-1][j-1]+c[u],f[i-1][j]);//选or不选
            }else{      //非叶子节点
                f[i][j]=max(f[i-1][j]+c[u],f[i-sz[u]][j]);
            }
        }
    }
    // 查找最大的k使得价值不小于0
    for(int i=m;i>=0;i--){
        if(f[tot][i]>=0){
            cout << i << endl;
            return 0;
        }
    }
```



#### 做法二

```c++
#include <bits/stdc++.h>
using namespace std;
const int maxn = 3005;
const int inf = 0x3f3f3f3f;
int n, m, a[maxn];
vector<vector<pair<int, int>>> adj(maxn);
int f[maxn][maxn];// DP数组：f[x][j] 表示以x为根的子树选j个用户时的最大净收益
// dfs返回以x为根的子树的用户数量（叶子节点数）
int dfs(int x, int fa) {
    // 叶子节点（用户终端）
    if (x > n - m) {
        f[x][1] = a[x];  // 选这个用户的收益就是他愿意支付的钱
        return 1;        // 子树只有1个用户
    }
    int siz = 0;  // 以x为根的子树的总用户数
    for (auto &p : adj[x]) {  // 遍历x的所有邻接节点
        int y = p.first;      // 子节点编号
        int w = p.second;     // 传输到y的费用
        if (y == fa) continue;

        int g = dfs(y, x);  // 递归计算子树y的用户数
        siz += g;

        // 树形背包DP：合并子树y的状态
        for (int j = siz; j >= 1; --j) {//遍历容量//逆序确保（j-k）无后效性
            for (int k = 1; k <= min(j, g); ++k) {//其他
                f[x][j] = max(f[x][j], f[x][j - k] + f[y][k] - w);
            }
        }
    }
    return siz;
}

int main() {
    memset(f, -inf, sizeof(f));
    for (int i = 1; i <= n; ++i) {
        f[i][0] = 0;
    }
    cin >> n >> m;
    for (int i = 1; i <= n - m; ++i) {
        int kksk;
        cin >> kksk;  // 子节点数量
        for (int j = 1; j <= kksk; ++j) {
            int aa, cc;
            cin >> aa >> cc;  // 子节点编号、传输费用
            // 双向加边（树的无向存储，dfs时通过fa避免回头）
            adj[i].emplace_back(aa, cc);
            adj[aa].emplace_back(i, cc);
        }
    }
    for (int i = n - m + 1; i <= n; ++i) {
        cin >> a[i];
    }
    // 从根节点1开始DFS，父节点为-1（无父节点）
    dfs(1, -1);
    for (int i = m; i >= 1; --i) {
        if (f[1][i] >= 0) {
            cout << i << endl;
            return 0;
        }
    }
    cout << 0 << endl;
    return 0;
}
```



### 基环树



## 状压

[Problem - B - Codeforces](https://codeforces.com/gym/103495/problem/B)

```c++
#include<bits/stdc++.h>
using namespace std;
#define ll long long
#define int long long
#define ull unsigned long long
#define all(v) v.begin(),v.end()
using PII=pair<int,int>;
const int N=1e6+5,mod=998244353,INF=1e18;//
//cout<<fixed<<setprecision(10);
int n,m,k,x,y;
int e,max_t;
// vector<int>f=  dij(x,mov,adj);
vector<int> dij(int s,vector<vector<vector<int>>>& mov,vector<vector<PII>> &adj){
    vector<vector<int>>dp(1<<k,vector<int>(n+1,INF));
    dp[0][s]=0;
    for(int i=0;i<(1<<k);i++){
        //zou
        priority_queue<PII,vector<PII>,greater<PII>>pq;
        for(int j=1;j<=n;j++){
            if(dp[i][j]<INF){
                pq.emplace(dp[i][j],j);
            }
        }
        while(pq.size()){
            auto [t,u]=pq.top();
            pq.pop();
            if(t>dp[i][u])continue;
            for(auto [v,w]:adj[u]){
                if(t+w<dp[i][v]){
                    dp[i][v]=t+w;
                    pq.emplace(dp[i][v],v);
                }
            }
        }
        //砍人 S→S|(1<<p)    杀人会改变集合 S，所以必须放在走路之后。
        for(int j=1;j<=n;j++){
            int now_t=dp[i][j];
            if(now_t>=INF||now_t>max_t)continue;
            for(int p=0;p<k;p++){
                if(i&(1<<p))continue;
                if(mov[p][j].empty())continue;
                auto it=lower_bound(all(mov[p][j]),now_t);
                if(it==mov[p][j].end())continue;
                if(*it>max_t)continue;
                int now_s=i|(1<<p);
                if(dp[now_s][j]>*it){
                    dp[now_s][j]=*it;
                }
            }
        }
    }
    vector<int>res;
    for(int i=0;i<(1<<k);i++){
        int ans=INF;
        for(int j=1;j<=n;j++)ans=min(ans,dp[i][j]);
        if(ans>max_t)ans=INF;
        res.push_back(ans);
    }
    return res;


}
void solve(){
    cin>>n>>m>>k;
    vector<vector<PII>>adj(n+1);
    vector<vector<vector<int>>>mov(k,vector<vector<int>>(n+1));
    for(int i=0;i<m;i++){
        int x,y,w;cin>>x>>y>>w;
        adj[x].push_back({y,w});
        adj[y].push_back({x,w});
    }
    cin>>e>>max_t;
    for(int i=0;i<e;i++){
        int p,x,t;
        cin>>p>>x>>t;
        p--;
        mov[p][x].push_back(t);
    }
    for(int i=0;i<k;i++){
        for(int j=1;j<=n;j++){
            sort(all(mov[i][j]));
        }
    }
    int ans=INF;
    int s1=(1<<k)-1;
    cin>>x>>y;
    vector<int>f=  dij(x,mov,adj);
    vector<int>f2= dij(y,mov,adj);
    for(int i=0;i<=s1;i++){
        int s2=s1^i;
        if(f[i]<INF&&f2[s2]<INF){
            ans=min(ans,max(f[i],f2[s2]));
        }
    }
        if (ans <= max_t) {
            cout << ans << '\n';
        } else {
            cout << -1 << '\n';
        }
}
signed main(){
    ios::sync_with_stdio(0);
    cin.tie(0);cout.tie(0);
    int t=1;
    cin>>t;
    while(t--){
        solve();
    }
    return 0;
}
```



## 数位

​	[2376. 统计特殊整数 - 力扣（LeetCode）](https://leetcode.cn/problems/count-special-integers/description/)请你返回区间 `[1, n]` 之间**”**每一个数位都是**互不相同**的整数**“**的数目。

确保一定正确就dfs的**所有参数记忆化**       要想优化，就**存储&提取 的条件一致**

```c++
 vector<vector<int>> memo(m, vector<int>(1 << 10, -1)); // -1 表示没有计算过
        auto dfs = [&](auto&& dfs, int i, int mask, bool is_limit, bool is_num) -> int {
            if (i == m) {
                return is_num; // is_num 为 true 表示得到了一个合法数字
            }
            if (!is_limit && is_num && memo[i][mask] != -1) {
                return memo[i][mask]; // 之前计算过    isLimit 的条件下不可以使用缓存数据
            }
            int res = 0;
            if (!is_num) { // 可以跳过当前数位
                res = dfs(dfs, i + 1, mask, false, false);
            }
            // 如果前面填的数字都和 n 的一样，那么这一位至多填数字 s[i]（否则就超过 n 啦）
            int up = is_limit ? s[i] - '0' : 9;
            // 枚举要填入的数字 d
            // 如果前面没有填数字，则必须从 1 开始（因为不能有前导零）
            for (int d = is_num ? 0 : 1; d <= up; d++) {
                if ((mask >> d & 1) == 0) { // d 不在 mask 中，说明之前没有填过 d
                    res += dfs(dfs, i + 1, mask | (1 << d), is_limit && d == up, true);
                }
            }
            memo[i][mask] = res; // 记忆化
            return res;
        };
```

[1012. 至少有 1 位重复的数字 - 力扣（LeetCode）](https://leetcode.cn/problems/numbers-with-repeated-digits/solutions/1748539/by-endlesscheng-c5vg/)求在 `[1, n]` 范围内具有 **至少 1 位** 重复数字的正整数的个数。

**正难则反**，转换成求无重复数字的个数。答案等于 ***n* 减去无重复数字**的个数。

```c++
int numDupDigitsAtMostN(int n) {
    auto s = to_string(n);
    int m = s.length(), memo[m][1 << 10];
    memset(memo, -1, sizeof(memo)); // -1 表示没有计算过
    function<int(int, int, bool, bool)> 
        f = [&](int i, int mask, bool is_limit, bool is_num) -> int {
        if (i == m)
            return is_num; // is_num 为 true 表示得到了一个合法数字
        if (!is_limit && is_num && memo[i][mask] != -1)
            return memo[i][mask];
        int res = 0;
        if (!is_num) // 可以跳过当前数位
            res = f(i + 1, mask, false, false);
        int up = is_limit ? s[i] - '0' : 9; 
        for (int d = 1 - is_num; d <= up; ++d) // 枚举要填入的数字 d
            if ((mask >> d & 1) == 0) // d 不在 mask 中
                res += f(i + 1, mask | (1 << d), is_limit && d == up, true);
        if (!is_limit && is_num)
            memo[i][mask] = res; // 记忆化搜索
        return res;
    };
    return n - f(0, 0, true, false);
}
```

```c++
Fun( i ,is_mit ,is_num){
   	1.边界条件 return;
    2.判断是否可以调用dp数组
       res=0;
    3 .(1) !is_num // 可以跳过当前数位
      .(2) for(d-up) // 枚举要填入的数字 d
    4 存储dp
    5.return res;
}
```

2.0 求区间[low,high]，不用is_num

```c++
// 代码示例：返回 [low, high] 中的恰好包含 target 个 0 的数字个数
// 比如 digitDP(0, 10, 1) == 2
// 要点：我们统计的是 0 的个数，需要区分【前导零】和【数字中的零】，前导零不能计入，而数字中的零需要计入
long long digitDP(long long low, long long high, int target) {
    string low_s = to_string(low);
    string high_s = to_string(high);
    int n = high_s.size();
    int diff_lh = n - low_s.size();
    vector memo(n, vector<long long>(target + 1, -1));
    auto dfs = [&](this auto&& dfs, int i, int cnt0, bool limit_low, bool limit_high) -> long long {
        if (cnt0 > target) {
            return 0; // 不合法
        }
        if (i == n) {
            return cnt0 == target;
        }
        if (!limit_low && !limit_high && memo[i][cnt0] >= 0) {
            return memo[i][cnt0];
        }
        int lo = limit_low && i >= diff_lh ? low_s[i - diff_lh] - '0' : 0;
        int hi = limit_high ? high_s[i] - '0' : 9;
        long long res = 0;
        int d = lo;
        // 通过 limit_low 和 i 可以判断能否不填数字，无需 is_num 参数
        // 如果前导零不影响答案，去掉这个 if block
        if (limit_low && i < diff_lh) {
            // 不填数字，上界不受约束
            res = dfs(i + 1, 0, true, false);
            d = 1;
        }
        for (; d <= hi; d++) {
            // 统计 0 的个数
            res += dfs(i + 1, cnt0 + (d == 0), limit_low && d == lo, limit_high && d == hi);
            // res %= MOD;
        }
        if (!limit_low && !limit_high) {
            memo[i][cnt0] = res;
        }
        return res;
    };
    return dfs(0, 0, true, true);
}
```

## 二次扫描/换根DP

### 模板一

[P3478 [POI 2008\] STA-Station - 洛谷](https://www.luogu.com.cn/problem/P3478)

**前序遍历+后序遍历**

```c++
// 第一遍DFS计算子树大小和深度
void dfs1(int x,int fa,int dep)
{
    sum+=dep;
    size[x]=1;
    for(int y : adj[x]) {
        if(y==fa) continue;
        dfs1(y,x,dep+1);
        size[x]+=size[y];
    }
}
// 第二遍DFS计算每个节点的距离和
void dfs2(int x,int fa){
    for(int y : adj[x]){
        if(y==fa) continue;
        f[y]=f[x]+n-2*size[y];
        dfs2(y,x);
    }
}
```

## 状压DP

**状态压缩 DP：**

1. 用二进制表示状态，用**十进制数**存储状态；
2. 用**位运算**筛选出合法状态；
3. 用**位运算**判断状态转移的条件；
4. 计算时每个类**累加**上一行的**兼容类**。

### 状态转移的条件

自身合法性：状态`s`中无相邻的 1（如同一行不能放相邻棋子）：`(s & (s << 1)) == 0`

转移合法性：当前状态`s`与上一行状态`pre`无重叠（如上下行棋子不上下对齐）：`(s & pre) == 0`

**攻击距离 2**：自身无相邻 / 隔位 1：`(s & (s << 1)) == 0 && (s & (s << 2)) == 0`

**斜向相邻**：上下行状态不能斜向重叠：`(pre & (s << 1)) == 0 && (pre & (s >> 1)) == 0`

**统计状态中 1 的个数**：`__builtin_popcount(s)`（GCC 内置函数，O (1)）

哈密顿路径：最终状态需覆盖所有点：`s == (1 << n) - 1`

**位运算速记**：

- `s & (s - 1)`：消去 s 的最后一个 1（统计 1 的个数 / 枚举子集）
- `s | (1 << k)`：将第 k 位设为 1（添加元素）
- `s & ~(1 << k)`：将第 k 位设为 0（移除元素）

### 模板题

[P1879 [USACO06NOV\] Corn Fields G - 洛谷](https://www.luogu.com.cn/problem/P1879)

```c++
const int p=100000000;
ll f[13][1<<12],n,m;
ll g[1<<12],h[1<<12],a[13][13];
ll F[13];
int main()
{
	scanf("%d%d",&n,&m);
	for(int i=1;i<=n;i++)
		for(int j=1;j<=m;j++)
			scanf("%d",&a[i][j]);
    //预处理肥沃
	for(int i=1;i<=n;i++)
		for(int j=1;j<=m;j++)
			F[i]=(F[i]<<1)+a[i][j];
    vector<int>G;
    //预处理合法
	for(int i=0;i<(1<<m);i++){
		if(!(i&(i>>1))){//横着不连续
            G.push_back(i);
			if((i&F[1])==i) f[1][i]=1;//选的都是肥沃的
		}
	}
	for(int x=2;x<=n;x++)
        for(int j:G)
			if(((j&F[x-1])==j))//上一行状态（肥沃的&&横着不连续）
                for(int k:G)
					if(((k&F[x])==k)&&!(j&k)){//当前行肥沃的&&横着不连续&&竖着不连续
						f[x][k]=(f[x][k]+f[x-1][j])%p;
					}
	int ans=0;
	for(int i=0;i<(1<<m);i++)
		ans=(ans+f[n][i])%p;
	printf("%d\n",ans);
    return 0;
}
```

## 期望DP/概率DP



## 数据结构优化



# 数据结构



## 单调队列

## 区间维护核心结构

| 名称                       | 具体作用                                                     | 时间复杂度                          | 代码量   |
| -------------------------- | ------------------------------------------------------------ | ----------------------------------- | -------- |
| **树状数组（BIT）**        | 单点修改 + 前缀和 / 前缀最值查询；扩展差分后可**区间修改 + 区间查询**；代码极短常数极小，是区间题首选 | 单点 / 区间操作 O (logn)            | 15-20 行 |
| **普通线段树（带懒标记）** | 维护区间和、区间最值、区间 gcd；支持区间加、区间乘、区间赋值等懒标记；解决**所有满足结合律的区间问题** | 建树 O (n)；操作 O (logn)           | 40-60 行 |
| **ST 表（稀疏表）**        | 静态数组**区间最值查询**，不支持修改，查询效率 O (1) 碾压其他结构；也可用于树上 LCA（欧拉序实现） | 预处理 O (nlogn)；查询 O (1)        | 20-30 行 |
| 分块（块状数组）           | 暴力美学，解决线段树难以维护的操作（如区间众数、区间开根号、区间模运算） | 单次操作 O (√n)                     | 30-40 行 |
| 主席树                     | 静态区间第 k 大、二维偏序、区间不同数计数、树上路径第 k 大、可持久化，保存历史版本 | 预处理 O (nlogn)，单次查询 O (logn) |          |

## 五、连通性维护

| 名称       | 具体作用                                                     | 时间复杂度                        | 代码量   |
| ---------- | ------------------------------------------------------------ | --------------------------------- | -------- |
| **并查集** | 管理集合连通性；扩展域 / 带权并查集可解决**二分图判定、种类并查集（如食物链）**；是 Kruskal 算法的核心 | 所有操作均摊 O (α(n))（近似常数） | 15-20 行 |

------

## 六、字符串专属结构

| 名称                       | 考场实现                      | 具体作用                                                     | 时间复杂度                           | 代码量   |
| -------------------------- | ----------------------------- | ------------------------------------------------------------ | ------------------------------------ | -------- |
| **字符串哈希（滚动哈希）** | 必手写（BKDR / 双哈希防碰撞） | 将字符串映射为数值，**O (1) 比较子串是否相等**；代替 KMP/SA 的轻量场景，如回文判定、子串匹配 | 预处理 O (n)；查询 O (1)             | 15-20 行 |
| **KMP（前缀函数）**        | 必手写                        | 单模式串匹配、循环节查找、子串定位                           | 预处理 O (n)；匹配 O (m)             | 20-25 行 |
| **字典树（Trie）**         | 必手写（数组模拟）            | 字符串前缀匹配、最大异或对（01-Trie）、字典序排序；是 AC 自动机的基础 | 插入 / 查询 O (L)（L 为字符串长度）  | 20-30 行 |
| **AC 自动机**              | 必手写（Trie+KMP fail 指针）  | 多模式串匹配、敏感词过滤、多模式串定位                       | 建 Trie/fail 指针 O (ΣL)；匹配 O (m) | 40-60 行 |
| 后缀自动机（SAM）          | 手写（可选，省选以上）        | 线性存储所有子串，解决不同子串计数、子串出现次数、最长公共子串 | 构建 O (n)                           | 50-70 行 |

------

## 七、图论 / 树专属结构

| 名称                     | 考场实现           | 具体作用                                                     | 时间复杂度                      | 代码量   |
| ------------------------ | ------------------ | ------------------------------------------------------------ | ------------------------------- | -------- |
| **树上倍增**             | 必手写             | 预处理节点的 2^k 级祖先，实现**高效 LCA 查询**、树上路径最值 / 和查询 | 预处理 O (nlogn)；查询 O (logn) | 30-40 行 |
| **树链剖分（重链剖分）** | 必手写（省选以上） | 将树拆分为重链，转为线性序列，用线段树 / BIT 维护**树上路径 / 子树修改与查询** | 预处理 O (n)；操作 O (log²n)    | 60-80 行 |

------

## 考场关键提醒

1. **优先用 STL**：栈、队列、优先队列、set/map 直接用 STL，绝不手写，节省时间。
2. **必写模板背熟**：树状数组、线段树（带懒标记）、并查集、链式前向星、字符串哈希、KMP、Trie、AC 自动机、树上倍增 —— 这些是**底线**，必须背到能闭着眼写。
3. **复杂结构放弃**：红黑树、LCT、树套树、珂朵莉树（除非有大量推平操作）—— 考场上除非时间充裕且准备充分，否则直接跳过，换用其他方法。

