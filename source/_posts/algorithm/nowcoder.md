---
title: 牛客网TOP101
date: 2022-02-23 14:21:25
tags:
categories: 算法
---

算是在LeetCode之外开一个新坑，毕竟LeetCode是写过一次的，而且不是专项练习。牛客网的101题是有分单元的，而且我没有写过。

## 链表

### BM1 反转链表

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

### BM2 链表内指定区间反转

```C++
ListNode* reverseBetween(ListNode* head, int m, int n) {
    ListNode* dummy = new ListNode(0);
    dummy->next = head;
    ListNode *pre = dummy, *cur = head; 
    //获取反转区间的起点和起点前的点pre
    for(int i = 1; i < m; i++){
        pre = cur;
        cur = cur->next;
    }
    // 这个循环相当于把每次遍历到的节点提前至反转区间的头部
    for(int i = 0; i < n - m; i++){
        ListNode *tmp = cur->next;
        cur->next = tmp->next;
        tmp->next = pre->next;
        pre->next = tmp;			//每次都更新反转区间与链表头部的连接的节点
    }
    return dummy->next;
}
```

相当于每次都将`cur->next`移动到`pre->next`，然后`cur`前进一格。

[<img src="https://s4.ax1x.com/2022/03/02/b8JUI0.png" alt="b8JUI0.png" style="zoom:50%;" />](https://imgtu.com/i/b8JUI0)

[<img src="https://s4.ax1x.com/2022/03/02/b8D73n.png" alt="b8D73n.png" style="zoom:50%;" />](https://imgtu.com/i/b8D73n)

### BM3 链表中的节点每k个一组翻转

和上一题一样的思路，采用递归写法，先判断节点数量是否需要反转。如果需要，那么执行`k-1`次，每次都是把`cur->next`移动到`pre->next`，然后`cur`前进一格。执行完后`cur`正好处在反转区间的尾部，`cur->next`就是下一个反转区间的开头。

```C++
ListNode* reverseKGroup(ListNode* head, int k) {
    if(!head || k <= 1) return head;
    ListNode *pre = new ListNode(-1), *cur = head, *next = nullptr;
    pre->next = head;
    for(int i = 0; i < k; i++){
        if(!cur) return head;	//节点数不到k个
        cur = cur->next;		//寻找反转区间的右边界，开区间
    }
    cur = head;
    for(int i = 1; i < k; i++){	//只需要执行k-1次
        next = cur->next;
        cur->next = next->next;
        next->next = pre->next;
        pre->next = next;
    }
    cur->next = reverseKGroup(cur->next, k);
    return pre->next;
}
```

### BM4 合并两个排序的链表

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

### BM5 合并k个已排序的链表

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

### BM6 判断链表中是否有环

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

### BM7 链表中环的入口结点

快慢指针。

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
        fast = pHead;	//这一步很关键，有数学证明这样可以使快慢指针在入口相遇
        while(fast != slow){
            fast = fast->next;
            slow = slow->next;
        }
        return fast;
    }else return nullptr;
}
```

### BM8 链表中倒数最后k个结点

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

### BM9 删除链表的倒数第n个节点

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

### BM10 两个链表的第一个公共结点

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

### BM11 两个链表生成相加链表

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

### BM12 单链表的排序

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

### BM13 判断一个链表是否为回文结构

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

### BM14 链表的奇偶重排

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

### BM15 删除有序链表中重复的元素-I

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

### BM16 删除有序链表中重复的元素-II

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

## 二分查找/排序

### BM17 二分查找-I

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

### BM18 二维数组中的查找

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

### BM19 寻找峰值

二分法，判断当前点`mid`是上坡还是下坡，向上坡方向靠拢。

```C++
int findPeakElement(vector<int>& nums) {
    int left = 0, right = nums.size() - 1;//由于right=mid，所以right是有可能被返回的
    while(left < right){//退出条件left==right
        int mid = (right + left) >> 1;
        if(nums[mid] < nums[mid + 1])//这是上坡，右侧可能有峰值
            left = mid + 1;//mid+1是因为mid不可能是峰值
        else right = mid;//mid可能是峰值
    }
    return left;
}
```

### BM20 数组中的逆序对

在归并排序中可以顺便统计逆序对。在合并阶段，左区间的所有节点应该小于右区间的所有节点，所以在比较左右区间中的点时，只要遇到了右区间的元素比左区间小的情况，那么该右区间元素与左区间的所有元素都形成了逆序对。

```C++
int M = 1000000007;
int InversePairs(vector<int> data) {
    int ans = 0;
    vector<int> tmp(data.size());
    mergeSort(data, tmp, 0, data.size() - 1, ans);
    return ans;
}

void mergeSort(vector<int> &data, vector<int> &tmp, int l, int r, int &ans){
    if(l >= r) return;
    int mid = (r + l) / 2;
    mergeSort(data, tmp, l, mid, ans);
    mergeSort(data, tmp, mid +1 , r, ans);
    merge(data, tmp, l, mid, r, ans);
}

void merge(vector<int> &data, vector<int> &tmp, int l, int mid, int r, int &ans){
    int i = l, j = mid + 1, k = 0;
    while(i <= mid && j <= r){
        if(data[i] > data[j]){//右区间的元素小于左区间，会产生逆序对
            tmp[k++] = data[j++];
            //data[i, mid]中所有元素都与data[j]产生逆序对
            ans = (ans + (mid - i + 1)) % M;
        }else tmp[k++] = data[i++];
    }
    while(i <= mid)
        tmp[k++] = data[i++];
    while(j <= r)
        tmp[k++] = data[j++];
    for(k = 0; l <= r; l++, k++)
        data[l] = tmp[k];
}
```

### BM21 旋转数组的最小数字

二分查找，判断`arr[mid]`和`arr[left]`的关系。但是这道题要小心`[1,0,1,1,1]`这样的数据，因为`mid`第一次迭代时就是中间的1，如果只是在`arr[mid] == arr[left]`的情况下`left++`的话，mid将永远访问不到0所在的位置。因此需要加上`if(rotateArray[left] < rotateArray[right])`的特判。

```C++
int minNumberInRotateArray(vector<int> rotateArray) {
    int left = 0, right = rotateArray.size() - 1;
    	while(left < right){	//最后剩下的元素一定是答案
        if(rotateArray[left] < rotateArray[right])
            return rotateArray[left];
        int mid = (left + right) / 2;
        if(rotateArray[mid] > rotateArray[left])
            left = mid + 1;	//mid不可能为答案
        else if (rotateArray[mid] < rotateArray[left])
            right = mid;	//mid可能为答案
        else left++;		//无法判断，保守策略
    }
    return rotateArray[left];
}
```

### BM22 比较版本号

双指针，每次提取一个数字来比较。如果遇到不相等就提前结束。

```C++
int compare(string version1, string version2) {
    int i = 0, j = 0, len1 = version1.size(), len2 = version2.size();
    int num1, num2;
    while(i < len1 || j < len2){
        num1 = consume(i, version1);
        num2 = consume(j, version2);
        if(num1 > num2)return 1;
        else if(num1 < num2)return -1;
    }
    return 0;
}

