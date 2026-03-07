# Leetcode Hot100

https://lqn53uggjd7.feishu.cn/wiki/VZMEwd6R2iTwIMk32uNc9wbCnpg?from=from_copylink

## 双指针&滑动窗口

### [283. 移动零](https://leetcode.cn/problems/move-zeroes/)

```python
class Solution(object):
    def moveZeroes(self, nums):
        """
        :type nums: List[int]
        :rtype: None Do not return anything, modify nums in-place instead.
        """
        index = 0
        for i in nums:
            if i != 0:
                nums[index] = i
                index += 1
        while index < len(nums):
            nums[index] = 0
            index += 1
```

两趟扫描，先顺序扫描不为0的元素依次放置到index，index++，第二趟从index开始后面全部赋值为0

```python
class Solution(object):
    def moveZeroes(self, nums):
        """
        :type nums: List[int]
        :rtype: None Do not return anything, modify nums in-place instead.
        """
        n = len(nums)
        left = right = 0
        while right < n:
            if nums[right] != 0:
                nums[left], nums[right] = nums[right], nums[left]
                left += 1
            right += 1
```

一趟扫描，left用于存放下一个要放非0元素的位置，right用于检查当前元素是不是非0



### [75. 颜色分类](https://leetcode.cn/problems/sort-colors/)

```python
class Solution(object):
    def sortColors(self, nums):
        """
        :type nums: List[int]
        :rtype: None Do not return anything, modify nums in-place instead.
        """
        n = len(nums)
        index = 0
        for i in range(n):
            if nums[i] == 0:
                nums[i], nums[index] = nums[index], nums[i]
                index += 1

        for i in range(index, n):
            if nums[i] == 1:
                nums[i], nums[index] = nums[index], nums[i]
                index += 1
```

单指针，类似283. 移动零一次扫描，先把0全部交换到前面，再把1全部交换到0后面



### [11. 盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/)

```python
class Solution(object):
    def maxArea(self, height):
        """
        :type height: List[int]
        :rtype: int
        """
        left = 0
        right = len(height) - 1
        max_area = 0
        while left < right:
            w = right - left
            h = min(height[left], height[right])
            max_area = max(max_area, w * h)

            if(height[left] < height[right]):
                left += 1
            else:
                right -= 1

        return max_area
```

容纳的水量是由  两个指针指向的数字中较小值∗指针之间的距离决定的，因为左右指针每次移动高度（数字中值较小的那一个）向中心。最后停止的条件是left和right指针不重合，即left<right



### [3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

```python
class Solution(object):
    def lengthOfLongestSubstring(self, s):
        """
        :type s: str
        :rtype: int
        """
        maxlen = 0
        for left in range(len(s)):
            hashset = set()
            right = left
            currentlen = 0
            while right < len(s):
                if s[right] in hashset:
                    # 遇到重复，更新最大长度并终止当前起点的遍历
                    maxlen = max(currentlen, maxlen)
                    break
                else:
                    hashset.add(s[right])
                    right += 1
                    currentlen += 1
            # 遍历到末尾仍无重复
            maxlen = max(currentlen, maxlen)
        return maxlen
        
```



### [438. 找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)

```python
class Solution(object):
    def findAnagrams(self, s, p):
        """
        :type s: str
        :type p: str
        :rtype: List[int]
        """
        slen = len(s)
        plen = len(p)
        if slen < plen:
            return []

        res = []
        s_record = [0] * 26
        p_record = [0] * 26
        # 统计p中各个单词出现的频率
        for i in range(plen):
            s_record[ord(s[i]) - ord('a')] += 1
            p_record[ord(p[i]) - ord('a')] += 1

        if s_record == p_record:
            res.append(0)
            
        # 循环次数 = s_len - p_len（比如s长5，p长3，循环2次：i=0→起始索引1；i=1→起始索引2）
        for i in range(slen - plen):
            # 步骤1：移除窗口左边界的字符（s[i]）
            s_record[ord(s[i]) - ord('a')] -= 1
            # 步骤2：添加窗口右边界的新字符（s[i + p_len]）
            s_record[ord(s[i + plen]) - ord('a')] += 1
            if s_record == p_record:
                res.append(i + 1)
        
        return res
```



