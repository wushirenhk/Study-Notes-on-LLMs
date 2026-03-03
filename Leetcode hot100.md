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

