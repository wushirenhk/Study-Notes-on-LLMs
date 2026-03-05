# Leetcode hot100

https://lqn53uggjd7.feishu.cn/wiki/VZMEwd6R2iTwIMk32uNc9wbCnpg?from=from_copylink



## 哈希表

### [242. 有效的字母异位词](https://leetcode.cn/problems/valid-anagram/)

```python
class Solution(object):
    def isAnagram(self, s, t):
        """
        :type s: str
        :type t: str
        :rtype: bool
        """
        record = [0] * 26
        for i in s:
            record[ord(i) - ord('a')] += 1

        for j in t:
            record[ord(j) - ord('a')] -= 1

        for p in record:
            if p != 0:
                return False
        return True
```



### [49. 字母异位词分组](https://leetcode.cn/problems/group-anagrams/)

```python
class Solution(object):
    def groupAnagrams(self, strs):
        """
        :type strs: List[str]
        :rtype: List[List[str]]
        """
        mp = collections.defaultdict(list)
        for st in strs:
            record = [0] * 26
            for ch in st:
                record[ord(ch) - ord('a')] += 1
            mp[tuple(record)].append(st)
        
        return list(mp.values())
```

字典不能将数组作为key，必须使用元组tuple作为key



### [1. 两数之和](https://leetcode.cn/problems/two-sum/)

```python
class Solution(object):
    def twoSum(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        mp = dict()
        for index, value in enumerate(nums):
            if target - value in mp:
                return [mp[target - value], index]
            mp[value] = index
        return []

```



### [128. 最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/)

```python
class Solution(object):
    def longestConsecutive(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        longest = 0
        numsset = set(nums)
        for i in numsset:
            if i - 1 not in numsset:
                current_num = i
                current = 1

                while current_num + 1 in numsset:
                    current_num += 1
                    current += 1
                    
                longest = max(longest, current)
            
        return longest
```

将数组中的所有数字放入哈希集合set

遍历每个数字，只从序列起点开始统计

判断起点：数字-1不在集合里，则为序列起点

从序列起点开始，连续检查数字+1是否存在直至断开



### [136. 只出现一次的数字](https://leetcode.cn/problems/single-number/)

```python
class Solution(object):
    def singleNumber(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        hashset = set()
        for i in nums:
            if i in hashset:
                # 元素已存在（第二次出现），移除
                hashset.remove(i)
            else:
                # 元素不存在（第一次出现），加入
                hashset.add(i)
        
        # 集合中仅剩一个元素，转换为列表后取第一个即可
        return list(hashset)[0]
```

set 是「无序的、不支持下标访问」的容器

这里是使用的哈希查找的方法，此题还可以利用位运算解决

对于这道题，可使用异或运算 ⊕。异或运算有以下三个性质。

任何数和 0 做异或运算，结果仍然是原来的数，即 a⊕0=a。
任何数和其自身做异或运算，结果是 0，即 a⊕a=0。
异或运算满足交换律和结合律，即 a⊕b⊕a=b⊕a⊕a=b⊕(a⊕a)=b⊕0=b。

```python
class Solution(object):
    def singleNumber(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        res = 0
        for i in nums:
            res = res ^ i
        return res
```



### [287. 寻找重复数](https://leetcode.cn/problems/find-the-duplicate-number/)

```python
class Solution(object):
    def findDuplicate(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        hashset = set()
        for i in nums:
            if i in hashset:
                return i
            else:
                hashset.add(i)
```



### [15. 三数之和](https://leetcode.cn/problems/3sum/)

```python
class Solution(object):
    def threeSum(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        result = []
        nums.sort()

        for i in range(len(nums)):
            if nums[i] > 0:
                return result
            
            if i > 0 and nums[i - 1] == nums[i]:
                continue
            
            left = i + 1
            right = len(nums) - 1

            while right > left:
                numsum = nums[i] + nums[left] + nums[right]

                if numsum < 0:
                    left += 1
                elif numsum > 0:
                    right -= 1
                else:
                    result.append([nums[i], nums[left], nums[right]])
                
                    while right > left and nums[right] == nums[right - 1]:
                        right -= 1
                    while right > left and nums[left] == nums[left + 1]:
                        left += 1
                    
                    left += 1
                    right -= 1

        return result
```



## 动态规划

### [70. 爬楼梯](https://leetcode.cn/problems/climbing-stairs/)

