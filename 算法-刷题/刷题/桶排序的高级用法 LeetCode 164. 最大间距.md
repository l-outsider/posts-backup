在传统桶排序中，桶的大小都是1，这似乎已经形成了思维定式，桶的大小只能为1，也没有想过，桶的大小这个概念。

因此在平时创建桶的时候，都是直接声明一个很大的数组，譬如：
```cpp
int t[1000000];

for (i...) {
	t[i] ++;
}
```

---

今天遇到了这么一道题：[LeetCode 164. 最大间距](https://leetcode-cn.com/problems/maximum-gap/)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201126115219330.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N3YWxsb3dibGFuaw==,size_16,color_FFFFFF,t_70#pic_center)

其中数据的范围是32位整形的范围，而且要求算法的时间复杂度在`O(n)`以内，那么我第一个想到的排序算法就是桶排序，但是开辟一个32位整数能到达的大小的数组明显是不现实的，**我还想到了用hashmap，因为我记得map是用红黑树实现的，插入复杂度是`logN`，但是不用排序，map肯定不行。hashmap是用哈希实现的，可以达到`O(1)`的复杂度，但是还是要排序**，所以最后还是不行。

看了题解以后，**居然还是用的桶排序**，但是完全是新的用法，因此特此记录，进行学习。

其实这里能用桶排序，不只是因为题目对复杂度的限制，**更是因为题目要求的巧妙之处**，这里涉及到一些数学证明，在官方的题解中可以详细学习。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201126115146481.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N3YWxsb3dibGFuaw==,size_16,color_FFFFFF,t_70#pic_center)
这里的桶不再是长度为1的单位桶了，而是一个区间桶，而且屏蔽掉了数组元素可以很大的事实，以区间为桶，维护一个区间的最大值和最小值，进行区间之间的比较，由于产生相邻元素差最大值的**两个元素一定产生在两个不同的区间**，因此用`i`区间的最小值，减去`i-1`区间的最大值，就能找到差的最大值，并且只有这样计算才能保证相邻（因为桶排序虽然把长度从1扩展到了区间上，但是依然是一个排序行为，桶内虽然不排序，但是桶内的最大值一定在区间的最右侧，桶内的最小值一定在区间的最左侧。）

其实这里只要关于这个数学证明看懂了，代码就很好写了。**具体的解释我附在注释里**。

献上答案

```cpp
class Solution {
public:
    int maximumGap(vector<int>& nums) {
        int size = nums.size();
        if (size < 2) return 0;
        / 计算区间的相关信息
        int MAX = *max_element(nums.begin(), nums.end());
        int MIN = *min_element(nums.begin(), nums.end());
        int d = max(1, (MAX - MIN) / (size - 1));
        int d_size = (MAX - MIN) / d + 1;
		
		/ 维护区间
        vector<pair<int, int>> bucket(d_size, {-1, -1});
        for (const auto &i : nums) {
            int idx_bucket = (i - MIN) / d;
            if (bucket[idx_bucket].second == -1) {
                bucket[idx_bucket].second = bucket[idx_bucket].first = i;
            } else {
                bucket[idx_bucket].first = max(bucket[idx_bucket].first, i);
                bucket[idx_bucket].second = min(bucket[idx_bucket].second, i);
            }
        }
        / 这里由于MIN一定在0号区间的最左侧，因此，prev索引可以直接从0开始，遍历从1开始，不需要考虑prev是否存在。
        int ans = 0, prev_bucket = 0;
        for (int i=1;i<d_size;++i) {
            if (bucket[i].first == -1) {
                continue;
            } else {
            	/ 注意，这里是要用下一个区间的最小值减上一个区间的最大值。不要搞反了。
                ans = max(ans, bucket[i].second - bucket[prev_bucket].first);
                prev_bucket = i;
            }
        }
        return ans;
    }
};
```