### [209. 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)

```python
class Solution(object):
    def minSubArrayLen(self, target, nums):
        """
        :type target: int
        :type nums: List[int]
        :rtype: int
        """
        n = len(nums)
        left = 0
        right = 0
        min_len = float('inf')
        cur_sum = 0
        while right < n:
            cur_sum += nums[right]

            # 累加值大于目标值
            while cur_sum >= target:
                min_len = min(min_len, right - left + 1)
                cur_sum -= nums[left]
                left += 1
            right += 1
        
        if min_len != float('inf'):
            return min_len
        else:
            return 0
```



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



## 回溯

### [17. 电话号码的字母组合](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/)

```python
class Solution(object):
    def __init__(self):
        self.letterMap = [
            "",     # 0
            "",     # 1
            "abc",  # 2
            "def",  # 3
            "ghi",  # 4
            "jkl",  # 5
            "mno",  # 6
            "pqrs", # 7
            "tuv",  # 8
            "wxyz"  # 9
        ]
        self.result = []
        self.s = ""

    def backtracking(self, digits, index):
        if index == len(digits):
            self.result.append(self.s)
            return

        # 将索引处的数字转换为整数
        digit = int(digits[index])
        # 获取对应的字符集
        letters = self.letterMap[digit]
        for i in range(len(letters)):
            # 处理字符
            self.s += letters[i]
            # 递归调用，注意索引加1，处理下一个数字
            self.backtracking(digits, index + 1)
            # 回溯，删除最后添加的字符
            self.s = self.s[:-1]

    def letterCombinations(self, digits):
        """
        :type digits: str
        :rtype: List[str]
        """
        self.backtracking(digits, 0)
        return self.result
```



### [78. 子集](https://leetcode.cn/problems/subsets/)

