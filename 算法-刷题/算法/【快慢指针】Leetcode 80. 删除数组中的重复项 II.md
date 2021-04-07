# part 1 题目介绍
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210407151606300.png)
注意读题，要注意两点：

1. 数组是有序的，根据示例可以看出是升序。
2. 一般提到原地算法就是，利用数组内的空间进行操作，比如这道题应该是要使用数组后面的数覆盖前面的数以达到**原地**目的。

---
# part 2 思路

使用快慢指针，慢指针指向应该被覆盖的位置，快指针指向最新的位置。

这里需要判断一个元素已经出现了几次。需要清晰知道fast、slow指针的含义，说实话这道题虽然代码简单，当时这里真的容易搞晕。

`fast指针`：`fast指针`指向待处理的元素，这里的待处理有两种情况：

1. 需要向前`slow指针`处拷贝
2. 由于重复，不需要拷贝，下次直接`fast ++`

`slow指针`：`slow指针`指向待拷贝的位置，`slow指针`之前的元素表示已经满足要求的元素。

所以就要判断当前处于哪种情况：

1. `fast指针`指向的元素向`slow`处拷贝
2. 当前已处理的部分的最新的两个元素已经达到上限，不能再拷贝了，`fast ++`

 **由于我们要把`fast元素` 拷贝到 `slow`处，并且`slow`之前的元素都是满足要求的（不重复三次且递增），那么只需要判断`fast元素`和`slow-2元素` 是否相同，如果相同则拷贝过来必定形成三个一样的元素。**

所以只要`nums[slow -  2] === nums[fast]` 就不能拷贝，直接`fast ++`。
如果`nums[slow -  2] !== nums[fast]`，就直接`nums[slow] = nums[fast]; fast ++; slow ++;`

---
# part 3 代码

```cpp
int removeDuplicates(vector<int>& nums) {
	int fast, slow;
	fast = slow = 2 <= nums.size() ? 2 : nums.size();
	while (fast < nums.size()) {
		if (nums[slow - 2] != nums[fast]) {
			nums[slow] = nums[fast];
			slow ++;
		}
		fast ++;
	}
	return slow;
}
```