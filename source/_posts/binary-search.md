---
title: 二分查找的细节问题
date: 2022-02-26 12:50:33
tags: 算法
categories: 笔记
---

二分查找的思路其实很好理解，但是细节很坑。到底要给 `mid` 加一还是减一，`while` 到底用`<=`还是`<`，这些都会对二分查找的效果有细微的影响。

二分查找总体上有几个应用：

1. 寻找一个数
2. 寻找一个数第一次出现的位置（左边界）
3. 寻找一个数最后一次出现的位置（右边界）

# 基础版本：寻找一个数

```C++
int binarySearch(vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;//细节点
    while(left <= right) {//细节点
        int mid = left + (right - left) / 2;
        if(nums[mid] == target)
            return mid; 
        else if (nums[mid] < target)
            left = mid + 1;//细节点
        else if (nums[mid] > target)
            right = mid - 1;//细节点
    }
    return -1;
}
```

这是最基础的版本：寻找一个数，如果存在则返回下标；不存在则返回-1。

## while应该是`<=`还是`<`

将`right`初始化为`nums.size()-1`，说明`right`的含义是数组的最后一个元素的索引，而不是数组长度或者尾后元素。**我们先按照`nums.size()-1`来讨论**。

- 如果使用`<=`，那么说明每次搜索的区间是`[left, right]`，注意是左闭右闭区间。
- 如果使用`<`，那么说明每次搜索的区间是`[left, right)`，注意是左闭右开区间。

终止搜索的条件为：

- 找到目标值：`if(nums[mid] == target)`
- 搜索区间为空
  - 对于`left <= right`：终止条件是`left == right + 1`，用区间表达为`[right + 1, right]`。这个区间显然是为空的。
  - 对于`left < right`：终止条件是`left == right`，用区间表达为`[right, right]`。这个区间其实是不为空的`left`或者`right`本身是合法。如果退出了循环，相当于该索引**没有被检查**。

那么结论很明显了，如果定义了`right = nums.size() - 1`，那么应该搭配`while(left <= right)`。

如果非要用`while(left <= right)`，那么就应该把漏掉了`left == right`情况给**特判**。注意，由于需要访问`nums[left]`所以还要加入对`nums`为空的特判。但是额外两处特判多少让人觉得有人不自然。

```pseudocode
if(nums.empty())return -1;
while(left < right) {
    // ...
}
return nums[left] == target ? left : -1;
```

## 左闭右开的版本

如果定义`right = nums.size()`，那么搜索的起始区间是`[0, nums.size)`，与之搭配的应该是`while(left < right)`，正如上面所说。

需要注意的点是`if(nums[mid] > target) right = mid;`。因为`right`是闭区间，本身就是无法访问的，如果让`right = mid - 1`，相当于不仅判断`mid`不满足，也认为`mid - 1`不满足，相当于多跳过了一个点。

代码如下，和最初版差不多简洁。很合理。

```C++
int left = 0, right = nums.size();//细节点
while(left < right) {//细节点
    int mid = left + (right - left) / 2;
    if(nums[mid] == target)
        return mid; 
    else if (nums[mid] < target)
        left = mid + 1;//细节点
    else if (nums[mid] > target)
		right = mid;//细节点
}
return -1;
```

# 寻找一个数第一次出现的位置（左边界）

```c++
int left_bound(vector<int>& nums, int target) {
    if (nums.empty()) return -1;
    int left = 0, right = nums.size();//细节点
    while(left < right) {//细节点
        int mid = (left + right) / 2;
        if (nums[mid] == target)
            right = mid;//这样才能用上二分的性质
        else if (nums[mid] < target)
            left = mid + 1;
        else if (nums[mid] > target)
            right = mid;//细节点
    }
    // 如果没有找到
    if(left >= nums.size() || nums[left] != target) return -1;
    return left;
}
```

这里先展示左闭右开的写法，`right = nums.size()`，` while(left < right)`，`right = mid;`

注意，最后返回`left`或`right`是一样的，因为终止条件是`left == right`。

# 寻找一个数最后一次出现的位置（右边界）

```C++
int right_bound(vector<int>& nums, int target) {
    if (nums.length == 0) return -1;
    int left = 0, right = nums.size();
    while (left < right) {
        int mid = (left + right) / 2;
        if (nums[mid] == target)
            left = mid + 1;
        else if (nums[mid] < target)
            left = mid + 1;
        else if (nums[mid] > target)
            right = mid;
    }
    if(left == 0 || nums[left-1] != target) return -1;
    return left - 1;
}
```

与左边界类似的，寻找右边界时，即使找到target了也要`left = mid + 1;`。由于`mid = left - 1`，因此最后返回的是`left - 1 `，除非寻找的是数组中第一个比target大的数。



# 最终记忆版本（板子）

统一采用左闭右闭的形式，即`right = nums.size() - 1`，` while(left <= right)`，`right = mid - 1;`,`left = mid + 1;`

```C++
int binary_search(vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1; 
    while(left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        else if (nums[mid] > target)
            right = mid - 1; 
        else if(nums[mid] == target)
            return mid;
    }
    return -1;
}

int left_bound(vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        else if (nums[mid] > target)
            right = mid - 1;
        else if (nums[mid] == target)
            right = mid - 1;
    }
    if (left >= nums.length || nums[left] != target) return -1;
    return left;
}

int right_bound(vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target)
            left = mid + 1;
        else if (nums[mid] > target)
            right = mid - 1;
        else if (nums[mid] == target)
            left = mid + 1;
    }
    //right == left - 1
    if (right < 0 || nums[right] != target) return -1;
    return right;
}
```

