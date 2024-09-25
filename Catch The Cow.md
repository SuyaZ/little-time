[3278 -- Catch That Cow (poj.org)](http://poj.org/problem?id=3278)
[P1588 [USACO07OPEN] Catch That Cow S - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P1588)

## 题目描述

FJ 丢失了他的一头牛，他决定追回他的牛。已知 FJ 和牛在一条直线上，初始位置分别为 $x$ 和 $y$，假定牛在原地不动。FJ 的行走方式很特别：他每一次可以前进一步、后退一步或者直接走到 $2\times x$ 的位置。计算他至少需要几步追上他的牛。

## 输入格式

第一行为一个整数 $t\ ( 1\le t\le 10)$，表示数据组数；

接下来每行包含一个两个正整数 $x,y\ (0<x,y \le 10^5)$，分别表示 FJ 和牛的坐标。

## 输出格式

对于每组数据，输出最少步数。

## 样例 #1

### 样例输入 #1

```
1 
5 17
```

### 样例输出 #1

```
4
```


## 题解

这是一道非常经典的BFS题，只需要将各个点视作一个状态，状态就是有限的。
但是对于DFS解法就需要剪枝，并且加以限制防止无限递归。


这里是对 n 作递归，会出现 n 出现无穷大的不可控的情况，由于考虑到 n 不能出现按无穷大，于是想办法返回一个非常大的数，让这种情况不会被考虑到。

==但是会出现后续出界的n掩盖前面计算的有效值的情况==
下面这个代码是有错误的，
```cpp
#include<iostream>
#include<cmath>

using namespace std;
const int NN = 4e5+10;

int N, K;
//int res = 0x3f3f3f3f;//保存最小值
bool vis[NN];//false
int dis;

int dfs(int n, int step)
{
    if(vis[n] == false)
    {
        vis[n] = true;
        
        if(n == K) return step;
        
        if(abs(n-K) > dis) return 0x3f3f3f3f;
        else if(abs(n-K) == dis && n != N) return 0x3f3f3f3f;
        else
        {
            int res;
            int x1 = dfs(n+1, step+1);
            int x2 = dfs(n-1, step+1);
            int x3 = dfs(2*n, step+1);
            
            if(x1 < x2) res = x1;
            if(res > x3) res = x3;
            
            return res;
            cout<<res<<" ***"<<endl;
        }
    }
}


int main()
{
    cin>>N>>K;
    dis = abs(N-K);
    
    int ans = dfs(N, 0);
    cout<<ans;
    
    return 0;
}
```

### DFS + 剪枝优化

#DFS #剪枝

分析：
![[微信图片_20240925183612.png]]
![[微信图片_20240925183631.png]]
![[微信图片_20240925183634.png]]

AC代码：
```cpp
#include<iostream>

using namespace std;
int x, y;
int res;

int abs(int xx)
{
    if(xx < 0) return -xx;
    else return xx;
}

void dfs(int x, int y, int step)
{
    
    //res == step && y != x --> res < step + abs(x-y)
    if(res <= step) return;//可行性剪枝
        
    //step < res, 但是 step + abs(x-y) 不一定小于res
    res = min(res, step + abs(y-x));
    
    if(x == y) return;
    else if(x > y) return;
    else
    {
        // //res == step && y != x --> res < step + abs(x-y)
        // if(res <= step) return;//可行性剪枝
        
        // //step < res, 但是 step + abs(x-y) 不一定小于res
        // res = min(res, step + abs(y-x));
        
        if(y % 2 == 0)
        {
            dfs(x, y/2, step+1);
        }
        else
        {
            dfs(x, y-1, step+1);
            dfs(x, y+1, step+1);
        }
    }
}

int main()
{
    int t;
    cin>>t;
    
    while(t--)
    {
        cin>>x>>y;
        res = abs(x - y);
        
        if(x < y) dfs(x, y, 0);
        
        cout<<res<<endl;
    }
    
    return 0;
}
```

但是，以下dfs代码对于样例
```
输入
1
5 17

期望输出
4
```
却错误地返回了结果 `5`.
剪枝位置的正确性如此重要。实际上，在上一层代码对step做修改之后，就需要在下次递归及时地作优化。
```cpp
void dfs(int x, int y, int step)
{   
    if(x == y) return;
    else if(x > y) return;
    else
    {
        // //res == step && y != x --> res < step + abs(x-y)
        if(res <= step) return;//可行性剪枝
        
        // //step < res, 但是 step + abs(x-y) 不一定小于res
        res = min(res, step + abs(y-x));
        
        if(y % 2 == 0)
        {
            dfs(x, y/2, step+1);
        }
        else
        {
            dfs(x, y-1, step+1);
            dfs(x, y+1, step+1);
        }
    }
}
```
![[微信图片_20240925183638.png]]
程序错误地返回了。