int consume(int& p, string& s){
    int ret = 0;
    while(p < s.size() && s[p] != '.')
        ret = ret * 10 + s[p++] - '0';
    p++;
    return ret;
}
```

## 二叉树

### BM23 二叉树的前序遍历

```C++
vector<int> preorderTraversal(TreeNode* root) {
    vector<int> ans;
    dfs(root, ans);
    return ans;
}
void dfs(TreeNode* root, vector<int> &ans){
    if(!root) return;
    ans.emplace_back(root->val);
    dfs(root->left, ans);
    dfs(root->right, ans);
}
```

### BM24 二叉树的中序遍历

和上题差不多，只需要改`dfs`部分

```C++
void dfs(TreeNode* root, vector<int> &ans){
    if(!root) return;
    dfs(root->left, ans);
    ans.emplace_back(root->val);
    dfs(root->right, ans);
}
```

### BM25 二叉树的后序遍历

```C++
void dfs(TreeNode* root, vector<int> &ans){
    if(!root) return;
    dfs(root->left, ans);
    dfs(root->right, ans);
    ans.emplace_back(root->val);
}
```

### BM26 求二叉树的层序遍历

可能bfs会直观一点，不过dfs也不难写。

```C++
vector<vector<int> > levelOrder(TreeNode* root) {
    vector<vector<int> > ans;
    dfs(root, ans, 0);
    return ans;
}
void dfs(TreeNode* root, vector<vector<int> > &ans, int depth){
    if(!root) return;
    if(ans.size() <= depth)
        ans.push_back(vector<int>());
    ans[depth].emplace_back(root->val);
    dfs(root->left, ans, depth + 1);
    dfs(root->right, ans, depth + 1);
}
```

### BM27 按之字形顺序打印二叉树

理论上可以借用上题的代码获取数据，然后把偶数行给`reverse`，但是这样效率不高，不如在递归的时候就把数据按顺序排好。

```C++
vector<vector<int> > Print(TreeNode* pRoot) {
    vector<vector<int> > ans;
    dfs(pRoot, ans, 0);
    return ans;
}
void dfs(TreeNode* root, vector<vector<int> > &ans, int depth){
    if(!root) return;
    if(ans.size() <= depth)
        ans.push_back(vector<int>());
    ans[depth].emplace_back(root->val);
    if(depth % 2 == 0){
        dfs(root->right, ans, depth + 1);
        dfs(root->left, ans, depth + 1);
    }else {
        dfs(root->left, ans, depth + 1);
        dfs(root->right, ans, depth + 1);
    }
}
```

### BM28 二叉树的最大深度

```C++
int maxDepth(TreeNode* root) {
    if(!root)return 0;
    return max(maxDepth(root->left), maxDepth(root->right)) + 1;
}
```

### BM29 二叉树中和为某一值的路径(一)

注意空树直接返回`false`的情况。

```C++
bool hasPathSum(TreeNode* root, int sum) {
    if(!root) return false;
    sum -= root->val;
    if(!root->left && !root->right && sum == 0) return true;
    return hasPathSum(root->left, sum) || hasPathSum(root->right, sum);
}
```

### BM30 二叉搜索树与双向链表

先中序遍历，把排好序的数都保存到数组上，再遍历一遍创建列表，这是一种直观的思路。递归的写法其实在时间复杂度和空间复杂度上没有明显优势。

```C++
TreeNode *dummy, *pre;
TreeNode* Convert(TreeNode* pRootOfTree) {
    if(!pRootOfTree) return nullptr;
    dummy = new TreeNode(-1);
    pre = dummy;
    dfs(pRootOfTree);
    dummy->right->left = nullptr;
    return dummy->right;
}

void dfs(TreeNode* node){
    if(!node) return;
    dfs(node->left);
    if(!dummy) dummy->right = node;
    pre->right = node;
    node->left = pre;
    pre = node;
    dfs(node->right);
}
```

### BM31 对称的二叉树

题解中很多写法声称每个节点只会访问一次，其实是不对的，这样写才能保证一次。

```C++
    bool isSymmetrical(TreeNode* pRoot) {
        if(!pRoot) return true;
        if(!pRoot->left && !pRoot->right) return true;
        if(!pRoot->left || !pRoot->right) return false;
        if(pRoot->left->val != pRoot->right->val ) return false;
        return isSame(pRoot->left, pRoot->right);
    }
    bool isSame(TreeNode* l, TreeNode* r){
        if(!l && !r) return true;
        if(!l || !r) return false;
        if(l->val != r->val) return false;
        return isSame(l->left, r->right) && isSame(l->right, r->left);
    }
```

### BM32 合并二叉树

在确保`t1`和`t2`非空的情况下，把`t2`合并到`t1`上。

```C++
TreeNode* mergeTrees(TreeNode* t1, TreeNode* t2) {
    if(!t1 && !t2) return nullptr;
    if(!t1 || !t2) return t1 ? t1 : t2;
    //到这里说明t1和t2都存在
    t1->val += t2->val;
    t1->left = mergeTrees(t1->left, t2->left);
    t1->right = mergeTrees(t1->right, t2->right);
    return t1;
}
```

### BM33 二叉树的镜像

关键在于使用**后序遍历**，从底层开始逐层向上翻转。

```C++
TreeNode* Mirror(TreeNode* pRoot) {
    if(!pRoot) return nullptr;
    Mirror(pRoot->left);
    Mirror(pRoot->right);
    swap(pRoot->left, pRoot->right);
    return pRoot;
}
```

### BM34 判断是不是二叉搜索树

中序遍历下二叉搜索树应该是有序的，因此使用`pre`记录前一个节点的值，每次遍历到的节点值都应该大于`pre`。

```C++
long long pre;
bool ans;
bool isValidBST(TreeNode* root) {
    pre = LONG_LONG_MIN;
    ans = true;
    dfs(root);
    return ans;
}

