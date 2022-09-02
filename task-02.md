## 1.3　怎样寻找最好的算法

### 例题1.3　总和最大区间问题　（难度系数3颗星)

给定一个实数序列，设计一个最有效的算法，找到一个总和最大的区间。

比如在下面的序列中：

1.5, −12.3, 3.2, −5.5, 23.2, 3.2, −1.4, −12.2, 34.2, 5.4, −7.8, 1.1, −4.9

总和最大的区间是从第5个数（23.2）到第10个数（5.4）。

#### 方法一， 三重循环 排列组合方法

共 *n(n + 1) / 2* 个区间 *O(n^2)*

每个区间平均有 *(n + 2) / 3* 个数， 共 *(n - 1) / 3* 次加法运算。 *O(n)*

因此算法复杂度为 *O(n^3)*

无用功： 加法运算多次重复。

cpp代码实现：

``` cpp
double arr[n];
// read arr

size_t begin{0};
size_t end{0};
double max{0.0};
double cur{0.0};

for (size_t length = 1; length != n; ++length) {
  for (size_t tmp_begin = 0, tmp_end = tmp_begin + length;
       tmp_end != n; ++tmp_begin, ++tmp_end) {
    cur = 0.0;
    for (size_t beg = tmp_begin; beg != tmp_end; ++beg) {
      cur += arr[beg];
    }
    if (cur > max) {
      max = cur;
      begin = tmp_begin;
      end = tmp_end;
    }
  }
}
```

#### 方法二， 两重循环

思路： 以计算出*S(p, q)*， 在计算\*S(p, q + 1)\*时， 只在原先基础上做一次加法。

初步想法： 用数组保存每个子区间和， 做两次遍历，
第一次p为1确定q的位置，
第二次q不变确定p的位置。
仔细考虑发现不需要数组， 只需保留最优值和位置。

似乎不小心找到了线性复杂度算法， 暂时不是很确信正确性。

重新考虑二重循环，在计算*p*的时候， 遍历*p + 1*到*n*,
从*1*到*n*依次遍历， 做两次循环取得结果，
复杂度*O(n^2)*。

cpp代码实现：

``` cpp
double arr[n];
// read arr

size_t begin{0};
size_t end{0};
double max{0.0};

double cur{0.0};
for (size_t tmp_begin = 0; tmp_begin == tmp_end; ++tmp_begin) {
  cur = 0.0;
  double in_max{0.0};
  size_t in_end{tmp_begin};

  size_t beg = tmp_begin;
  while (true) {
    if (cur > in_max) {
      in_max = cur;
      in_end = beg;
    }
    if (beg == n) {
      break;
    }
    cur += arr[beg];
    ++beg;
  }

  if (in_max > max) {
    max = in_max;
    begin = tmp_begin;
    end = in_end;
  }
}
```

#### 方法三， 分治算法

分治算法核心是划分和归并。

划分可以递归的逐步一分为二直至最小区间。

归并需要考虑一下几种情况：

1.  两个子区间是连续的。 此时我们可以直接将俩个区间合并， 在合并后的区间与这两个子区间之间寻找最大的一个。

2.  两个子区间不连续。 假设两个子区间为 *\[p1, q1\]*, *\[p2, q2\]* 这时需要分三种情况考虑：
    
    1.  *\[p1, q1\]*;
    2.  *\[p2, q2\]*;
    3.  *\[p1, q2\]*.
        在我看来， 第三种情况可视作 *\[p1, q1\]*, *\[q1 + 1, p2 - 1\]*, *\[p2, q2\]* 三个区间的和，
        可以肯定*S1 + S2 \< S1*, *S2 + S3 \< S3*, 但不能确定*S1 + S2 + S3* , *S1*, *S2* 之间的大小关系。

设计分治算法，
核心在于如何高效的计算区间 *\[p1, q2\]* 的和，
先以*O(n)*复杂度即依次累加的方式实现，
大约需要*log(n)*个区间和，
故复杂度为*O(nlogn)*。

以自顶向下的方式设计，

cpp代码实现：

``` cpp
double arr[n];
// read arr

auto maxsub(size_t begin, size_t end)
  -> pair<size_t, size_t> {
    if (begin + 1 == end) {
      return {begin, end, };
    }

    [b1, e1] = maxsub(begin, (begin + end) / 2);
    [b2, e2] = maxsub((begin + end) / 2, end);
    s1 = accumulate(b1, e1, 0);
    s2 = accumulate(b2, e2, 0);
    s3 = accumulate(b1, e2, 0);
    m = max({s1, s2, s3, });
    if (m == s1) {
      return {b1, e1};
    } else (m == s2) {
      return {b2, e2};
    } else {
      return {b1, e2};
    }
  }
```

