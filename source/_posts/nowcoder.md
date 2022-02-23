---
title: 牛客网TOP101
date: 2022-02-23 14:21:25
tags:
categories: 算法
---

算是在LeetCode之外开一个新坑，毕竟LeetCode是写过一次的，而且不是专项练习。牛客网的101题是有分单元的，而且我没有写过。

# **BM1** **反转链表**

```C++
ListNode* ReverseList(ListNode* pHead) {
    ListNode* last = nullptr;
    ListNode* next = pHead->next;
    while(pHead){
        next = pHead->next;
        pHead->next = last;
        last = pHead;
        pHead = next;
    }
    return last;
}
```

