# 面试刷题：计数排序矩阵中的负数 | 第44期

## 1. 题目 (Leetcode Q1351)

给定一个m * n矩阵grid，该grid按行和列均以非递增顺序排序。
返回grid中的负数。

## 2. 约束条件

(a) m == grid.length

(b) n == grid[i].length

(c) 1 <= m, n <= 100

(d) -100 <= grid[i][j] <= 100

## 示例 1

```
Input: grid = [[4,3,2,-1],[3,2,1,-1],[1,1,-1,-2],[-1,-1,-2,-3]]
Output: 8
```
解释：在矩阵中有8个负数

## 示例 2

```
Input: grid = [[3,2],[1,0]]
Output: 0
```

## 示例 3

```
Input: grid = [[1,-1],[-1,-1]]
Output: 3
```

## 示例 4

```
Input: grid = [[-1]]
Output: 1
```

## 3. 解决思路 1

阅读题目之后就会发现这是一个非常简单的问题，因为有一个非常直观的解决思路，即将矩阵中的每一个元素都遍历一遍，同时计数出负数的个数。对于如此简单的思路，就算是刚入门编程的朋友也会很快实现一个解决方案，这里就直接给出一个C++的实现。

```C++
class Solution {
public:
    // Runtime: 20 ms, faster than 60.28%
    // Memory Usage: 9.5 MB, less than 100.00%
    int countNegatives(vector<vector<int>>& grid) {
        int ret = 0;
        for (auto & r : grid) {
            for (auto & v : r) {
                if (v < 0) {
                    ++ret;
                }
            }
        }
        return ret;
    }
};
```

从评测结果来看，超过了60%的提交，不算太好，但也勉勉强强。按照这个思路使用Python实现一个如下：

```Python
class Solution:
    # Runtime: 124 ms, faster than 81.38%
    # Memory Usage: 14 MB, less than 100.00%
    def countNegatives(self, grid: List[List[int]]) -> int:
        ret = 0
        for row in grid:
            for v in row:
                if v < 0:
                    ret += 1
        return ret
```

从评测结果来看，Python的解决方案超过了81%的提交，看起来似乎还不错，但我更愿意相信这是因为Python的提交总量太少，而不是这种解决思路比较好。毕竟，我们还有一个很明显的前提条件没有使用，即矩阵有序。

## 4. 解决思路 2

该如何来利用题目中所提到的“行和列均以非递增顺序排序”这个条件呢？从这个条件出发，我们会发现以下3点：

1. 任意一行如果存在负数的话，那么负数一定排在最后面

2. 任意一列如果存在负数的话，那么负数一定排在最后面

3. 如果任意一位置是正数的话，那么其左侧和上侧的所有位置都必定是正数

基于以上三个判断，我们可以形成一个思路，即从最后一行开始，每一行的行尾往行首查找，遇到正数就停止该行的搜索，并记录该位置列索引为A；然后开始搜索上一行，还是从行尾开始搜索，搜到A列或者提前遇到正数停止；如此循环，直到搜索完所有行。

从时间复杂度上来看，在最坏的情况下，与第一种思路一样，都是 O(n^2)，但是从实际运行的角度来看，可以减少一些不必要的遍历。这里首先给出这种思路的C++实现的解决方案。

```C++
class Solution {
public:
    // Runtime: 16 ms, faster than 93.57%
    // Memory Usage: 9.4 MB, less than 100.00%
    int countNegatives(vector<vector<int>>& grid) {
        int m = grid.size() - 1, n = grid[0].size() - 1;
        int ret = 0, end = 0, c = 0;
        for (; m >= 0; --m) {
            for (c = n; c >= end; --c) {
                if (grid[m][c] < 0) {
                    ++ret;
                } else {
                    end = c + 1;
                    break;
                }
            }
            if (end > n) {
                break;
            }
        }
        return ret;
    }
};
```

从评测结果来看，第二种解决思路超过了93%的提交，相比较第一种C++解决方案而言，还是有一些提升。这个应该是在意料之中，毕竟在不全负数的情况下能够减少一些计算。接下来我们使用Python实现以下这种思路。

```Python
class Solution:
    # Runtime: 132 ms, faster than 31.87%
    # Memory Usage: 14 MB, less than 100.00%
    def countNegatives_2(self, grid: List[List[int]]) -> int:
        m, n = len(grid) - 1, len(grid[0]) - 1
        end, ret = 0, 0
        while m >= 0:
            c = n
            while c >= end:
                if grid[m][c] < 0:
                    ret += 1
                else:
                    end = c + 1
                    break
                c -= 1
            if end > n:
                break
            m -= 1
        return ret
```

从评测结果来看，第二种思路的Python解决方案只超过了31%的提交，明显比第一个思路的Python解决方案要差一些，这是一个比较意外的结果。不过，仔细想想也还比较合理。因为Python语言的循环是比较慢的，在Python的循环中放太多东西，包括变量和条件判断，很容易拉低性能。事实上，在这种比较简单的、比较基础的Python实现上，能够减少使用循环才是运行性能优化的一个重要方向。

## 5. Python下的另一种写法

Python提供了一些内置方法可以帮助我们减少对循环的使用，比如这里就可以用到filter()函数，来对矩阵中的负数进行提取和计数。

```Python
class Solution:
    # Runtime: 120 ms, faster than 93.16%
    # Memory Usage: 13.7 MB, less than 100.00%
    def countNegatives(self, grid: List[List[int]]) -> int:
        ret = 0
        for row in grid:
            ret += len(list(filter(lambda x: x < 0, row)))
        return ret
```

由于Python3中的filter()函数返回的是一个可迭代对象，因此需要使用list的构造函数将其转换为列表数据，然后进行计数。从评测结果，这种Python解决方案超过了93%的提交。从数据上看是有一些提升，不过意义并不是很大，因为这种简单问题在多次提交评测得到的结果波动很大。

## 6. 一种n*log(n)的解决思路

由于每一行都是有序的，因此可以使用二分法查找出第一个负数的索引，然后使用一行的元素总数减去这个索引就能计算出一行中负元素的个数。二分法的时间复杂度为log(n)，在正方形的情况下，这种逐行使用二分法的思路的时间复杂度为 n*log(n)。下面给出C++实现的解决方案。

```C++
class Solution {
public:
    // Runtime: 16 ms, faster than 93.46%
    // Memory Usage: 9.4 MB, less than 100.00%
    int countNegatives(vector<vector<int>>& grid) {
        int n = grid[0].size();
        int ret = 0, low = 0, high = n - 1, mid = 0;
        for (auto & row : grid) {
            low = 0;
            high = n - 1;
            // all elements are non-negative
            if (row[high] >= 0) {
                continue;
            } 
            // all elements are negative
            if (row[low] < 0) {
                ret += n;
                continue;
            }
            // binary search
            while (low < high) {
                mid = (low + high) / 2;
                if (row[mid] < 0) {
                    high = mid;
                } else {
                    low = mid+1;
                }
            }
            // cout << low << "," << high<< endl;
            ret += n - low;
        }
        return ret;
    }
};
```

从评测结果来看似乎提升并不大，多次提交测试发现该问题测试数据的波动太大，实测数据置信度不高。不过作为算法思路分析和训练而言没有关系。

从这个问题的解决思路来看，可以有以下几点启发：

1. Python实现中，尽量使用内建的或者第三方模块提供的接口代替手动编写的多重循环

2. 在简单的问题中，先给出直观的解决思路，然后再想办法优化

3. 观察数据特点并善用往往能够设计出复杂度更优的算法