void dfs(TreeNode* node){
    if(!node || !ans) return;
    dfs(node->left);
    if(pre < node->val){
        pre = node->val;
    }else{
        ans = false;
        return;
    }
    dfs(node->right);
}
```

### BM35 判断是不是完全二叉树

用BFS逐层遍历，用`flag`变量记住是否遇到过空节点。如果已经遇到过空节点后，再遇到正常节点，那么就是不完全二叉树。

```C++
bool isCompleteTree(TreeNode* root) {
    queue<TreeNode*> q;
    bool flag = false;
    q.push(root);
    while(!q.empty()){
        TreeNode *node = q.front();
        q.pop();
        if(!node){
            flag = true;
        }else{
            if(flag) return false;//flag为true说明在此节点前已经遇到空节点了
            q.push(node->left);
            q.push(node->right);
        }
    }
    return true;
}
```

### BM36 判断是不是平衡二叉树

如果写一个计算高度的函数，然后从上往下遍历，那么会在计算高度的时候重复计算。可以在计算高度时从下往上判断，如果不合法则返回-1，合法则返回高度。遇到-1之后就不需要继续计算了。

```C++
bool IsBalanced_Solution(TreeNode* pRoot) {
    return getDepth(pRoot) != -1;
}
int getDepth(TreeNode *node){
    if(!node) return 0;
    int left = getDepth(node->left);
    if(left == -1) return -1;
    int right = getDepth(node->right);
    if(right == -1) return -1;
    //如果不合法则返回-1，合法则返回高度
    return abs(left - right) > 1 ? -1 : 1 + max(left, right);
}
```

### BM37 二叉搜索树的最近公共祖先

根节点一定是祖先，因此从根往下找。如果`root`比p和q都大，那么说明p和q都在root的左子树上，反之则在右子树上。当p和q分别位于`root`的左右两边时，说明`root`是最近公共祖先。

```C++
int lowestCommonAncestor(TreeNode* root, int p, int q) {
    while(root){
        if(root->val > p && root->val > q)
            root = root->left;
        else if(root->val < p && root->val < q)
            root = root->right;
        else return root->val;
    }
    return -1;
}
```

### BM38 在二叉树中找到两个节点的最近公共祖先（重要）

1. 如果该节点不是O1也不是O2，那么O1与O2必然分别在该节点的左子树和右子树中

2. 如果该节点就是O1或者O2，那么另一个节点在它的左子树或右子树中



```C++
int lowestCommonAncestor(TreeNode* root, int o1, int o2) {
    return dfs(root, o1, o2)->val;
}

TreeNode* dfs(TreeNode* root, int o1, int o2){
    if(!root || root->val == o1 || root->val == o2)
        return root;
    TreeNode* left = dfs(root->left, o1, o2);
    TreeNode* right = dfs(root->right, o1, o2);
    //如果不在左子树，就肯定在右子树；反正依然
    if(!left) return right;
    if(!right) return left;
    //如果左右子树都不为空，那么说明该点就是最近公共祖先
    return root;
}
```

### BM39 序列化二叉树

采用先序遍历的方式序列化和反序列化。序列化时遇到空节点则设置为`#`，反序列化时遇到空节点则设置为不可能的值。采用队列来保存节点值。

```C++
char* Serialize(TreeNode *root) {    
    string ans = "";
    dfs(root, ans); 
    char* ret = new char[ans.size()+1];
    strcpy(ret, ans.c_str());
    return ret;
}

TreeNode* Deserialize(char *str) {
    queue<int> vals;
    int tmp = 0;
    for(int i = 0; i < strlen(str); i++){
        if(str[i] == ','){
            vals.push(tmp);
            tmp = 0;
        }else if(str[i] == '#'){
            tmp = -1;
        }else{
            tmp = tmp * 10 + str[i] - '0';
        }
    }
    index = 0;
    return build(vals);
}

void dfs(TreeNode* node, string& ans){
    if(!node){
        ans += "#,";
        return;
    }
    ans += to_string(node->val) + ",";
    dfs(node->left, ans);
    dfs(node->right, ans);
}

TreeNode* build(queue<int>& vals){
    if(vals.front() == -1){
        vals.pop();
        return nullptr;
    }
    TreeNode* node = new TreeNode(vals.front());
    vals.pop();
    node->left = build(vals);
    node->right = build(vals);
    return node;
}
```

### BM40 重建二叉树

```C++
TreeNode* reConstructBinaryTree(vector<int> pre,vector<int> vin) {
    if(pre.empty()) return nullptr;
    TreeNode *root = new TreeNode(pre[0]);
    if(pre.size() == 1) return root;
    auto mid = find(vin.begin(), vin.end(), root->val);
    int left = mid - vin.begin(), right = vin.end() - mid;
    root->left = reConstructBinaryTree(vector<int>(pre.begin()+1, pre.begin()+1+left), vector<int>(vin.begin(), mid));
    root->right = reConstructBinaryTree(vector<int>(pre.begin()+1+left, pre.end()), vector<int>(mid+1, vin.end()));
    return root;
}
```

### BM41 输出二叉树的右视图

先使用上一题的代码重建二叉树，再层序遍历，每层取最后一个。

```C++
vector<int> solve(vector<int>& xianxu, vector<int>& zhongxu) {
    TreeNode *root = reConstructBinaryTree(xianxu, zhongxu);
    queue<TreeNode*> q;
    q.push(root);
    vector<int> ans;
    while(!q.empty()){
        int size = q.size();
        TreeNode *last = nullptr;
        for(int i = 0; i < size; i++){
            last = q.front();
            q.pop();
            if(last->left)q.push(last->left);
            if(last->right)q.push(last->right);
        }
        ans.push_back(last->val);
    }
    return ans;
}
```

### 

```C++

```

### 

```C++
d
```

## 堆/栈/队列

### BM42 用两个栈实现队列

```C++
public:
    void push(int node) {
        stack1.push(node);
    }

    int pop() {
        if(stack2.empty()){
            while(!stack1.empty()){
                stack2.push(stack1.top());
                stack1.pop();
            }
        }
        int tmp = stack2.top();
        stack2.pop();
        return tmp;
    }

private:
    stack<int> stack1;
    stack<int> stack2;
```

### BM43 包含min函数的栈

```C++
public:
    void push(int value) {
        s.push(value);
        if(min_s.empty())
            min_s.push(value);
        else min_s.push(std::min(value, min_s.top()));
    }
    void pop() {
        s.pop();
        min_s.pop();
    }
    int top() {
        return s.top();
    }
    int min() {
        return min_s.top();
    }
private:
    stack<int> s;
    stack<int> min_s;
```

### BM44 有效括号序列

```C++
bool isValid(string s) {
    stack<char> left;
    for(char c: s){
        if(c == '(' or c == '[' or c == '{'){
            left.push(c);
        }else{
            if(left.empty()) return false;
            if(c == ')' && left.top() != '(') return false;
            if(c == ']' && left.top() != '[') return false;
            if(c == '}' && left.top() != '{') return false;
            left.pop();
        }
    }
    return left.empty();
}
```

### BM45 滑动窗口的最大值

本来应该是用优先队列来维护窗口内的最大值，但是C++的`priority_queue`无法提供快速的查找功能，这样在串口移动的时候无法删除失效的元素。因此可以改用`multiset`，内部也是有序的，可以直接获取最大值，而且可以O(logN)查找失效元素。

```C++
vector<int> maxInWindows(const vector<int>& nums, int size) {
  multiset<int> m;
  vector<int> ans;
  for(int i = 0; i < nums.size(); i++){
    m.insert(nums[i]);
    if(i >= size) m.erase(m.find(nums[i - size]));
    if(i >= size - 1) ans.push_back(*m.rbegin());
  }
  return ans;
}
```

### BM46 最小的K个数

用一个大根堆，从大到小维护目前遍历过的元素中最小的k个。每次遍历一个新的数，就和堆顶比较，如果比堆顶小，就入堆，并控制堆大小为k；如果比堆顶还大，就跳过。

