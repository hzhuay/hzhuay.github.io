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

```C++

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

## 

```C++

```

## 

```C++

```

## 

```C++

```

## 

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

### 

```C++
d
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

### 

```C++
d
```

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

### 

```C++
d
```

