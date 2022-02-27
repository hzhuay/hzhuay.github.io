---
title: 牛客网TOP101
date: 2022-02-23 14:21:25
tags:
categories: 算法
---

算是在LeetCode之外开一个新坑，毕竟LeetCode是写过一次的，而且不是专项练习。牛客网的101题是有分单元的，而且我没有写过。

# 链表

## BM1 反转链表

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

## BM2 链表内指定区间反转

```C++
ListNode* reverseBetween(ListNode* head, int m, int n) {
    ListNode* dummy = new ListNode(0);
    dummy->next = head;
    ListNode *pre = dummy, *left = head; 
    //获取反转区间的起点和起点前的点pre
    for(int i = 1; i < m; i++){
        pre = left;
        left = left->next;
    }
    // 这个循环相当于把每次遍历到的节点提前至反转区间的头部
    for(int i = 0; i < n - m; i++){
        ListNode *tmp = left->next;
        left->next = tmp->next;
        tmp->next = pre->next;
        pre->next = tmp;			//每次都更新反转区间与链表头部的连接的节点
    }
    return dummy->next;
}
```

![image-20220224144831853](D:\code\hzhuay.github.io\source\img\image-20220224144831853.png)

## BM4 合并两个排序的链表

递推版本

```C++
ListNode* Merge(ListNode* pHead1, ListNode* pHead2) {
    ListNode* cur = new ListNode(0);
    ListNode* dummy = cur;
    while(pHead1 && pHead2){
       if (pHead1->val < pHead2->val) {
            cur->next = pHead1;
            pHead1 = pHead1->next;
        }
        else {
            cur->next = pHead2;
            pHead2 = pHead2->next;
        }
        cur = cur->next;
    }
    cur->next = pHead1 ? pHead1 : pHead2;
    return dummy->next;
}
```

 递归版本

```C++
ListNode* Merge(ListNode* pHead1, ListNode* pHead2) {
    if(!pHead1)return pHead2;
    if(!pHead2)return pHead1;
    if(pHead1->val < pHead2->val){
        pHead1->next = Merge(pHead1->next, pHead2);
        return pHead1;
    }else{
        pHead2->next = Merge(pHead1, pHead2->next);
        return pHead2;
    }
}
```

## BM5 合并k个已排序的链表

只要利用了优先队列这样的数据结构，这题就很容易了，让优先队列自动帮我们维护排序即可。

```C++
struct cmp{
    bool operator()(const ListNode* l, const ListNode* r)
    	{return l->val > r->val;}
};
ListNode *mergeKLists(vector<ListNode *> &lists) {
    priority_queue<ListNode*, vector<ListNode*>, cmp> q;
    for(auto l: lists)
        if(l) q.push(l);
    if(q.empty()) return nullptr;
    ListNode *dummy = new ListNode(0);
    ListNode *cur = dummy;
    while(!q.empty()){
        cur->next = q.top();
        q.pop();
        cur = cur->next;
        if(cur->next) q.push(cur->next);
    }
    return dummy->next;
}
```



## BM6 判断链表中是否有环

快慢指针。

```C++
bool hasCycle(ListNode *head) {
    ListNode *fast = head, *slow = head;
    while(fast && fast->next){
        fast = fast->next->next;
        slow = slow->next;
        if(fast == slow)return true;
    }
    return false;
}
```

## BM7 链表中环的入口结点



```C++
ListNode* EntryNodeOfLoop(ListNode* pHead) {
    // 先检查有没有环
    ListNode *fast = pHead, *slow = pHead;
    while(fast && fast->next){
        fast = fast->next->next;
        slow = slow->next;
        if(fast == slow)break;
    }
    if(fast && fast->next){
        // 确定有环，再来寻找环的入口
        fast = pHead;
        while(fast != slow){
            fast = fast->next;
            slow = slow->next;
        }
        return fast;
    }else return nullptr;
}
```



## BM8 链表中倒数最后k个结点

第一种思路比较符合直觉：先遍历一遍获得链表长度，然后再遍历一次找到截取的节点。

```C++
ListNode* FindKthToTail(ListNode* pHead, int k) {
    int len = 0;
    ListNode* cur = pHead;
    while(cur){
        cur = cur->next;
        len++;
    }
    if(k > len)return nullptr;
    cur = pHead;
    int n = len - k;
    while(n--)cur = cur->next;
    return cur;
}
```

另一种写法是快慢指针，其实性能应该是差不多的，但是看起来高端一点。

```C++
ListNode* FindKthToTail(ListNode* pHead, int k) {
    ListNode *fast = pHead, *slow = pHead;
    while(k--){
        if(!fast)return nullptr;
        else fast = fast->next;
    }
    while(fast){
        fast = fast->next;
        slow = slow->next;
    }
    return slow;
}
```

## BM9 删除链表的倒数第n个节点

```C++
ListNode* removeNthFromEnd(ListNode* head, int n) {
    ListNode *fast = head, *slow = head;
    for(int i = 0; i < n; i++)
        fast = fast->next;
    //如果此时快指针已经到达尾后，说要删除的就是链表头
    if(!fast) return head->next;
    while(fast->next){//直到fast到达结尾
        fast = fast->next;
        slow = slow->next;
    }
    //此时慢指针停在该被删除的节点之前，修改其next跳过被删除的节点
    slow->next = slow->next->next;
    return head;
}
```