```python
class Solution(object):
    def climbStairs(self, n):
        """
        :type n: int
        :rtype: int
        """
        if n == 1:
            return 1

        dp = [0] * (n + 1)
        dp[1] = 1
        dp[2] = 2

        for i in range(3, n + 1):
            dp[i] = dp[i - 1] + dp[i - 2]

        return dp[n]
```

dp[i]:达到i阶有dp[i]种方法



### [118. 杨辉三角](https://leetcode.cn/problems/pascals-triangle/)

```python
class Solution(object):
    def generate(self, numRows):
        """
        :type numRows: int
        :rtype: List[List[int]]
        """
        res = []
        for i in range(numRows):
            row = []
            for j in range(i + 1):
                if j == 0 or j == i:
                    row.append(1)
                else:
                    row.append(res[i - 1][j] + res[i - 1][j - 1])
            res.append(row)

        return res
```



## 链表

### [203. 移除链表元素](https://leetcode.cn/problems/remove-linked-list-elements/)

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution(object):
    def removeElements(self, head, val):
        """
        :type head: Optional[ListNode]
        :type val: int
        :rtype: Optional[ListNode]
        """
        # 创建虚拟头部节点以简化删除过程
        dummy_head = ListNode(next = head)
        
        # 遍历列表并删除值为val的节点
        current = dummy_head
        while current.next:
            if current.next.val == val:
                current.next = current.next.next
            else:
                current = current.next
        
        return dummy_head.next
```



### [707. 设计链表](https://leetcode.cn/problems/design-linked-list/)

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class MyLinkedList(object):

    def __init__(self):
        self.dummy_head = ListNode()
        self.size = 0


    def get(self, index):
        """
        :type index: int
        :rtype: int
        """
        if index < 0 or index >= self.size:
            return -1
        current = self.dummy_head
        for i in range(index + 1):
            current = current.next
        return current.val
        

    def addAtHead(self, val):
        """
        :type val: int
        :rtype: None
        """
        self.dummy_head.next = ListNode(val, self.dummy_head.next)
        self.size += 1
        

    def addAtTail(self, val):
        """
        :type val: int
        :rtype: None
        """
        current = self.dummy_head
        while current.next:
            current = current.next
        current.next = ListNode(val)
        self.size += 1
        

    def addAtIndex(self, index, val):
        """
        :type index: int
        :type val: int
        :rtype: None
        """
        current = self.dummy_head
        if index < 0 or index > self.size:
            return -1
        else:
            for i in range(index):
                current = current.next
            current.next =ListNode(val,current.next)
            self.size += 1

    def deleteAtIndex(self, index):
        """
        :type index: int
        :rtype: None
        """
        current = self.dummy_head
        if index < 0 or index >= self.size:
            return -1
        else:
            for i in range(index):
                current = current.next
            current.next = current.next.next
            self.size -= 1

        
# Your MyLinkedList object will be instantiated and called as such:
# obj = MyLinkedList()
# param_1 = obj.get(index)
# obj.addAtHead(val)
# obj.addAtTail(val)
# obj.addAtIndex(index,val)
# obj.deleteAtIndex(index)
```



### [19. 删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution(object):
    def removeNthFromEnd(self, head, n):
        """
        :type head: Optional[ListNode]
        :type n: int
        :rtype: Optional[ListNode]
        """
        def getLenghth(head):
            current = head
            length = 0
            while current:
                current = current.next
                length += 1
            return length
        
        dummy_head = ListNode(next=head)
        current = dummy_head
        length = getLenghth(head)
        for i in range(length - n):
            current = current.next
        current.next = current.next.next

        return dummy_head.next             
```



### [21. 合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution(object):
    def mergeTwoLists(self, list1, list2):
        """
        :type list1: Optional[ListNode]
        :type list2: Optional[ListNode]
        :rtype: Optional[ListNode]
        """
        if list1 is None:
            return list2
        elif list2 is None:
            return list1
        elif list1.val < list2.val:
            list1.next = self.mergeTwoLists(list1.next, list2)
            return list1
        else:
            list2.next = self.mergeTwoLists(list1, list2.next)
            return list2
```



### [160. 相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists/)

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def getIntersectionNode(self, headA, headB):
        """
        :type head1, head1: ListNode
        :rtype: ListNode
        """
        hashset = set()
        currentA = headA
        while currentA:
            hashset.add(currentA)
            currentA = currentA.next

        currentB = headB
        while currentB:
            if currentB not in hashset:
                currentB = currentB.next
            else:
                return currentB
        return None
