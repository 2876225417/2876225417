# Algorithm_Tutorial

## Quick_Sort ##

### 快速排序 ###
Luogu [P1177](https://www.luogu.com.cn/problem/P1177)
    将读入的 N 个数从小到大排序后输出。

### 输入格式 ###
第一行为一个正整数 N。
第二行包含 N 个空格隔开的正整数a~i~ 。

### 输出格式 ###
将给定的 N 个数从小到大输出，数之间空格隔开，行未换行且无空格。

对于 100% 的数据，有 1 ≤ N ≤ 10^5^ ，1 ≤ a~i~ ≤ 10^9^。

样例：
```
输入
8
9 1 7 6 6 3 2 8
```
```
输出
1 2 3 6 6 7 8 9
```

快速排序主要利用**分治思想**，时间复杂度**O(nlogn)**。
```
int n, a[100005];

void quicksort(int l, int r){
    if(l == r) return;
    int i = l - 1, j = r + 1, x = q[(l + r) >> 1];
    while(i < j){
        do i++; while(q[i] < x);    // 向右查找 ≥x 的数
        do j--; while(q[j] > x);    // 向左查找 ≤x 的数
        if(i < j) swap(q[i], q[j]);
    }
    quicksort(l, j), quicksort(j + 1, r);
}
```
### 思路如下 ###
1. 利用**i（左指针）**,**j（右指针）** 指向数列的区间外侧，数列的中值记为**x**。
2. 将数列中 **≤x** 的数放左段，**≥x** 的数放右段。
3. 对于左右两段，再递归以上两个过程，直到每段只有一个数，即全部有序。
![quick_sort1.png](quick_sort1.png)

### 过程模拟 ###
![quick_sort2.png](quick_sort2.png)

### 补充 ###
![quick_sort4.png](quick_sort4.png)

即在中间值的左边找一个大于中间值的数m~1~, 在中间值的右边找一个小于中间值的数m~2~, 并且当m~1~小于m~2~时，才交换m~1~和m~2~，交换完成后再进行分割，再对分割的区域重复上述的操作。

**STL实现**
```
#include <iostream>
#include <algorithm>
using namespace std;

int n, a[100005];

int main(){
    cin >> n;
    for(int i = 0; i < n; i++) scanf("%d", &a[i]);
    sort(a, a + n);
    for(int i = 0; i < n; i++) printf("%d", &a[i]);
    return 0;
}
```

__一般实现__
```
#include <iostream>
using namespace std;

int n, a[100005];

void quicksort(int l, int r){
    if(l == r) return;
    int i = l - 1, j = r + 1, x = q[(l + r) >> 1];
    while(i < j){
        do i++; while(q[i] < x);    // 向右查找 ≥x 的数
        do j--; while(q[j] > x);    // 向左查找 ≤x 的数
        if(i < j) swap(q[i], q[j]);
    }
    quicksort(l, j), quicksort(j + 1, r);
}

int main(){
    cin >> n;
    for(int i = 0; i < n; i++) scanf("%d", &a[i]);
    quicksort(0, n - 1);
    for(int i = 0; i < n; i++) printf("%d", &a[i]); 
    return 0; 
}
```

### 题目 ###
Luogu [P1923](https://www.luogu.com.cn/problem/P1923) 求第 k 小的数

输入 _n_ （1 ≤ _n_ ≤ 5000000 且 _n_ 为奇数）个数字a~i~ (1 ≤ a~i~ ≤ 10^9^) ，
输出这些数字的第 _k_ 小的数。最小的数是第 0 小。

请尽量不要使用 ``` nth_element ``` 来写本题，因为本题的重点在于练习分治算法。
因为``` nth_element ```可以待```quicksort```后再输出**nth_element**，这种做法会消耗多余的时间去处理多余的一半部分。 

样例:
```
输入
5 1
4 3 2 1 5
```
```
输出
2
```
![kth_number.png](kth_number.png)

**STL实现**
```
#include <iostream>
using namespace std;

int n, k, a[5000010];

int main(){
    cin >> n >> k;
    for(int i = 0; i < n; i++) scanf("%d",&a[i]);
    nth_element(a, a + k, a + n);
    printf("%d\n", a[k]);
    return 0;
}
```

**一般实现**
```
#include <iostream>
using namespace std;

int n, k, q[5000010];

template<typename T>
void Swap(T& t1_, T& t2_){
    T t_ = t1_;
    t1_ = t2_;
    t2_ = t;
}

void kth_number(int q[], int l, int r, int k){
    if(l >= r） return q[l]; 
    
    int i = l - 1, j = r + 1, x = q[(l + r) >> 1];
    while(i < j){
        do i++; while(q[i] < x);
        do j--; while(q[j] > x);
        if(i < j) Swap(q[i], q[j]); 
    }
    if(k <= j) return kth_number(q, l, j, k);
    else return kth_number(q, j + 1, r, k);
}

int main(){
    cin >> n >> k;
    for(int i = 0; i < n; i++) scanf("%d", q[i]);
    cout << kth_number(q, 0, n - 1, k);
    return 0;
}

```
## Merge_Sort ##

### 归并排序 ###
Luogu [P1177](https://www.luogu.com.cn/problem/P1177)
    将读入的*N*个数从小到大排序后输出

### 输入格式 ###
第一行为一个正整数*N*。
第二行包含*N*个空格隔开的正整数a~i~ ，为你需要进行排序的数。

### 输出格式 ###
将给定的*N*个数从小到大输出，数之间空格隔开，行末换行且无空格。

对于20%的数据，有1≤*N*≤10^3^;

对于100%的数据，有1≤*N*≤10^5^ ，1≤a~i~≤10^9^。

### 样例 ###
```
输入
5
5 2 4 5 1
```
```
输出
1 2 4 4 5
```

归并排序主要利用**分治思想**，时间复杂度为O(*nlogn*)。

```
int n, a[100010], b[100010];    /* 此处a，b为辅助数组 */

void merge_sort(int q[], int l, int r){
    if(l >= r) return;
    int mid = (l + r) >> 1;
    merge_sort(q, l, mid);
    merge_sort(q, mid + 1, r);  /* 递归拆分 */
    
    int i = l, j = mid + 1, k = l;  /* 合并 */
    while(i <= mid && j <= r){
        if(a[i] <= a[j]) b[k++] = a[i++];
        else b[k++] = a[j++];
    }
    
    while(i <= mid) b[k++] = a[i++];
    while(j <= r) b[k++] = a[j++];
    for(i = l; i <= r; i++) a[i] = b[i];
}
### 思路如下 ###
```
1. 对数列不断进行等长**拆分**，直到为**一个数**的长度;
2. 回溯时，按升序**合并**左右两段。
3. 重复以上两个过程，直到递归结束。

**划分**

![divide.png](divide.png)

```
合并：
1. i,j 分别指向a的左右段起点，k指向b的起点。
2. 枚举a数组，如果左数≤右数，把左数放入b数组，否则，把右数放入b数组。
3. 把左段或右端剩余的数放入b数组。
4. 把b数组的当前段复制回a数组。
```
![conjunct.png](conjunct.png)

|     |  快速排序  |   归并排序 |
|-----|:------:|-------:|
 | 分治  | 先交换后拆分 | 先拆分后交换 |
| 稳定性 |  不稳定   |     稳定 |

### 完整实现如下 ###
```
#include <iostream>
using namespace std;
int n, a[100010], b[100010];

void merge_sort(int l, int r){
    if(l >= r) return;
    int mid = (l + r) >> 1;
    merge_sort(l, mid); 
    merge_sort(mid + 1, r);

    int i = l, j = mid + 1, k = l;
    while(i <= mid && j <= r)
        if(a[i] <= a[j]) b[k++] = a[i++];
        else b[k++] = a[j++];

    while(i <= mid) b[k++] = a[i++];
    while(j <= r) b[k++] = a[j++];
    for(i = l; i <= r; i++) a[i] = b[i];
}

int main(){
    cin >> n;
    for(int i = 0; i < n; i++) scanf("%d", &a[i]);
    merge_sort(0, n - 1);
    for(int i = 0; i < n; i++) printf("%d ", a[i]);
    return 0;
}
```

Luogu [P1908](https://www.luogu.com.cn/problem/P1908)逆序对 

对于给定的一段正整数序列，逆序对就是序列中a~i~ > a~j~ 且*i*<*j*的有序对。注意序列中可能有重复数字。

### 输入格式 ###
第一行，一个数n，表示序列中有*n*个数。

第二行*n*个数，表示给定的序列。序列中每个数字不超过10^9^。

对于25%的数据，n≤2500。

对于50%的数据，n≤4 x 10^4^。

对于所有数据，n≤5 x 10^5^。



### 输出格式 ###
输出序列中逆序对的数目。
```
输入
6 
5 4 2 6 3 1
```
```
输出
11
```

### 示例 ###
![reverse_pair.png](reverse_pair.png)
### 实现方法 ###
```
#include <iostream>
using namespace std;

typedef long long LL;

int n, a[5000010], b[5000010];
LL res = 0;

void reverse_pair(int l, int r){
    if(l >= r) return;
    int mid = (l + r) >> 1;
    reverse_pair(l, mid);
    reverse_pair(mid + 1, r);
    
    int i = l, j = mid + 1, k = l;
    while(i <= mid && j <= r)
        if(a[i] <= a[j]) b[k++] = a[i++];
        else b[k++] = a[j++], res += mid - i + 1;
     
    while(i <= mid) b[k++] = a[i++];
    while(j <= r) b[k++] = a[j++];
    for(i = l; i <= r; i++) a[i] = b[i];
}

int main(){
    cin >> n;
    for(int i = 0; i < n; i++) scanf("%d", &a[i]);
    reverse_pair(0, n - 1);
    cout << res;
    return 0;
}
```
`解释 of 'res += mid - i + 1': 当合并两个有序数组时，如果a[i]>a[j]，说明a[i]及其后面的元素都比a[i]大，因为数组是有序的。所以，对于当前的a[j]来说，它与前半部分数组中剩余的元素构成逆序对。`

Luogu [P1966](https://www.luogu.com.cn/problem/P1966)火柴排队


### 题目描述 ###

涵涵有两盒火柴，每盒装有 *n* 根火柴，每根火柴都有一个高度。 现在将每盒中的火柴各自排成一列， 同一列火柴的高度互不相同， 两列火柴之间的距离定义为：**∑(a~i~ - b~i~)^2^**

其中 a~i~ 表示第一列火柴中第 *i* 个火柴的高度，b~i~ 表示第二列火柴中第  *i* 个火柴的高度。

每列火柴中相邻两根火柴的位置都可以交换，请你通过交换使得两列火柴之间的距离最小。请问得到这个最小的距离，最少需要交换多少次？如果这个数字太大，请输出这个最小交换次数对 **10^8^ - 3** 取模的结果。

### 输入格式 ###

共三行，第一行包含一个整数 *n*，表示每盒中火柴的数目。

第二行有 *n* 个整数，每两个整数之间用一个空格隔开，表示第一列火柴的高度。

第三行有 *n* 个整数，每两个整数之间用一个空格隔开，表示第二列火柴的高度。

### 输出格式 ###

一个整数，表示最少交换次数对 **10^8^ - 3** 取模的结果。

### 样例1 ###
```
输入
4
2 3 1 4
3 2 1 4
```
```
输出
1
```
### 样例2 ###
```
输入
4
1 3 4 2
1 7 2 4
```
```
输出
2
```

**输入输出样例说明一**

最小距离是 *0*，最少需要交换 *1* 次，比如：交换第 *1* 列的前 *2* 根火柴或者交换第 *2* 列的前 *2* 根火柴。

**输入输出样例说明二**

最小距离是 *10*，最少需要交换 *2* 次，比如：交换第 *1* 列的中间 *2* 根火柴的位置，再交换第 *2* 列中后 *2* 根火柴的位置。

**数据范围**

对于 10% 的数据， 1 ≤ *n* ≤ 10^1^；

对于 30% 的数据，1 ≤ *n* ≤ 10^2^；

对于 60% 的数据，1 ≤ *n* ≤ 10^3^；

对于 100% 的数据，1 ≤ *n* ≤ 10^5^，0 ≤ 火柴高度 ≤ 2^31^。

### 整体思路 ###
1. 根据每盒火柴的高度，用快速排序`quick_sort`对其从小到大进行排序
2. 利用第**2**盒火柴中的位置对第**1**盒火柴中的位置进行映射，使之与排序之前的位置相同
3. 再利用归并排序`reverse_pair`计算逆序数(即要交换的次数，题目中指出相邻的两根火柴可交换)。

### 实现方法 ###
```
#include <cstdio>

struct node{
    int num, ord;
    bool operator<(node b1_){ return num < b1_.num; }
    bool operator>(node b2_){ return num > b2_.num; }
}f[100010], s[100010];

template<typename T>
void Swap(T& t1_, T& t2_){
    T t_ = t1_;
    t1_ = t2_;
    t2_ = t_;
}

int a[100010], b[100010], n, ans = 0;

// 类比归并排序求逆序数
void reverse_pair(int l, int r){
    if(l >= r) return;
    int mid = (l + r) >> 1;
    reverse_pair(l, mid);
    reverse_pair(mid + 1, r);
    
    int i = l, j = mid + 1, k = l;
    while(i <= mid && j <= r)
        if(a[i] <= a[j]) b[k++] = a[i++];
        else{
            b[k++] = a[j++];
            ans += mid - i + 1; // 求逆序数
            ans %= 99999997;    // 处理结果
        }
        
    while(i <= mid) b[k++] = a[i++];
    while(j <= r) b[k++] = a[j++];
    for(i = l; i <= r; i++) a[i] = b[i];
}

// 一般快速排序
void quick_sort(node q[], int l, int r){
    if(l >= r) return;
    int i = l - 1, j = r + 1;
    node x = q[(l + r) >> 1];
    while(i < j){
        do i++; while(q[i] < x);
        do j--; while(q[j] > x);
        if(i < j) Swap(q[i], q[j]);
    }
    quick_sort(q, l, j), quick_sort(q, j + 1, r);
}

int main(){
    // 读入数据
    scanf("%d", &n);
    for(int i = 1; i <= n; i++){
        scanf("%d", &f[i].num);
        f[i].ord = i;
    }
    
    for(int i = 1; i <= n; i++){
        scanf("%d", &s[i].num);
        s[i].ord = i;
    }
    
    // 根据高度对两盒火柴进行排序
    quick_sort(f, 1, n);    
    quick_sort(s, 1, n);
    
    // 映射火柴的顺序
    for(int i = 1; i <= n; i++) a[f[i].ord] = s[i].ord;
    
    reverse_pair(1, n);
    printf("%d", ans);
    
    return 0;
}
```

## Binary_Search ##
Luogu [P2249](https://www.luogu.com.cn/problem/P2249) 查找

### 题目描述 ###
输入 *n* 个不超过 10^9^ 的单调不减的（就是后面的数字不小于前面的数字）非负整  
数a~1~, a~2~, ... , a~n~, 然后进行 *m* 次询问。对于每次询问，给出一个整数 *q*，要求输出这个数字在序列中第一次出现的编号，如果没有找的话输出 -1。

### 输入格式 ###
第一行**2**个整数*n*和*m*，表示数字个数和询问次数。

第二行*n*个整数，表示这些待查询的数字。

第三行*m*个整数，表示询问这些数字的编号，从1开始编号。

### 输出格式 ###
输出一行，m个整数，以空格隔开，表示答案。

### 样例 ###
```
输入
11 3
1 3 3 3 5 7 9 11 13 15 15
1 3 6
```
```
输出
1 2 -1
```

数据保证，1 ≤ *n* ≤ 10^6^ ， 0 ≤ *a~i~ , q* ≤ 10^9^, 1 ≤ *m* ≤ 10^5^

### 思路 ###
1. 向存在目标值的区间缩小
2. 同时注意区间问题

### 模板一 ###
```
#include <iostream> 
using namespace std;

int n, m, q, a[1000005];

int find(int q){
    int l = 0, r = n + 1;
    while(l + 1 < r){   // 开区间
        int mid = (l + r) >> 1;
        if(a[mid] >= q) r = mid;
        else l = mid;
    }
    return a[r] == q ? r : -1;
}

int main(){
    scanf("%d %d, &n, &m);
    for(int i = 1; i <= n; i++) scanf("%d", &a[i]);
    for(int i = 1; i <= m; i++) scanf("%d", &q), printf("%d ", find(q));
    return 0;
}
```
当要查找的值不在左右边界时

![binary_search.png](binary_search.png)

当要查找的值在左右边界时

![binary_search_.png](binary_search_.png)

最大化和最小化查找
最大化即找最大下标
最小化即找最小下标

![binary_search__.png](binary_search__.png)

### 模板二 ###
```
/* 闭区间 */
#include <cstdio>

int n, m, q, a[1000005];

int find(int k){
    int l = 1, r = n;   /* 闭区间 */
    while(l < r){
        int mid = (l + r) >> 1;
        if(a[mid] >= k) r = mid;
        else l = mid + 1; /* *** */
    }
    return a[r] == k ? r : -1;
}

int main(){
    scanf("%d %d", &n, &m);
    for(int i = 1; i <= n; i++) scanf("%d", &a[i]);
    for(int i = 1; i <= m; i++) scanf("%d", &q), printf("%d ", find(q));
    return 0;
}
```
### 模板三 ###
```
#include <cstdio>

int n, m, q, a[1000005];

int find(int k){
    int ans = 0;
    int l = 1, r = n; /* 闭区间 */
    while(l <= r){  // l == r + 1 时结束
        int mid = (l + r) >> 1;
        if(a[mid] >= q) ans = mid, r = mid - 1; /* 利用 ans 记录 mid
        else l = mid + 1;
    }
    return a[ans] == q ? ans : -1;
}

int main(){
    scanf("%d %d", &n, &m);
    for(int i = 1; i <= n; i++) scanf("%d", &a[i]);
    for(int i = 1; i <= m; i++) scanf("%d", &q), printf("%d ", find(q));
    return 0;
}
```
### 模板四 ###
```
/* 算法库实现 */
#include <cstdio>
#include <algorithm>

int n, m, q, a[1000005];

int main(){
    scanf("%d %d", &n, &m);
    for(int i = 1; i <= n; i++) scanf("%d", &a[i]);
    for(int i = 1; i <= m; i++){
        scanf("%d", &q);
        int ans = lower_bound(a + 1, a + n + 1, q) - a; // 缩小区间
        if(a[ans] == q) printf("%d ", ans);
        else printf("-1 ");
    }
    return 0;
}
```
Luogu [P1024](https://www.luogu.com.cn/problem/P1024) 一元三次方程求解

### 题目描述 ###
有形如：*ax^3^ + bx^2^ + cx + d = 0* 这样的一个一元三次方程。给出该方程中各项的系数（*a,b,c,d*均为实数），
并约定该方程存在三个不同的实根（根的范围在 -100 至 100 之间），且根于根之差的绝对值 ≥ 1。要求由小到大依次在同一
行输出这三个实根（根于根之间留有空格），并精确到小数点后2位。

### 输入格式 ###
一行，*4*个实数*a,b,c,d*。

### 输出格式 ###
一行，*3*个实根，从小到大输出，并精确到小数点后2位。

### 样例 ###
```
输入
1 -5 -4 20
```
```
输出
-2.00 2.00 5.00
```

![root1_.png](root1_.png)

![root2_.png](root2_.png)

### 实现 ###
```
#include <cstdio>

double a, b, c, d;

double fun(double x) { return a * x * x * x + b * x * x + c * x + d; }

double root(double l, double r){
    while(r - l > 0.0001){  // 精度控制
        double mid = (l + r) / 2;
        if(fun(mid) * fun(l) < 0) r = mid;
        else l = mid;
    }
    return r;
}

int main(){
    scanf("%lf %lf %lf %lf", &a, &b, &c, &d);
    for(int i = -100; i < 100; i++){
        double y1 = fun(i), y2 = fun(i + 1);
        if(!y1) printf("%.2lf ", i * 1.0);  // 若 i 本身就是根的情况
        if(y1 * y2 < 0) printf("%.2lf ", root(i, i + 1));
    }
    return 0;
}
```

## High_Precision ##

### 高精度加法 ###

### 题目描述 ###
高精度加法，相当于 a + b problem, 需用考虑负数

### 输入格式 ###
分两行输入。 *a, b ≤ 100^500^。*

### 输出格式 ###
输出只有一行，代表 *a + b*的值。

样例
```
输入
1001
9099
```
```
输出
10100
```

### 实现一 ###
```
#include <iostream>
using namespace std;

const int N = 505;
int a[N], b[N], c[N];
int la, lb, lc;

void add(int a[], int b[], int c[]){
    for(int i = 1; i <= lc; i++){
        c[i] += a[i] + b[i];
        c[i + 1] = c[i] / 10;
        c[i] %= 10;
    }
    if(c[lc + 1]) lc++; // 若 a = 900, b = 300, c(lc = 3) = 1300
                        // 此时 lc = 3，故需要特判 lc 位置的下一位，检测是否还有数字
}

int main(){
    string sa, sb; cin >> sa >> sb;
    la = sa.size(), lb = sb.size(), lc = max(la, lc);
    // 因为读入时 高位在后，所以要逆序读入
    for(int i = 1; i <= la; i++) a[i] = sa[la - i] - '0'; //将 sa 中的数字逆序存入 a 中
    for(int i = 1; i <= lb; i++) b[i] = sb[lb - i] - '0'; //同理
    add(a, b, c);
    for(int i = lc; i; i--) printf("%d", c[i]); //逆序输出     
    return 0;
}

```
### 实现二 ###
```
#include <iostream>
#include <vector>
using namespace std;

typedef vector<int> VI;
VI a, b, c;
int la, lb, lc;

void add(VI& a, VI& b, VI& c){
    int t = 0;
    for(int i = 0; i < lc; i++){
        if(i < la) t += a[i];
        if(i < lb) t += b[i];
        c.push_back(t % 10);
        t /= 10;
    }
    if(t) c.push_back(t);   
}

int main(){
    string sa, sb; cin >> sa >> sb;
    la = sa.size(), lb = sb.size(), lc = max(la, lb);
    for(int i = la - 1; ~i; i--) a.push_back(sa[i] - '0');
    for(int i = lb - 1; ~i; i--) b.push_back(sb[i] - '0');
    add(a, b, c);
    for(int i = c.size() - 1; ~i; i--) printf("%d", c[i]);    
    return 0;
}
```

### 高精度减法 ###

### 题目描述 ###
高精度减法

### 输入格式 ###
两个整数 *a, b*（第二个可能比第一个大）

### 输出格式 ###
结果（是负数要输出负号）

### 样例 ###
```
输入
2
1
```
```
输出
1
```

### 实现一 ###
```
#include <iostream>
using namespace std;

const int N = 100050;
int a[N], b[N], c[N];
int la, lb, lc;

bool cmp(int a[], int b[]){
    if(la != lb) return la < lb;
    for(int i = la; i; i--) if(a[i] < b[i]) return a[i] < b[i];
    return false;
}

void sub(int a[], int b[], int c[]){
    for(int i = 1; i <= lc; i++){
        if(a[i] < b[i]) a[i + 1]--, a[i] += 10;
        c[i] = a[i] + b[i];
    }
    while(c[lc] == 0 && lc > 1) lc--;
}

int main(){
    string sa, sb; cin >> sa >> sb;
    for(int i = 1; i < la; i++) a[i] = sa[la - i] - '0';
    for(int i = 1; i < lb; i++) b[i] = sb[lb - i] - '0';
    if(cmp(a, b) cout << "-";
    sub(a, b, c);
    for(int i = lc; i; i--) printf("%d", c[i]);
    
    return 0;
}


```

### 实现二 ###
```
#include <iostream>
#include <vector>
using namespace std;

typedef vector<int> VI;
VI a, b, c;

bool cmp(VI a, VI b){
    if(a.size() != b.size()) return a.size() < b.size();
    for(int i = a.size(); ~i; i--) if(a[i] != b[i]) return a[i] < b[i];
    return false;
}

void sub(VI a, VI b, VI& c){
    int t = 0;
    for(int i = 0; i < a.size(); i++){
        t = a[i];
        if(i < b.size()) t -= b[i];
        if(t < 0) a[i + 1]--, t += 10;
        c.push_back(t);
    }
    while(c.size() > 1 && !c.back()) c.size()--;
}

int main(){
    string sa, sb; cin >> sa >> sb;
    for(int i = sa.size() - 1; ~i; i--) a.push_back(sa[i] - '0');
    for(int i = sb.size() - 1; ~i; i--) b.push_back(sb[i] - '0');
    if(cmp(a, b)) swap(a, b), cout << '-'";
    sub(a, b, c);
    for(int i = c.size() - 1; ~i; i--) printf("%d", c[i]);
    return 0;
}

```

## Prefix_Sum & Difference ##

## Dual_Pointers ##

## Bit_Operation ##

## Discretization ##

## Merge_Parts ##

### Edited by ppQwQqq ###