```C++
vector<int> GetLeastNumbers_Solution(vector<int> input, int k) {
    vector<int> ans;
    if(k == 0)return ans;
    priority_queue<int> pq;
    for(int i : input){
        if(pq.size() < k)
            pq.push(i);
        else{
            if(pq.top() > i){
                pq.pop();
                pq.push(i);
            }
        }
    }
    while(!pq.empty()){
        ans.push_back(pq.top());
        pq.pop();
    }
    return ans;
}
```

### BM47 寻找第K大

两种思路。第一种，构造一个小顶堆，控制堆的size不超过k。在size达到k的时候，新元素和堆顶比较，如果比堆顶更大则可以入堆。最后返回堆顶就是第k大。

```C++
int findKth(vector<int> a, int n, int K) {
    priority_queue<int, vector<int>, greater<int>> pq;
    for(int i : a){
        if(pq.size() < K) pq.push(i);
        else if(pq.top() < i){
            pq.pop();
            pq.push(i);
        }
    }
    return pq.top();
}
```

第二种思路是分治，利用快排的思想。每次只对第k大的元素可能在的半边排序。

```C++
int findKthLargest(vector<int>& a, int n, int k) {
    return quickSort(a, 0, n - 1, k);
}
int quickSort(vector<int>& a, int l, int r, int k){
    int pivot = partition(a, l, r);
    if(pivot == k - 1) return a[pivot];
    else if(pivot > k - 1) return quickSort(a, l, pivot - 1, k);
    else return quickSort(a, pivot + 1, r, k);
}
int partition(vector<int>& a, int l, int r){
    int tmp = a[l];
    while(l < r){
        while(l < r && tmp >= a[r]) r--;
        a[l] = a[r];
        while(l < r && tmp <= a[l]) l++;
        a[r] = a[l];
    }
    a[l] = tmp;
    return l;
}
```

### BM48 数据流中的中位数

维护一个大顶堆和一个小顶堆。大顶堆保存数据流中较小的数据，小顶堆保存较大的数据。插入时尽量插入小顶堆，并且控制小顶堆比大顶堆多1个或者相等。然后如果需要取中位数的时候，奇数情况下从小顶堆取。

```C++
priority_queue<int> big_heap;
priority_queue<int, vector<int>, greater<int>> small_heap;
void Insert(int num) {
    if(!big_heap.empty() && num < big_heap.top())
        big_heap.push(num);
    else 
        small_heap.push(num);
    if(small_heap.size() == big_heap.size() + 2){
        big_heap.push(small_heap.top());
        small_heap.pop();
    }else if(small_heap.size() + 1 == big_heap.size()){
        small_heap.push(big_heap.top());
        big_heap.pop();
    }
}

double GetMedian() { 
    if((small_heap.size() + big_heap.size()) % 2 == 0)
        return (small_heap.top() + big_heap.top()) / 2.0;
    else 
        return small_heap.top();
}
```

### BM49 表达式求值

首先提供的是直接求值的版本。

```C++
int solve(string s) {
    stack<char> ops;
    stack<int> nums;
    for(int i = 0; i < s.size(); i++){
        char c = s[i];
        if(c == '('){
            ops.push(c);
        }else if(c == ')'){
            while(ops.top() != '(')cal(ops, nums);
            ops.pop();
        }else if(c == '+' || c == '-' || c == '*'){
            while(!ops.empty() && priority(ops.top()) >= priority(c))
                cal(ops, nums);
            ops.push(c);
        }else{
            int tmp = 0;
            while('0' <= s[i] && s[i] <= '9'){
                tmp = tmp * 10 + s[i] - '0';
                i++;
            }
            i--;
            nums.push(tmp);
        }
    }
    while(!ops.empty())cal(ops, nums);
    return nums.top();
}

int priority(char op){
    if(op == '+' || op == '-')return 0;
    if(op == '*')return 1;
    return -1;
}

void cal(stack<char>& ops, stack<int>& nums){
    int b = nums.top(); nums.pop();
    int a = nums.top(); nums.pop();
    char op = ops.top(); ops.pop();
    printf("%d %c %d\n", a, op, b);
    if(op == '+')nums.push(a + b);
    else if(op == '-')nums.push(a - b);
    else if(op == '*')nums.push(a * b);
}
```

然后提供的版本由2个函数组成。先由`getSuffixExpr()`将中缀表达式转换为后缀表达式，注意这一步已经将运算符优先级处理完毕。第二部由`parseSuffixExpr()`将后缀表达式转换为计算结果，只一步不需要考虑运算符优先级，只需要按照规则逐个运算符计算即可。

如何处理运算符优先级：每次运算符入栈时，先将栈内优先级大于等于新运算符的出栈，表示这些应当被优先计算。

```C++
int solve(string s) {
    vector<string> exp = getSuffixExpr(s);
    return parseSuffixExpr(exp);
}

vector<string> getSuffixExpr(string s){
    stack<char> ops;
    vector<string> exp;
    for(int i = 0; i < s.size(); i++){
        char c = s[i];
        if(c == '('){
            ops.push(c);
        }else if(c == ')'){
            while(ops.top() != '(')exp.push_back(pop(ops));
            ops.pop();
        }else if(isDigit(c)){
            int tmp = 0;
            while(isDigit(s[i])){
                tmp = tmp * 10 + s[i] - '0';
                i++;
            }
            i--;
            exp.push_back(to_string(tmp));
        }else{
            while(!ops.empty() && priority(ops.top()) >= priority(c))
                exp.push_back(pop(ops));
            ops.push(c);
        }
    }
    while(!ops.empty())exp.push_back(pop(ops));
    return exp;
}

int parseSuffixExpr(vector<string>& exp){
    stack<int> nums;
    for(string& s : exp)
        if(isDigit(s[0]))
            nums.push(atoi(s.c_str()));
        else
            cal(s[0], nums);
    return nums.top();
}

bool isDigit(char c){
    return '0' <= c && c <= '9';
}

string pop(stack<char>& ops){
    string ret(1, ops.top());
    ops.pop();
    return ret;
}

int priority(char op){
    if(op == '+' || op == '-')return 0;
    if(op == '*' || op == '/')return 1;
    return -1;
}

void cal(char op, stack<int>& nums){
    int b = nums.top(); nums.pop();
    int a = nums.top(); nums.pop();
    if(op == '+')nums.push(a + b);
    else if(op == '-')nums.push(a - b);
    else if(op == '*')nums.push(a * b);
    else if(op == '/')nums.push(a / b);
}
```



## 哈希

### BM50 两数之和

```C++
vector<int> twoSum(vector<int>& numbers, int target) {
    unordered_map<int, int> m;
    vector<int> ans(2);
    for(int i = 0; i < numbers.size(); i++){
        if(m[target - numbers[i]] != 0){
            ans[0] = m[target - numbers[i]];
            ans[1] = i + 1;
            break;
        }else{
            m[numbers[i]] = i + 1;
        }
    }
    return ans;
}
```

### BM51 数组中出现次数超过一半的数字

投票法。记录当前遇到的出现最多的数字和次数，如果遍历到一样的就增加次数，不一样则减少次数；次数如果减到-1则换一个数字来记录。

