---
title: LeetCode算法复健
date: 2022-02-20 18:36:15
tags: 算法
categories: 算法
---

临近求职，面试手撕算法避无可避，不得不将搁置很久的算法题重新捡起来练习。这里算开个新坑，将LeetCode前几页的热门题目重新写一遍，最好能记录每次的用时。

# [1. 两数之和](https://leetcode-cn.com/problems/two-sum/)

进阶要求是时间复杂度小于 O(n^2^)，那么应该需要借助排序和二分查找，但是直接排序会打乱下标，因此借助map结构（unordered_map基于哈希表则更快）。

坑点：遍历的时候一定要先查找`nums[i]`对应的搭档在不在，然后再将其加入到map中，不然会出现自己和自己相加的情况。

```C++

vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map<int, int>m;
    vector<int> tmp;
    for(int i = 0; i < nums.size(); i++){
        if(m.find(target - nums[i]) != m.end()){
            tmp.push_back(i);
            tmp.push_back(m[target - nums[i]]);
            break;
        }
        m[nums[i]] = i;
    }
    return tmp;
}
```



# [2. 两数相加](https://leetcode-cn.com/problems/add-two-numbers/)

递推的写法比较直白，分成三段：两个链表合并、合并剩余的较长链表、把最后剩余的进位加上。这三段可以简化到一个while循环中。

```c++
ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
    ListNode* head = new ListNode(0);
    ListNode* cur = head;
    int tmp = 0;
    while(l1 || l2 || tmp){
        int sum = tmp;
        sum += l1 ? l1->val : 0;
        sum += l2 ? l2->val : 0;
        tmp = sum / 10;
        sum %= 10;
        cur->next = new ListNode(sum);
        cur = cur->next;
        if(l1)l1 = l1->next;
        if(l2)l2 = l2->next;
    }
    return head->next;
}
```

# [3. 无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

用map的数据结构可以很容易地统计词频，用上了之后这就是一道很简单的滑动窗口问题。

遍历时每次都强制把刚遇到的字符`s[i]`加入到字串中，然后如果该字符出现了2次，则持续右移`l`，直到遇到一个和`s[i]`相同的字符，将`s[i]`的频率减到1。

```C++
int lengthOfLongestSubstring(string s) {
    unordered_map<char, int> cnt;
    int l = 0, len = 0, ans = 0;
    for(int i = 0; i < s.size(); i++){
        cnt[s[i]]++;
        len++;
        while(cnt[s[i]] > 1){
            cnt[s[l++]]--;
            len--;
        }
        ans = max(ans, len);
    }
    return ans;
}
```

# [4. 寻找两个正序数组的中位数](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/)

比较难的二分查找，边界处理很繁琐。数组元素数量是固定的，可以把两个有序的数组看做一个整体，由一个折现分别将X和Y分为两部分，左半部分和右半部分数量相等（或者左半边多一个，多出的那个即为中位数），规定左半边的所有数小于右半边的所有数。我们定义i分隔了X，j分隔了Y，由于左半边数量等于右半边，则只要i确定那么j也确定，因此对i进行二分查找。

如代码中注释所画，比较`X[i-1]`和`Y[j]`的大小关系即可，并不在乎`X[i-1]`和`Y[j-1]`的大小关系，只要i和j数量关系维持住即可，因为`Y[j-1]`一定比`Y[j]`小。由于X长度小于Y，所以只移动i是安全的。

```c++
double findMedianSortedArrays(vector<int>& X, vector<int>& Y) {
    if(X.size() > Y.size())swap(X, Y);  //保证X长度小于Y
    int sizeX = X.size(), sizeY = Y.size();
    int leftSize = (sizeX + sizeY + 1) / 2; //中位数一定在左边
    int l = 0, r = sizeX, i, j;
    while(l < r) {
        /*  X[i-1] | X[i]
        *           --
        *      Y[j-1] | Y[j]
        */
        i = (l + r + 1) / 2, j = leftSize - i;
        if(X[i-1] > Y[j]) r = i - 1;
        else l = i;
    }
    i = l, j = leftSize - i;
    //如果i和j取到了边界，那么说明这两个数组没有交叉，要用int的极值排除边界的干扰
    int XLeftMax = i == 0 ? INT_MIN : X[i-1];
    int XRightMin = i == sizeX ? INT_MAX : X[i];
    int YLeftMax = j == 0 ? INT_MIN : Y[j-1];
    int YRightMin = j == sizeY ? INT_MAX : Y[j];
    if((sizeX + sizeY) % 2 == 1){
        return max(XLeftMax, YLeftMax); //如果是奇数个，那么中位数只有一个，一定是这两者中较大的
    }else{
        // 如果是偶数个，那么中位数由两个数算出：左区间最大值和右区间最小值
        return double((max(XLeftMax, YLeftMax) + min(XRightMin, YRightMin))) / 2;
    }
}

```

