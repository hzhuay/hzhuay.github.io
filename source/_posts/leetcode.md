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