```C++
int MoreThanHalfNum_Solution(vector<int> numbers) {
    int ans = -1, cnt = 0;
    for(int i : numbers){
        if(ans == i){
            cnt++;
        }else{
            if(cnt == 0){
                ans = i;
                cnt++;
            }else{
                cnt--;
            }
        }
    }
    return ans;
}
```

### BM52 *数组中只出现一次的两个数字*

首先需要做过一道前置题目：数组中只出现一次的数字。那道题中将所有的数组元素异或即可，因为出现两次的数字异或后会抵消，最后只留下出现一次的数字。

这道题中有2个数字只出现一次，如果按照同样的方式，只能得到这两个数字的异或值。已知这两个数肯定不同，那么异或的结果必然不同，至少有一位为1。那么就找到这一位，将数组中的数按照这一位是否为1来分成2类，分别将两类数异或后即是所求的答案。可以理解为每一类数各contribute to其中一个答案。

```C++
vector<int> FindNumsAppearOnce(vector<int>& array) {
    int a = 0, p = 1;
    for(int i : array) a ^= i;
    cout << a << endl;
    while((a & p) == 0) p <<= 1;
    cout << p << endl;
    // p在最先遇到a为1的那位上为1
    int n = 0, m = 0;
    for(int i : array){
        if(i & p) n ^= i;
        else m ^= i;
    }
    if(n > m) swap(n, m);
    return {n,m};
}
```

### BM53 *缺失的第一个正整数*

在恢复后，数组应当有 `[1, 2, ..., N]` 的形式，但其中有若干个位置上的数是错误的，每一个错误的位置就代表了一个缺失的正数。我们遍历，如果遍历到的`x=nums[i]`在`[1,N]`内，就把这个数交换到他应该在的位置：`nums[x-1]`。持续交换直到`x`是一个`[1,N]`以外的数，或者就是这个位置应该所在的数：`x-1==i`。

由于每次的交换操作都会使得某一个数交换到正确的位置，因此交换的次数最多为 N，整个方法的时间复杂度为 O(N)。

```C++
int minNumberDisappeared(vector<int>& nums) {
    for(int i = 0; i < nums.size(); i++){
        // 如果nums[i]不是该在的位置上应有的数，则继续循环。注意避免死循环
        while(0 < nums[i] && nums[i] <= nums.size() && nums[nums[i]-1] != nums[i])
            swap(nums[i], nums[nums[i]-1]);
    }
    for(int i = 0; i < nums.size(); i++){
        if(nums[i] != i + 1)
            return i + 1;
    }
    return nums.size() + 1;
}
```

### 三数之和

```C++
d	
```

### BM55 没有重复项数字的全排列

```C++
bool vis[10];
vector<vector<int> > ans;
vector<vector<int> > permute(vector<int> &num) {
    memset(vis, 0, sizeof(vis));
    ans.clear();
    vector<int> tmp;
    dfs(num, tmp);
    return ans;
}

void dfs(vector<int> &num, vector<int> &tmp){
    if(tmp.size() == num.size()){
        ans.push_back(tmp);
        return;
    }
    for(int i = 0; i < num.size(); i++){
        if(!vis[num[i]]){
            tmp.push_back(num[i]);
            vis[num[i]] = 1;
            dfs(num, tmp);
            tmp.pop_back();
            vis[num[i]] = 0;
        }
    }
}
```

### BM56 有重复项数字的全排列

通过`last`来控制，在同一层递归的循环中，不重复使用相同数字。

```C++
bool vis[10];
vector<vector<int> > permuteUnique(vector<int> &nums) {
    vector<vector<int>> ans;
    if(nums.empty())return ans;
    memset(vis, 0, sizeof(vis));
    sort(nums.begin(), nums.end());
    vector<int> tmp;
    dfs(nums, tmp, ans);
    return ans;  
}

void dfs(vector<int>& nums, vector<int>& tmp, vector<vector<int>>& ans){
    if(tmp.size() == nums.size()){
        ans.push_back(tmp);
        return;
    }
    int last = INT_MAX;
    for(int i = 0; i < nums.size(); i++){
        if(vis[i] || last == nums[i])
            continue;
        last = nums[i];
        vis[i] = true;
        tmp.push_back(nums[i]);
        dfs(nums, tmp, ans);
        vis[i] = false;
        tmp.pop_back();
    }
}
```

### BM57 岛屿数量

```C++
int solve(vector<vector<char> >& grid) {
    int ans = 0;
    for(int i = 0; i < grid.size(); i++){
        for(int j = 0; j < grid[0].size(); j++){
            if(grid[i][j] == '1'){
                ans++;
                dfs(i, j, grid);
            }
        }
    }
    return ans;
}

int mr[4] = {0, 1, 0, -1};
int mc[4] = {1, 0, -1, 0};
void dfs(int r, int c, vector<vector<char> >& grid){
    grid[r][c] = '0';
    for(int i = 0; i < 4; i++){
        int nr = r + mr[i];
        int nc = c + mc[i];
        if(nr >= 0 && nr < grid.size() && nc >= 0 && nc < grid[0].size() && grid[nr][nc] == '1'){
            dfs(nr, nc, grid);
        }
    }
}
```

### BM58 字符串的排列

直接用set保存所有遇到过的字符串排列会使得空间复杂度达到O(n!)，通过先将字符串排序，然后每层递归都判断自己这个字符是否是第一次使用，这样可以将空间复杂度减少到O(n)。还有一种next_permutation算法，基于交换，对每一个排列找到需要交换的元素，得到下一个字典序的排列，可以将空间复杂度减少到O(1)。

```C++
vector<string> ans;
vector<bool> vis;
string tmp;
vector<string> Permutation(string str) {
    sort(str.begin(), str.end());
    vis.resize(str.size(), false);
    dfs(str);
    return ans;
}
void dfs(string& str){
    if(tmp.size() == str.size()){
        ans.push_back(tmp);
        return;
    }
    for(int i = 0; i < str.size(); i++){
        if(vis[i])continue;
        if(i > 0 && vis[i-1] && str[i-1] == str[i])continue;
        vis[i] = true;
        tmp.push_back(str[i]);
        dfs(str);
        tmp.pop_back();
        vis[i] = false;
    }
}
```

### 

```C++
d
```

### BM60 括号生成

```C++
vector<string> generateParenthesis(int n) {
    vector<string> ans;
    string tmp;
    dfs(ans, tmp, n, 0);
    return ans;
}

void dfs(vector<string>& ans, string& cur, int n, int l){
    if(cur.size() == 2 * n){
        ans.push_back(cur);
        return;
    }
    if(l < n){
        string tmp = cur + "(";
        dfs(ans, tmp, n, l+1);
    }
    if(l == n || l > cur.size() - l){
        string tmp = cur + ")";
        dfs(ans, tmp, n, l);
    }
}
```

### BM61 矩阵最长递增路径

一种新的写法，不同于我习惯的循环向四个方向扩展。注意这是递归，会先探到死路在逐层返回，所以每个节点都已经充分访问过其可能路径，之后不需要再访问，可直接返回。