## BM10 两个链表的第一个公共结点

既然保证输入数据肯定是正确的，那么只要不断遍历，两个节点一定会相遇。

```C++
ListNode* FindFirstCommonNode( ListNode* pHead1, ListNode* pHead2) {
    if(!pHead1 || !pHead2) return nullptr;
    ListNode *p1 = pHead1, *p2 = pHead2;
    while(p1 != p2){
        p1 = p1 ? p1->next : pHead1;
        p2 = p2 ? p2->next : pHead2;
    }
    return p1;
}
```

## BM11 两个链表生成相加链表

结合力扣第2题和牛客第1题，先将两个链表反转，相加后再将结果反转后返回。

```C++
ListNode* addInList(ListNode* head1, ListNode* head2) {
        return ReverseList(addTwoNumbers(ReverseList(head1), ReverseList(head2)));
}
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

## BM12 单链表的排序

要求：空间复杂度 O*(*n)，时间复杂度 O(nlogn)，那我为何不直接塞到数组里，然后对数组排序，再重建列表呢？

```C++
static bool cmp(const ListNode* a, const ListNode* b){
    return a->val < b->val;
}
ListNode* sortInList(ListNode* head) {
    vector<ListNode*> arr;
    ListNode* tmp = head;
    while(tmp){
        arr.push_back(tmp);
        tmp = tmp->next;
    }
    sort(arr.begin(), arr.end(), cmp);
    for(int i = 0; i < arr.size() - 1; i++){
        arr[i]->next = arr[i+1];
    }
    arr[arr.size()-1]->next = nullptr;
    return arr[0];
}
```

## BM13 判断一个链表是否为回文结构

这里也复用了BM1的代码。用快慢指针找到链表中间的部分，然后反转后半链表。

将数据都读到数组里也是一种做法，这题并没有限制空间，但是反转链表的方式可以节省内存，但是会改变链表本身。

```C++
bool isPail(ListNode* head) {
    ListNode *fast = head, *slow = head;
    while(fast && fast->next){
        fast = fast->next->next;
        slow = slow->next;
    }
    //从slow开始讲后半部分的链表反转
    ListNode *l1 = ReverseList(slow), *l2 = head;
    while(l1){
        if(l1->val != l2->val) return false;
        l1 = l1->next;
        l2 = l2->next;
    }
    return true;
}
```

## BM14 链表的奇偶重排

```C++
ListNode* oddEvenList(ListNode* head) {
    if(!head || !head->next) return head;
    ListNode *even_head = head->next, *odd = head, *even = head->next;
    //如果是奇数个节点，因为even本身起点就比head多1，所以即使只剩一个节点
    while(even && even->next){
        odd->next = odd->next->next;
        odd = odd->next;
        even->next = even->next->next;
        even = even->next;
    }
    odd->next = even_head;
    return head;
}
```



## BM15 删除有序链表中重复的元素-I

每次遍历到该节点和下一个节点一样的，就让该节点跳过下一个节点。

```C++
ListNode* deleteDuplicates(ListNode* head) {
    ListNode *cur = head;
    while(cur){
        while(cur->next && cur->val == cur->next->val)
            cur->next = cur->next->next;
        cur = cur->next;
    }
    return head;
}
```

## BM16 删除有序链表中重复的元素-II

```C++
ListNode* deleteDuplicates(ListNode* head) {
    ListNode *dummy = new ListNode(-1);
    dummy->next = head;
    ListNode *cur = head, *pre = dummy;
    while(cur){
        if(cur->next && cur->val == cur->next->val){
            //跳过除了自己之外的所有重复节点
            while(cur->next && cur->val == cur->next->val){
                cur->next = cur->next->next;
            }
            pre->next = cur->next;	//pre->next跳过重复节点cur
        }else{
            pre = cur;
        }
        cur = cur->next;
    }
    return dummy->next;
}
```

# 二分查找/排序

## BM17 二分查找-I

二分查找的思路本身没什么好说的，但是细节部分很值得深究，比如每次重新赋值`left`和`right`要不要对`mid`加一或者减一；循环判断条件用`<=`还是`<`，这些会影响到查找的效果。我会在[这篇笔记](/笔记/binary-search/)中单独讲这个问题。

这里先提供一个最基本的二分查找。

```C++
int search(vector<int>& nums, int target) {
    int l = 0, r = nums.size() - 1;
    while(l <= r){
        int mid = (r + l) / 2;
        if(nums[mid] < target)
            l = mid + 1;
        else if (nums[mid] > target)
            r = mid - 1;
        else return mid;
    }
    return -1;
}
```

## BM18 二维数组中的查找

从矩阵的右上角开始查找，如果偏大了就向左走，偏小了就向右走。

```C++
bool Find(int target, vector<vector<int> > array) {
    if(array.empty())return false;
    int R = array.size(), C = array[0].size();
    int r = 0, c = C - 1;//起点是右上角
    while(r < R && c >= 0){//坐标还在合理范围内
        if(array[r][c] < target) r++;
        else if(array[r][c] > target) c--;
        else return true;
    }
    return false;
}
```

## BM19 寻找峰值

```C++

```



## 

```C++
d
```

## 

```C++
d
```

## 

```C++
d
```

## 

```C++
d
```

## 

```C++
d
```

## 

```C++
d
```