以自底向上的方式设计理论可行， 目前未能给出代码实现。

#### 方法四， 正反两遍扫描

回到方法二中两次遍历的尝试，
第二遍遍历找左端点时， 自右向左遍历应为正确表述。
但是遇到了左端点在右端点右侧的错误情况。

经过阅读思考， 上述思路， 只能排除掉最左侧和最右侧不需要包含的值，
所得到的两个端点不一定是同一个区间的一对端点，
可能为不同区间的端点， 而最大子区间应为这些区间中的一个。

如书上所示， 两区间中任意一个与两区间之间间隔的和一定为负，
因此两区间及间隔的和不可能为最大， 最大子区间一定为其中一个区间。

cpp代码实现：

``` cpp
double arr[n];
// read arr

double max{0.0};
size_t left{0};
size_t right;
double sum;

size_t i = 0;
while (i != n;) {
  while (arr[i] < 0) {
    if (++i == n) {
      goto Out;
    }
  }
  sum = 0.0;
  size_t j = i;
  while (true) {
    if (sum > max) {
      max = sum;
      left = i;
      right = j;
    }
    if (sum < 0.0) {
      break;
    }
    if (j == n) {
      break;
    }
    sum += arr[j];
    ++j;
  }
  i = j
}
Out:
```

### 思考题 1.3

#### 1\. 将例题1.3的线性复杂度算法写成伪代码。（难度系数2颗星）

已按cpp代码实现此处略。

#### 2\. 在一个数组中寻找一个区间，使得区间内的数字之和等于某个事先给定的数字。 难度系数3颗星）

##### 方法一， 排列组合方法

以排列组合的方式列出所有区间组合， 并对每个区间累加。

复杂度*O(n^3)*

##### 方法二， 二重循环

减少重复累加， 依旧需要计算所有区间。

复杂度*O(n^2)*

``` cpp
double arr[n];
// read arr

double target;
// read target

size_t begin{0};
size_t end{0};

double sum{0.0};
for (size_t tmp_begin = 0; tmp_begin != tmp_end; ++tmp_begin) {
  sum = 0.0;
  size_t in_end{tmp_begin};

  size_t beg = tmp_begin;
  while (true) {
    if (sum == target) {
      begin = tmp_begin;
      end = beg;
    }
    if (beg == n) {
      break;
    }
    sum += arr[beg];
    ++beg;
  }
}
```

##### 更优方法分析

要满足等于这一苛刻条件， 则每个区间之间不具有偏序关系，
预计必须计算所有区间， 复杂度下限*O(n^2)*， 即不具有更优解。

正确性待之后考证。

#### 3\. 在一个二维矩阵中，寻找一个矩形的区域，使其中的数字之和达到最大值。 （难度系数4颗星）

##### 方法一， 排列组合方法

依据数学推理， *n \* m* 二维矩阵中共有 *n(n + 1)m(m + 1) / 4* 个矩形。
复杂度为*O(n^4)*。

计算每个矩形和复杂度约为*O(n^2)*。

##### 方法二， 四重循环

省去重复加法运算， 遍历每个小矩形。

##### 方法三， 正反两边遍历

此思路是一维情况的拓展，
第一遍通过遍历排除掉左侧、上侧不可能的小矩形， 确定左上角端点，
第二遍通过遍历排除掉右侧、下侧不可能的小矩形， 确定右下角端点。

通过类似一维的情况进行修正， 依次确定若干矩形， 从中确当最大矩形。

因为每个子区间并不能确保都可以直接合并， 需要计算累加和， 复杂度约为*O(n^2)*。

复杂度约为*O(n^4)*。

正确性待之后考证。

cpp代码实现：

``` cpp
double arr[n][m];
// read arr

size_t left{0};
size_t up{0};
size_t right{0};
size_t down{0};

double max{0.0};

double sum{0.0};
size_t i{0};
size_t j{0};

while (i != n) {
  j = 0;
  while (j != m) {
    if (arr[i][j] >= 0) {
      goto findrd; 
    }
    findlu:
    ++j;
  }
  ++i;
}

findrd:
size_t tmp_i{i};
size_t tmp_j{j};
while (tmp_i != n) {
  tmp_j = j;
  while (tmp_j != m) {
    sum += arr[tmp_i][tmp_j];
    if (sum > max) {
      right = tmp_i;
      down = tmp_j;
      max = sum;
    }
    if (sum < 0) {
      goto findlu;
    }
    ++tmp_j;
  }
  ++tmp_i;
}
```