```C++
    int n,m;
    int solve(vector<vector<int> >& matrix) {
        n = matrix.size();
        if(n == 0)return 0;
        m = matrix[0].size();
        vector<vector<int>> dp(n, vector<int>(m, -1));
        int ans = 0;
        for(int i = 0; i < n; i++){
            for(int j = 0 ; j < m; j++){
                ans = max(ans, dfs(matrix, dp, i, j, -1));
            }
        }
        return ans;
    }

    int dfs(vector<vector<int> >& matrix, vector<vector<int> >& dp, int r, int c, int pre){
        if(matrix[r][c] <= pre)return 0;
        if(dp[r][c] != -1)return dp[r][c];
        int tmp = 0;
        if(r + 1 < n) tmp = max(tmp, dfs(matrix, dp, r+1, c, matrix[r][c]));
        if(r - 1 >= 0)tmp = max(tmp, dfs(matrix, dp, r-1, c, matrix[r][c]));
        if(c + 1 < m) tmp = max(tmp, dfs(matrix, dp, r, c+1, matrix[r][c]));
        if(c - 1 >= m)tmp = max(tmp, dfs(matrix, dp, r, c-1, matrix[r][c]));
        dp[r][c] = tmp + 1;
        return tmp + 1;
    }
```

## 动态规划

### BM62 斐波那契数列

```C++
int Fibonacci(int n) {
    if(n == 1)return 1;
    if(n == 2)return 1;
    return Fibonacci(n-1) + Fibonacci(n-2);
}
```

### BM63 跳台阶

```C++
int jumpFloor(int n) {
    if(n <= 2)return n;
    return jumpFloor(n-1) + jumpFloor(n-2);
}
```

### BM64 最小花费爬楼梯

```C++
int minCostClimbingStairs(vector<int>& cost) {
    int dp[100005];
    memset(dp, 0, sizeof(dp));
    int n = cost.size();
    for(int i = 2; i <= n; i++){
        dp[i] = min(dp[i-1] + cost[i-1], dp[i-2] + cost[i-2]);
    }
    return dp[n];
}
```
### ❗BM65 最长公共子序列(二)

如果是只用求最长公共子序列的长度，只需要前半部分的代码。如果需要求具体的字串是什么，需要根据长度反推，从后往前推。

