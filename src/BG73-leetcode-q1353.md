# 面试刷题：可以参加的最大活动数

## 1. 题目 (Leeetcode Q1353)

给定一系列事件，其中events[i] = [startDay_i，endDay_i]。
每个事件我都从 startDay_i 开始，到 endDay_i 结束。
您可以在d的任何一天（startDay_i <= d <= endDay_i）参加活动 i。
请注意，您在任何时间只能参加一个活动。返回您可以参加的最大活动数。

## 2. 约束条件

(a) 1 <= events.length <= 10^5
(b) events[i].length == 2
(c) 1 <= events[i][0] <= events[i][1] <= 10^5

## 示例 1

```
Input: events = [[1,2],[2,3],[3,4]]
Output: 3
```

解释：你能参与这三个事件中的所有事件，其中一种安排方式如下：第一天参加第一个事件，第二天参加第二个事件，第三天参加第三个事件。

## 示例 2

```
Input: events= [[1,2],[2,3],[3,4],[1,2]]
Output: 4
```

## 示例 3

```
Input: events = [[1,4],[4,4],[2,2],[3,4],[1,1]]
Output: 4
```

## 示例 4

```
Input: events = [[1,100000]]
Output: 1
```

## 示例 5

```
Input: events = [[1,1],[1,2],[1,3],[1,4],[1,5],[1,6],[1,7]]
Output: 7
```

## 3. 解决思路

刚阅读完题目的时候似乎有点不知所措，因为这个问题与那种区间重叠问题具有很大的相似性，但解决方法又不能照搬。我们可以考虑一下这个问题：如果两个事件具有相同的开始事件，不同的终止事件，那么我们是该提前安排“早”终止的，还是“晚”终止的呢？答案是，提前安排“早”终止的。因为在时间轴上，“晚”终止的事件还可以安排在“稍后”一点的时间里。如果提前安排了“晚”终止的事件，那么可能造成“早”终止的事件没有合适的时间来安排。事实上，以上分析对于不同起始时间的事件也是有效的。

也就是说，解决这个问题的第一个策略是：在时间轴上，必须先安排终止时间靠前的事件。对于每个事件，在其起始和终止的时间之内，选择尽量早的时间进行安排，才能保证安排最多的事件。因此，可以得到以下算法表述：

```
Input: events
Initialize: ret = 0
-------------------------
Step 1: events = sort_by_end_day(events)
Step 2: event = get_first_element_and_not_back(events)
Step 3: ret += arrange_a_day(event.start_day, event.end_day)
Step 4: goto Step 2 until events set is empty
-------------------------
Output: ret
```

中文表述如下：

```
输入：events
初始化：ret = 0
-----------------------
第1步：按照终止时间的先后对events中的事件进行排序
第2步：从events事件集合中顺序选取第一个事件，不放回
第3步：为当前事件在起始到终止时间范围内安排一个时间点，成功则累积数量
第4步：跳转到第2步，知道events集合为空
-----------------------
输出：ret
```

## 4. C++解决方案

C++解决方案可以考虑使用STL中的sort函数，不过需要提供一个比较函数，用于将事件的终止时间设置为排序的指标。可以使用仿函数解决，也可以直接提供一个函数。由于比较函数内部逻辑简单，这里给出的解决方案使用static函数作为比较函数。在排序完成之后，最后一个时间的终止时间就是时间轴的终止时间。由于需要在时间轴上标注已使用的时间，因此需要创建对应的存储空间。为了减少内存的开辟次数，最好一次将时间轴的内存申请完成。下面直接给出源代码。

```C++
class Solution {
private:
    static bool _comp_func(const vector<int> & a, 
                           const vector<int> & b) {
        return a[1] < b[1];
    }
    
public:
    // Runtime: 228 ms, faster than 92.22%
    // Memory Usage: 44.5 MB, less than 100.00%
    int maxEvents(vector<vector<int>>& events) {
        // sort by increasing order of the end day
        sort(events.begin(), events.end(), this->_comp_func);
        // check all events
        int num_events = events.size(), ret = 0;
        vector<bool> used(events[num_events-1][1] + 1, false);
        for (auto & e : events) {
            // find a free day to arrange this event
            for (int i = e[0]; i<=e[1]; ++i) {
                if (used[i]) {
                    continue; // to check the next day
                }
                used[i] = true;
                ++ret;
                break; // arrange current event completely
            }
        }
        return ret;
    }
};
```

从评测结果来看，C++解决方案超过了92%的提交，是一个不错的方案。

## 5. Python解决方案

已经用C++验证解决思路的有效性，接下来就可以使用Python再实现一个解决方案。Python中的排序函数可以选择sorted，该函数的key参数可以指定为一个函数从而设置事件终止时间作为排序指标。在实现时，key参数设置为lambda表达式会比较简洁一点。另外，Python中没有“++”或者“--”运算符，所以，这里需要使用“ret += 1”代替C++解决方案中的“++ret”。此外，sorted函数进行排序并不是“inplace”，即不是原处修改，所以需要将sorted的返回值重新赋值给events变量。以下直接给出Python解决方案的源代码。

```Python
class Solution:
    # Runtime: 840 ms, faster than 85.92%
    # Memory Usage: 50.4 MB, less than 100.00%
    def maxEvents(self, events: List[List[int]]) -> int:
        # sort by increasing order of the end day
        events = sorted(events, key=lambda x : x[1])
        #print(events)
        # check all events 
        ret = 0
        used = [False for i in range(events[-1][1]+1)]
        for e in events:
            # find a free day to arrange this event
            for i in range(e[0], e[1]+1):
                if used[i]:
                    continue
                used[i] = True
                ret += 1
                # arrange current event completely
                break
        return ret
```

从评测结果来看，Python解决方案超过了85%的提交，也是一个可以接受的性能。

## 6. 时间复杂度

最后，我们来分析一下这个解决思路的时间复杂度。在最坏的情况下，n个事件对应的起始时间和对应的终止时间都是完全重合的，则该解决思路的时间复杂度为 O(n^2)。在最好的情况下，当每个事件的时间段都彼此错开，则时间复杂度为 O(n)。我们通常所说的时间复杂度是指在最坏情况下的，因为这样有利于我们评估算法所需资源的上限。所以，以上的C++和Python解决方案虽然都超过了绝大多数的提交，但是从时间复杂度上来看，并没有取得太好的结果，应该还有进一步的优化空间。