```

判断两个链表是否相交，可以使用哈希集合存储链表节点。

首先遍历链表 headA，并将链表 headA 中的每个节点加入哈希集合中。然后遍历链表 headB，对于遍历到的每个节点，判断该节点是否在哈希集合中：

如果当前节点不在哈希集合中，则继续遍历下一个节点；

如果当前节点在哈希集合中，则后面的节点都在哈希集合中，即从当前节点开始的所有节点都在两个链表的相交部分，因此在链表 headB 中遍历到的第一个在哈希集合中的节点就是两个链表相交的节点，返回该节点。

如果链表 headB 中的所有节点都不在哈希集合中，则两个链表不相交，返回 null。



注意：题目要求找的是「两个链表的相交节点」（内存地址相同的同一个节点），而不是「值相同的节点」，此处不能写成将currentA.val放在hashset里，然后检查currentB中第一个在hashset中重复的值。



### [206. 反转链表](https://leetcode.cn/problems/reverse-linked-list/)

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution(object):
    def reverseList(self, head):
        """
        :type head: Optional[ListNode]
        :rtype: Optional[ListNode]
        """
        cur = head   
        pre = None
        while cur:
            temp = cur.next # 保存一下 cur的下一个节点，因为接下来要改变cur->next
            cur.next = pre #反转
            #更新pre、cur指针
            pre = cur
            cur = temp
        return pre
```



### [24. 两两交换链表中的节点](https://leetcode.cn/problems/swap-nodes-in-pairs/)

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution(object):
    def swapPairs(self, head):
        """
        :type head: Optional[ListNode]
        :rtype: Optional[ListNode]
        """
        dummy_head = ListNode(next=head)
        current = dummy_head

        while current.next and current.next.next:
            temp = current.next
            temp2 = current.next.next
            temp3 = current.next.next.next

            current.next = temp2
            current.next.next = temp
            current.next.next.next = temp3
            current = current.next.next
        
        return dummy_head.next
            
```



### [234. 回文链表](https://leetcode.cn/problems/palindrome-linked-list/)

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution(object):
    def isPalindrome(self, head):
        """
        :type head: Optional[ListNode]
        :rtype: bool
        """
        vals = []
        current = head
        while current:
            vals.append(current.val)
            current = current.next
        return vals == vals[::-1] 
```

一共为两个步骤：

1. 复制链表值到数组列表中。
2. 使用双指针法判断是否为回文。

回文常用vals == vals[::-1] 进行判断



```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution(object):
    def isPalindrome(self, head):
        """
        :type head: Optional[ListNode]
        :rtype: bool
        """
        def reverseList(head):
            pre = None
            cur = head
            while cur:
                temp = cur.next
                cur.next = pre
                pre = cur
                cur = temp
            return pre
        
        dummy = ListNode()
        copy_current = dummy
        original_current = head
        while original_current:
            copy_current.next = ListNode(original_current.val)
            copy_current = copy_current.next
            original_current = original_current.next
        copy_head = dummy.next
        
        reversed_head = reverseList(copy_head)
        reversed_current = reversed_head
        current = head
        while current:
            if current.val != reversed_current.val:
                return False
            else:
                current = current.next
                reversed_current = reversed_current.next
        return True
```

此为暴力解法，先反转再一个个比较值



### [141. 环形链表](https://leetcode.cn/problems/linked-list-cycle/)

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def hasCycle(self, head):
        """
        :type head: ListNode
        :rtype: bool
        """
        current = head
        hashset = set()
        while current:
            if current in hashset:
                return True
            hashset.add(current)
            current = current.next
        return False
```



### [142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/)

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def detectCycle(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        current = head
        hashset = set()
        while current:
            if current in hashset:
                return current
            hashset.add(current)
            current = current.next
        return None
```

用哈希表解决，和[160. 相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists/)有相似之处

此题也有用快慢指针的解决方法，但更为麻烦，需要数学上想通

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def detectCycle(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        slow = head
        fast = head
        
        while fast and fast.next: 
            slow = slow.next # 慢指针走1步
            fast = fast.next.next # 快指针走2步
            
            # 快慢指针相遇 → 链表有环，开始找环的入口
            if slow == fast:
                # 关键步骤2：快慢指针以相同速度（1步/次）移动，直到相遇
                slow = head
                while slow != fast:
                    slow = slow.next
                    fast = fast.next
                # 相遇点就是环的入口，返回该节点
                return slow
        # 循环终止（fast或fast.next为None）→ 无环，返回None
        return None
```