[![jjJsSS.png](https://s1.ax1x.com/2022/07/24/jjJsSS.png)](https://imgtu.com/i/jjJsSS)

```C++
string LCS(string s1, string s2) {
    // dp[i][j]表示s[:i]和s[:j]之间的最长公共字串长度
    int dp[2005][2005];
    memset(dp, 0, sizeof(dp));
    for(int i = 1; i <= s1.size(); i++){
        for(int j = 1; j <= s2.size(); j++){
            if(s1[i-1] == s2[j-1]){
                dp[i][j] = dp[i-1][j-1] + 1;
            }else{
                dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
            }
        }
    }
    // 得到了最长公共字串的长度，接下来求对应子串
    printf("%d\n", dp[s1.size()][s2.size()]);
    string ans = "";
    for(int i = s1.size(), j = s2.size(); dp[i][j] >= 1; ){
        if(s1[i-1] == s2[j-1]){
            // dp[i][j]来自dp[i-1][j-1]
            ans += s1[i-1];
            i--; j--;
        }else if(dp[i-1][j] > dp[i][j-1]) i--; // 哪边大就来自哪边
        else j--;
    }
    reverse(ans.begin(), ans.end());
    return ans.empty() ? "-1" : ans;
}
```
### ❗BM66 最长公共子串

全部清零后，直接二重遍历，如果遇到更长的字串就记录长度和结束位置。

```C++
string LCS(string str1, string str2) {
    int dp[5005][5005];
    // 最长公共字串的长度，以及结束位置
    int maxLen = 0, last = 0;
    memset(dp, 0, sizeof(dp));
    for(int i = 1; i <= str1.size(); i++){
        for(int j = 1; j <= str2.size(); j++){
            if(str1[i-1] == str2[j-1]){
                dp[i][j] = dp[i-1][j-1] + 1;
                if(dp[i][j] > maxLen){
                    maxLen = dp[i][j];
                    last = i;
                }
            }
        }
    }
    return str1.substr(last - maxLen, maxLen);
}
```
### BM67 不同路径的数目(一)

能到达`dp[i][j]`的路径只能来自`dp[i-1][j]`和`dp[i][j-1]`。

```C++
int uniquePaths(int m, int n) {
    int dp[105][105];
    memset(dp, 0, sizeof(dp));
    for(int i = 0; i < m; i++){
        for(int j = 0; j < n; j++){
            if(i == 0 || j == 0){
                dp[i][j] = 1;
            }else{
                dp[i][j] = dp[i-1][j] + dp[i][j-1];
            }
        }
    }
    return dp[m-1][n-1];
}
```
### BM68 矩阵的最小路径和

```C++
int minPathSum(vector<vector<int> >& matrix) {
    int dp[505][505];
    int n = matrix.size(), m = matrix[0].size();
    memset(dp, 0, sizeof(dp));
    for(int i = 0; i < n; i++){
        for(int j = 0; j < m; j++){
            if(i == 0 && j == 0) dp[i][j] = matrix[i][j];
            else if(i == 0) dp[i][j] = matrix[i][j] + dp[i][j-1];
            else if(j == 0) dp[i][j] = matrix[i][j] + dp[i-1][j];
            else dp[i][j] = matrix[i][j] + min(dp[i-1][j], dp[i][j-1]);
        }
    }
    return dp[n-1][m-1];
}
```
### BM69 把数字翻译成字符串

由于每次解析要么用1位数，要么用2位数，所以`dp[i]`的状态由`dp[i-1]`和`dp[i-2]`推出。

```C++
int solve(string nums) {
    // dp[i]: 到nums[:i]为止有几种解析方式
    int dp[100];
    memset(dp, 0, sizeof(dp));
    dp[0] = 1;
    for(int i = 1; i <= nums.size(); i++){
        // 如果当前数字是0，那么无法作为1位数解析
        dp[i] = nums[i-1] == '0' ? 0 : dp[i-1];
        if(i >= 2){
            int twoDigits = nums[i-1] - '0' + 10 * (nums[i-2] - '0');
            // 十位数不能是0，并且值要在范围内
            if(nums[i-2] != 0 && twoDigits >= 10 && twoDigits <= 26){
                dp[i] += dp[i-2];
            }
        }
    }
    return dp[nums.size()];
}
```
### BM70 兑换零钱(一)

```C++
int minMoney(vector<int>& arr, int aim) {
    // dp[i]: 达到i金额，最少需要d[i]张纸币
    // MAX: 不可能达到的最大值
    int dp[5005], MAX = aim + 1;
    dp[0] = 0;
    if(aim == 0) return 0;
    for(int i = 1; i <= aim; i++) dp[i] = MAX;
    for(int i = 1; i <= aim; i++){
        for(int v : arr){
            // 目前检查的金额i大于钞票v，说明状态dp[i]可以由dp[i-v]达到
            if(i >= v){
                dp[i] = min(dp[i], dp[i-v] + 1);
            }
        }
    }
    return dp[aim] == MAX ? -1 : dp[aim];
}
```
### BM71 最长上升子序列(一)

```C++
int LIS(vector<int>& arr) {
    // dp[i]: arr[:i]以内的最长上升子序列
    int dp[1005], ans = 0;
    memset(dp, 0, sizeof(dp));
    // 这一步很关键，因为每一个元素都可以作为子序列的起点
    for(int i = 0; i < arr.size(); i++) dp[i] = 1;
    for(int i = 1; i < arr.size(); i++){
        for(int j = 0; j < i; j++){
            if(arr[j] < arr[i]){
                dp[i] = max(dp[i], dp[j] + 1);
                // 答案不一定是dp[arr.size()-1]
                ans = max(ans, dp[i]);
            }
        }
    }
    return ans;
}
```
### BM72 连续子数组的最大和

```C++
int FindGreatestSumOfSubArray(vector<int> array) {
    int ans = array[0];
    for(int i = 1; i < array.size(); i++){
        array[i] = max(array[i], array[i] + array[i-1]);
        ans = max(ans, array[i]);
    }
    return ans;
}
```
### BM73 最长回文子串

中心扩散法和动态规划法的时间复杂度都是O(N^2^)， 但是中心扩散法的空间复杂度是O(1)，动态规划是O(N^2^)。还有一种时间复杂度O(N)的马拉车算法，暂时还没学会。

```C++
int getLongestPalindrome(string A) {
    if(A.size() < 2) return A.size();
    int ans = 1;
    for(int i = 0; i < A.size() - 1; i++){
        ans = max(ans, getLen(A, i, i));
        ans = max(ans, getLen(A, i, i + 1));
    }
    return ans;
}

int getLen(string& A, int l, int r){
    while(l >= 0 && r < A.size()){
        if(A[l] == A[r]){
            l--;
            r++;
        }else break;
    }
    // 此时A[l] ！= A[r]，需要减2
    return r - l + 1 - 2;
}
```
### BM74 数字字符串转化成IP地址

比牛客的官解稍微简洁一点。

```C++
vector<string> restoreIpAddresses(string s) {
    vector<string> ans;
    solve(s, ans, vector<string>(), 0);
    return ans;
}

void solve(string s, vector<string>& ans, vector<string> tmp, int beg){
    if(beg == s.size() && tmp.size() != 4) return;
    if(beg >= s.size() && tmp.size() == 4){
        // 拼接答案
        string s1 = "";
        for(string& i : tmp)
            s1 += i + '.';
        ans.push_back(s1.substr(0, s1.size() - 1));
        return;
    }
    for(int len = 1; len <= 3 && beg + len <= s.size(); len++){
        int num = stoi(s.substr(beg, len));
        if(num > 255 || (len > 1 && s[beg] == '0')) break;
        tmp.push_back(s.substr(beg, len));
        solve(s, ans, tmp, beg + len);
        tmp.pop_back();
    }
}
```
### BM75 编辑距离(一)

```C++

```
### BM76 正则表达式匹配

```C++

```
### BM77 最长的括号子串

始终保持栈底元素为当前已经遍历过的元素中**最后一个没有被匹配的右括号的下标**，这样的做法主要是考虑了边界条件的处理，栈里其他元素维护左括号的下标：

1. 对于遇到的每个`(` ，将它的下标放入栈中
2. 对于遇到的每个`)`，先弹出栈顶元素表示匹配了当前右括号：
   1. 如果栈为空，说明当前的右括号为没有被匹配的右括号，将其下标放入栈中来更新我们之前提到的**最后一个没有被匹配的右括号的下标**
   2. 如果栈不为空，当前右括号的下标减去栈顶元素即为**以该右括号为结尾的最长有效括号的长度**
3. 从前往后遍历字符串并更新答案即可。

需要注意的是，如果一开始栈为空，第一个字符为左括号的时候我们会将其放入栈中，这样就不满足提及的**最后一个没有被匹配的右括号的下标**，为了保持统一，我们在一开始的时候往栈中放入一个值为 -1 的元素。(注意，这样栈里无论如何都**至少有一个元素**）

```C++
int longestValidParentheses(string s) {
    stack<int> st;
    int ans = 0;
    st.push(-1);
    for(int i = 0 ; i < s.size(); i++){
        if(s[i] == '('){
            st.push(i);
        }else{
            if(st.size() <= 1){
                st.pop();
                st.push(i);
            }else{
                st.pop(); 
                ans = max(i - st.top(), ans);
            }  
        }
    }
    return ans;
}
```
### BM78 打家劫舍(一)

`dp[i]`表示`nums[:i]`部分（左闭右开）能取得的最大值。对于每一个i，要么不偷，那么`dp[i+1]=dp[i]`；要么偷，那么`dp[i+1]=dp[i-1]+nums[i]`

```C++
int rob(vector<int>& nums) {
    vector<int> dp(nums.size() + 1, 0);
    for(int i = 0; i < nums.size(); i++){
        dp[i+1] = max(dp[i], dp[i-1] + nums[i]);
    }
    return dp[nums.size()];
}
```
### BM79 打家劫舍(二)

使用两个dp数组，分别忽略第一个和最后一个房子进行dp。

```C++
int rob(vector<int>& nums) {
    // 最开始的房不偷
    vector<int> dp1(nums.size() + 1, 0);
    // 最后一个房不偷
    vector<int> dp2(nums.size() + 1, 0);
    for(int i = 1; i < nums.size(); i++)
        dp1[i+1] = max(dp1[i], dp1[i-1] + nums[i]);
    for(int i = 0; i < nums.size() - 1; i++)
        dp2[i+1] = max(dp2[i], dp2[i-1] + nums[i]);
    return max(dp1[nums.size()], dp2[nums.size() - 1]);
}
```
### BM80 买卖股票的最好时机(一)

不需要用到动态规划，只用遍历数组，维护目前遇到的最小值和最大利润，遍历时更新这两个值即可。

```C++
int maxProfit(vector<int>& prices) {
    int minPrice = prices[0], ans = 0;
    for(int i : prices){
        ans = max(ans, i - minPrice);
        minPrice = min(minPrice, i);
    }
    return ans;
}
```

### BM81 买卖股票的最好时机(二)

一个合理的买卖股票过程就是寻找数组中的一段连续上升子序列。只要找出所有的连续上升子序列，就可以得到总利润。

```C++
int maxProfit(vector<int>& prices) {
    int ans = 0, last = prices[0];
    for(int i = 1; i < prices.size(); i++){
        if(prices[i] < prices[i-1]){
            // 连续上升子序列断了，结算之前的利润，更新起点为当前价格
            ans += prices[i-1] - last;
            last = prices[i];
        }
    }
    if(*prices.rbegin() > last)
        ans += *prices.rbegin() - last;
    return ans;
}
```

### ❓BM82 买卖股票的最好时机(三)

- `dp[i][0]`表示到第i天为止没有买过股票的最大收益
- `dp[i][1]`表示到第i天为止买过一次股票还没有卖出的最大收益
- `dp[i][2]`表示到第i天为止买过一次也卖出过一次股票的最大收益
- `dp[i][3]`表示到第i天为止买过两次只卖出过一次股票的最大收益
- `dp[i][4]`表示到第i天为止买过两次同时也买出过两次股票的最大收益

```C++
int maxProfit(vector<int>& prices) {
    int ans = 0, last = prices[0];
    vector<vector<int>> dp(prices.size(), vector<int>(5, -1e9));
    dp[0][0] = 0;
    dp[0][1] = -prices[0];
    for(int i = 1; i < prices.size(); i++){
        // 没有操作过，所以和前一天一样
        dp[i][0] = dp[i-1][0];
        // 第一次买入，从没操作过转移
        dp[i][1] = max(dp[i-1][1], dp[i-1][0] - prices[i]);
        // 第一次卖出，从第一次买入
        dp[i][2] = max(dp[i-1][2], dp[i-1][1] + prices[i]);
        // 第二次买入，从没操作过转移
        dp[i][3] = max(dp[i-1][3], dp[i-1][2] - prices[i]);
        // 第二次卖出，从第一次买入
        dp[i][4] = max(dp[i-1][4], dp[i-1][3] + prices[i]);
    }
    return max(dp[prices.size()-1][2], dp[prices.size()-1][4]);
}
```

## 字符串

### BM83 字符串变形

```C++
string trans(string s, int n) {
    vector<string> tmp;
    int last = 0;
    for(int i = 0; i < n; i++){
        if('a' <= s[i] && s[i] <= 'z') s[i] -= 'a' - 'A';
        else if('A' <= s[i] && s[i] <= 'Z') s[i] += 'a' - 'A';
        else {
            tmp.push_back(s.substr(last, i - last));
            last = i + 1;

        }
    }
    string ans = s.substr(last, s.size() - last);
    for(auto it = tmp.rbegin(); it != tmp.rend(); ++it){
        ans += " " + (*it);
    }
    return ans;
}
```
### BM84 最长公共前缀

牛客上只有O(N*M)的复杂度，N是字符串数量，M是字符串平均长度。但是牛客的进阶要求是O(N)，题解随便翻翻没有翻到。

```C++
string longestCommonPrefix(vector<string>& strs) {
    if(strs.empty()) return "";
    int len = strs[0].size();
    for(int i = 0; i < len; i++) {
        char c = strs[0][i];
        for(int j = 1; j < strs.size(); j++) {
            if(strs[j].size() == i || strs[j][i] != c){
                return strs[0].substr(0, i);
            }
        }
    }
    return strs[0];
}
```

### BM85 验证IP地址

```C++

```
### BM86 大数加法

```C++
string solve(string s, string t) {
    string ans = "";
    reverse(s.begin(), s.end());
    reverse(t.begin(), t.end());
    int tmp = 0, a, b;
    for(int i = 0; i < s.size() || i < t.size(); i++){
        a = i < s.size() ? s[i] : '0';
        b = i < t.size() ? t[i] : '0';
        tmp += a + b - ('0' * 2);
        ans += (tmp % 10) + '0';
        tmp /= 10;
    }
    if(tmp) ans += '1';
    reverse(ans.begin(), ans.end());
    return ans;
}
```
## 双指针

### BM87 合并两个有序的数组

`A[m:]`部分是空的，所以不能思维定式从小到大处理，而是从大到小处理，优先填补`A[m:]`这部分空数组。

```C++
void merge(int A[], int m, int B[], int n) {
    int a = m - 1, b = n - 1, i = m + n - 1;
    while(a >= 0 || b >= 0){
        if(a < 0) A[i--] = B[b--];
        else if(b < 0) A[i--] = A[a--];
        else if(A[a] > B[b]) A[i--] = A[a--];
        else A[i--] = B[b--];
    }
}
```
### BM88 判断是否为回文字符串

```C++
bool judge(string str) {
    for(int l = 0, r = str.size() - 1; l < r; l++, r--){
        if(str[l] != str[r])
            return false;
    }
    return true;
}
```
### BM89 合并区间

先按照区间的左边界排序，然后逐一判断合并。

```C++
vector<Interval> merge(vector<Interval> &intervals) {
    vector<Interval> ans;
    if(intervals.empty())return ans;
    sort(intervals.begin(), intervals.end(), cmp);
    ans.push_back(intervals[0]);
    for(auto i : intervals){
        auto tmp = ans.back();
        if(tmp.start <= i.start && tmp.end >= i.start){
            ans[ans.size()-1].end = max(tmp.end, i.end);
        }else{
            ans.push_back(i);
        }
    }
    return ans;
}
static bool cmp(const Interval& a, const Interval& b){
    return a.start != b.start ? a.start < b.start : a.end < b.end;
}
```
### BM90 最小覆盖子串

设置两个指针left，right。表示S的子串tmp可由left和right表示，当需要添加元素时候，就将right++，pop元素就left++。

我们用哈希表判断left到right是否完全包含T，动态维护窗口中所有字符以及个数。具体过程如下：

如果新加入的字符是被需要的（指在T里面），那么这个字符加入到窗口中，当窗口中的字符数目和被需要的数目相等时候，匹配度加一。right右移，这里匹配度是window里面的字符与need里面字符相等的数目。

如果新加入的字符不被需要（指不在T里面），right右移

当匹配度等于被需要的字符种类数，说明left-right覆盖到了T的所有字符，并且记录当前的left和right位置，然后就开始向右移动left

如果left位置的字符是被T所需要的，windo所统计的left字符要减一，当窗口中left处的字符数目小于need的字符数目，匹配度减一

如果left位置的字符不是被T所需要的，直接右移即可。

```C++
string minWindow(string S, string T) {
    // write code here
    int need[128], cnt[128], target = 0;
    memset(need, 0, sizeof(need));
    memset(cnt, 0, sizeof(cnt));
    for(char c: T) {
        // 记录T中出现多少种字符
        if(need[c] == 0)target++;
        need[c]++;
    }
    int l = 0, r = 0, start = 0, minlen = INT_MAX;
    // match记录窗口匹配多少个字符
    int match = 0;
    for(; r < S.size(); r++){
        char c = S[r];
        if(need[c] > 0) {//如果这个字符有需求
            cnt[c]++;
            // 如果这个字符的出现次数满足要求了
            if(cnt[c] == need[c]) match++;
        }
        // 当目前字串符合要求时，尽量尝试缩小左边界，直到窗口不符合要求
        while(match == target){
            if(minlen > r - l + 1){
                minlen = r - l + 1;
                start = l;
            }
            c = S[l];
            if(need[c] > 0){
                cnt[c]--;
                if(cnt[c] < need[c]) match--;
            }
            l++;
        }
    }
    return minlen == INT_MAX ? "" : S.substr(start, minlen);
}
```
### BM91 反转字符串

```C++
string solve(string str) {
    for(int i = 0; i < str.size() / 2; i++){
        swap(str[i], str[str.size() - i - 1]);
    }
    return str;
}
```
### 

```C++

```
### 

```C++

```
### 

```C++

```
### 

```C++

```
### 

```C++

```
### 

```C++

```
### 

```C++

```