```python
class Solution(object):

    def backtracking(self, nums, start_index, path, result):
        # 收集子集，要放在终止添加的上面，否则会漏掉自己
        result.append(path[:])

        if start_index == len(nums):
            return 

        for i in range(start_index, len(nums)):
            path.append(nums[i])
            self.backtracking(nums, i + 1, path, result)
            path.pop()

    def subsets(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        result = []
        path = []
        self.backtracking(nums, 0, path, result)

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



### [198. 打家劫舍](https://leetcode.cn/problems/house-robber/)

```python
class Solution(object):
    def rob(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        n = len(nums)
        if n == 0:
            return 0
        if n == 1:
            return nums[0]
        # dp[i]：考虑下标i（包括i）以内的房屋，最多可以偷窃的金额为dp[i]
        dp = [0] * n
        dp[0] = nums[0]
        dp[1] = max(nums[0], nums[1])

        for i in range(2, n):
            dp[i] = max(dp[i - 2] + nums[i], dp[i - 1])
        
        return dp[-1]
```



## 动态规划0-1背包

### [416. 分割等和子集](https://leetcode.cn/problems/partition-equal-subset-sum/)

```python
class Solution(object):
    def canPartition(self, nums):
        """
        :type nums: List[int]
        :rtype: bool
        """
        total_sum = sum(nums)
        if total_sum % 2 != 0:
            return False
        
        target_sum = total_sum // 2
        dp = [False] * (target_sum + 1)
        dp[0] = True

        # 遍历物品
        for i in range(len(nums)):
            # 遍历背包
            # 关键：倒序遍历背包容量，避免重复选当前元素
            for j in range(target_sum, nums[i] - 1, -1):
                dp[j] = dp[j] or dp[j - nums[i]]

        return dp[target_sum]
```



## 动态规划完全背包

### [518. 零钱兑换 II](https://leetcode.cn/problems/coin-change-ii/)

```python
class Solution(object):
    def change(self, amount, coins):
        """
        :type amount: int
        :type coins: List[int]
        :rtype: int
        """
        dp = [0] * (amount + 1)
        dp[0] = 1

        # 遍历物品
        for i in range(len(coins)):
            # 遍历背包
            for j in range(coins[i], amount + 1):
                dp[j] += dp[j - coins[i]]
            
        return dp[amount]
```



### [322. 零钱兑换](https://leetcode.cn/problems/coin-change/)

```python
class Solution(object):
    def coinChange(self, coins, amount):
        """
        :type coins: List[int]
        :type amount: int
        :rtype: int
        """
        dp = [float('inf')] * (amount + 1)
        dp[0] = 0

        # 遍历物品
        for i in range(len(coins)):
            # 遍历背包
            for j in range(coins[i], amount + 1):
                dp[j] = min(dp[j - coins[i]] + 1, dp[j])

        if dp[amount] == float('inf'):
            return -1

        return dp[amount] 
```

凑足总额为j - coins[i]的最少个数为dp[j - coins[i]]，那么只需要加上一个钱币coins[i]即dp[j - coins[i]] + 1就是dp[j]（考虑coins[i]）

所以dp[j] 要取所有 dp[j - coins[i]] + 1 中最小的。

递推公式：dp[j] = min(dp[j - coins[i]] + 1, dp[j]);



### [279. 完全平方数](https://leetcode.cn/problems/perfect-squares/)

```python
class Solution(object):
    def numSquares(self, n):
        """
        :type n: int
        :rtype: int
        """
        dp = [float('inf')] * (n + 1)
        dp[0] = 0

        # 遍历物品
        for i in range(1, int(n ** 0.5) + 1):
            # 遍历背包
            for j in range(i * i, n + 1):
                dp[j] = min(dp[j - i * i] + 1, dp[j])
        
        return dp[n]
```

我们只需要遍历所有满足`i² ≤ n`的正整数`i`—— 也就是`i`的取值范围是 `1 ≤ i ≤ √n`



### [139. 单词拆分](https://leetcode.cn/problems/word-break/)

```python
class Solution(object):
    def wordBreak(self, s, wordDict):
        """
        :type s: str
        :type wordDict: List[str]
        :rtype: bool
        """
        hashset = set(wordDict)
        n = len(s)
        dp = [False] * (n + 1)
        dp[0] =  True

        # 遍历背包
        for i in range(1, n + 1):
            # 遍历物品
            for j in range(i):
                # 如果 s[0:j] 可以被拆分成单词，并且 s[j:i] 在单词集合中存在，则 s[0:i] 可以被拆分成单词
                if dp[j] and s[j:i] in hashset:
                    dp[i] = True
                    break
        
        return dp[n]
```

`dp[i]` 表示前`i`个字符能否拆分，是判断的核心目标；

`j` 是分割点，`s[j:i]` 是分割后 “最后一个待验证的单词”；



### [300. 最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/)

```python
class Solution(object):
    def lengthOfLIS(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        res = 1
        n = len(nums)
        dp = [1] * n
        for i in range(1, n):
            for j in range(i):
                if nums[i] > nums[j]:
                    dp[i] = max(dp[i], dp[j] + 1)
            res = max(res, dp[i])

        return res
```

dp[i]表示i之前包括i的以nums[i]结尾的最长递增子序列的长度



### [53. 最大子数组和](https://leetcode.cn/problems/maximum-subarray/)

```python
class Solution(object):
    def maxSubArray(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        res = nums[0]
        n = len(nums)
        dp_max = [nums[0]] * n

        for i in range(1, n):
            dp_max[i] = max(nums[i], dp_max[i - 1] + nums[i])
            res = max(res, dp_max[i])

        return res
```



### [152. 乘积最大子数组](https://leetcode.cn/problems/maximum-product-subarray/)

```python
class Solution(object):
    def maxProduct(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        res = nums[0]
        n = len(nums)
        dp_max = [nums[0]] * n
        dp_min = [nums[0]] * n

        for i in range(1, n):
            dp_max[i] = max(nums[i], dp_max[i - 1] * nums[i], dp_min[i - 1] * nums[i])
            dp_min[i] = min(nums[i], dp_max[i - 1] * nums[i], dp_min[i - 1] * nums[i])
            res = max(res, dp_max[i])
            
        return res
```

类似53. 最大子数组和，我们可以根据正负性进行分类讨论。

考虑当前位置如果是一个负数的话，那么我们希望以它前一个位置结尾的某个段的积也是个负数，这样就可以负负得正，并且我们希望这个积尽可能「负得更多」，即尽可能小。如果当前位置是一个正数的话，我们更希望以它前一个位置结尾的某个段的积也是个正数，并且希望它尽可能地大。



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



### [2. 两数相加](https://leetcode.cn/problems/add-two-numbers/)

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution(object):
    def addTwoNumbers(self, l1, l2):
        """
        :type l1: Optional[ListNode]
        :type l2: Optional[ListNode]
        :rtype: Optional[ListNode]
        """
        if l1.val + l2.val >= 10:
            carry = 1
        else:
            carry = 0
        node = ListNode((l1.val + l2.val) % 10)
        current = node

        if l1.next and l2.next:
            while l1.next and l2.next:
                current.next = ListNode((l1.next.val + l2.next.val + carry) % 10)
                carry = (l1.next.val + l2.next.val + carry) / 10
                l1 = l1.next
                l2 = l2.next
                current = current.next
        
        #如果l2已经遍历完了，则接着只遍历l1
        if l1.next and l2.next is None:
            while l1.next:
                current.next = ListNode((l1.next.val + carry) % 10)
                carry = (l1.next.val + carry) / 10
                l1 = l1.next
                current = current.next

        #如果l1已经遍历完了，则接着只遍历l2
        if l2.next and l1.next is None:
            while l2.next:
                current.next = ListNode((l2.next.val + carry) % 10)
                carry = (l2.next.val + carry) / 10
                l2 = l2.next
                current = current.next

        if carry:
            current.next = ListNode(1)
        
        return node
```

我们同时遍历两个链表，逐位计算它们的和，并与当前位置的进位值相加。具体而言，如果当前两个链表处相应位置的数字为 n1,n2，进位值为 carry，则它们的和为 n1+n2+carry；其中，答案链表处相应位置的数字为 (n1+n2+carry)mod10，即(n1+n2+carry)%10，而新的进位值为 （n1+n2+carry）/10。



## 二叉树DFS

前序遍历：中左右

中序遍历：左中右

后序遍历：左右中

### [144. 二叉树的前序遍历](https://leetcode.cn/problems/binary-tree-preorder-traversal/)

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def preorderTraversal(self, root):
        """
        :type root: Optional[TreeNode]
        :rtype: List[int]
        """
        res = []

        def dfs(node):
            if node is None:
                return None
            # 前序：中左右
            res.append(node.val)
            dfs(node.left)
            dfs(node.right)

        dfs(root)

        return res
```



### [94. 二叉树的中序遍历](https://leetcode.cn/problems/binary-tree-inorder-traversal/)

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def inorderTraversal(self, root):
        """
        :type root: Optional[TreeNode]
        :rtype: List[int]
        """
        res = []
        # 中序：左中右
        def dfs(node):
            if node is None:
                return None
            
            dfs(node.left)
            res.append(node.val)
            dfs(node.right)
        
        dfs(root)

        return res
```



### [145. 二叉树的后序遍历](https://leetcode.cn/problems/binary-tree-postorder-traversal/)

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def postorderTraversal(self, root):
        """
        :type root: Optional[TreeNode]
        :rtype: List[int]
        """
        res = []
        # 后序：左右中
        def dfs(node):
            if node is None:
                return None
            
            dfs(node.left)
            dfs(node.right)
            res.append(node.val)
        
        dfs(root)

        return res
```



### [114. 二叉树展开为链表](https://leetcode.cn/problems/flatten-binary-tree-to-linked-list/)

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def flatten(self, root):
        """
        :type root: Optional[TreeNode]
        :rtype: None Do not return anything, modify root in-place instead.
        """
        res = []

        def dfs(node):
            if node is None:
                return []
            res.append(node)
            dfs(node.left)
            dfs(node.right)

        dfs(root)
        for i in range(len(res) - 1):
            res[i].left = None
            res[i].right = res[i + 1]
```

先dfs前序遍历，然后给每个节点左节点改为null，右节点改为前序遍历的下一个节点



## 二叉树BFS

### [102. 二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/)

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def levelOrder(self, root):
        """
        :type root: Optional[TreeNode]
        :rtype: List[List[int]]
        """
        if root is None:
            return []
            
        queue = collections.deque([root])
        res = []
        while queue:
            level = []
            for i in range(len(queue)):
                current = queue.popleft()
                level.append(current.val)
                if current.left:
                    queue.append(current.left)
                if current.right:
                    queue.append(current.right)
            res.append(level)

        return res
```



### [104. 二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/)

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def maxDepth(self, root):
        """
        :type root: Optional[TreeNode]
        :rtype: int
        """
        if root is None:
            return 0

        queue = collections.deque([root])
        depth = 0
        while queue:
            level = []
            depth += 1
            for i in range(len(queue)):
                current = queue.popleft()
                level.append(current.val)
                if current.left:
                    queue.append(current.left)
                if current.right:
                    queue.append(current.right)
              
        return depth
```



### [111. 二叉树的最小深度](https://leetcode.cn/problems/minimum-depth-of-binary-tree/)

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def minDepth(self, root):
        """
        :type root: Optional[TreeNode]
        :rtype: int
        """
        if root is None:
            return 0
        
        queue = collections.deque([root])
        depth = 0
        
        while queue:
            level = []
            depth += 1
            for i in range(len(queue)):
                current = queue.popleft()
                level.append(current.val)
                if current.left:
                    queue.append(current.left)
                if current.right:
                    queue.append(current.right)
                if current.left is None and current.right is None:
                    return depth

        return depth
```



### [226. 翻转二叉树](https://leetcode.cn/problems/invert-binary-tree/)

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def invertTree(self, root):
        """
        :type root: Optional[TreeNode]
        :rtype: Optional[TreeNode]
        """
        if root is None:
            return None

        root.left, root.right = root.right, root.left
        self.invertTree(root.left)
        self.invertTree(root.right)

        return root
```

按前序遍历的方式翻转二叉树比较合适



### [101. 对称二叉树](https://leetcode.cn/problems/symmetric-tree/)

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def isSymmetric(self, root):
        """
        :type root: Optional[TreeNode]
        :rtype: bool
        """
        if root is None:
            return True

        queue = collections.deque([root])

        while queue:
            level = []
            for i in range(len(queue)):
                current = queue.popleft()
                if current is not None:
                    level.append(current.val)
                    queue.append(current.left)
                    queue.append(current.right)
                else:
                    level.append(None)

            if level != level[::-1]:
                return False
        
        return True
```



## 技巧

### [169. 多数元素](https://leetcode.cn/problems/majority-element/)

```python
class Solution(object):
    def majorityElement(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        nums.sort()
        res = nums[len(nums) // 2]
        return res
```

如果将数组 `nums` 中的所有元素按照单调递增或单调递减的顺序排序，那么下标为 ⌊n/2⌋ 的元素（下标从 `0` 开始）一定是众数。
