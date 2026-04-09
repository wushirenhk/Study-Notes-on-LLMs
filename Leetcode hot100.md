# Leetcode Hot100

https://lqn53uggjd7.feishu.cn/wiki/VZMEwd6R2iTwIMk32uNc9wbCnpg?from=from_copylink

## 华为高频面试

第一梯队

LeetCode 20.有效的括号:5次 ok 
第二梯队(高频)
LeetCode 200.岛屿数量:5次 ok
LeetCode1.两数之和:4次 ok 
LeetCode3.无重复字符的最长子串:3次ok
第三梯队3
LeetCode46.全排列:3次 ok
LeetCode 994.腐烂的橘子:3次
LeetCode 394.字符串解码:2次 ok
LeetCode 739.每日温度:2次 ok
LeetCode56.合并区间:2次 ok
LeetCode 64.最小路径和:2次 ok
第四梯队(出现过至少一次，建议补充刷)
LeetCode 155.最小栈 ok
LeetCode 986.区间列表的交集
LeetCode 132.分割回文串| ok
LeetCode 11.盛最多水的容器 ok
LeetCode 49.字母异位词分组 ok
LeetCode 71.简化路径
LeetCode 217.存在重复元素 ok
LeetCode 1423.可获得的最大点数



华为一面

[239. 滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/)🔥（困难）



## 双指针&滑动窗口

### [283. 移动零](https://leetcode.cn/problems/move-zeroes/)🔥

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



### [75. 颜色分类🔥](https://leetcode.cn/problems/sort-colors/)

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



### [11. 盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/)🔥

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



### [3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)🔥(中等)

4.09

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



### [438. 找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)🔥

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



## 子串

### [560. 和为 K 的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/)🔥（中等）

```python
class Solution(object):
    def subarraySum(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        # 前缀和：记录从数组开头到当前元素的累计和
        prefix = 0
        # 结果：记录和为 k 的子数组数量
        res = 0
        # 哈希表：key = 前缀和，value = 该前缀和出现的次数
        current_sum = {}
        # 初始化：前缀和 0 出现 1 次（处理从数组开头就满足条件的子数组）
        current_sum[prefix] = 1

        # 遍历数组中的每个数字
        for i in nums:
            # 更新当前前缀和
            prefix += i
            
            # 核心逻辑：如果 (当前前缀和 - k) 存在于哈希表中
            # 说明存在 前缀和 = 当前前缀和 - k 的位置，这两个位置之间的子数组和为 k
            if prefix - k in current_sum:
                # 累加符合条件的子数组数量
                res += current_sum[prefix - k]
            
            # 将当前前缀和加入哈希表
            # 如果已存在，次数+1；不存在，设为1
            if prefix in current_sum:
                current_sum[prefix] += 1
            else:
                current_sum[prefix] = 1
            
            # 上面的 if-else 可以简化成这一行（你注释掉的那行）
            # current_sum[prefix] = current_sum.get(prefix, 0) + 1
        return res
```

前缀和



### [239. 滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/)🔥（困难）

4.09

```python
class Solution(object):
    def maxSlidingWindow(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: List[int]
        """
        if len(nums) == 0 or k == 0:
            return []
        # 双向队列用来存数组的索引
        queue = collections.deque()
        res = []
        
        for index, num in enumerate(nums):
            while queue and nums[queue[-1]] <= num:
                queue.pop()
            queue.append(index)

            # 移除窗口队头边界外的元素
            if queue[0] <= index - k:
                queue.popleft()
            
            # 当窗口形成之后，向res中添加元素
            # 如k=3时，index为2时窗口形成
            if index >= k - 1:
                res.append(nums[queue[0]])
        
        return res
```

**用「单调递减双向队列」维护滑动窗口内的最大值，把暴力解法 O (n・k) 优化到 O (n)**。

1.只保留“有可能成为最大值”的元素

2.维护单调递减的双端队列，队头是最大值

3.新元素比队尾元素大时，弹出队尾元素

4.队头元素不在滑动窗口内时，弹出

注意：双向队列中存的一定是序列



### [1423. 可获得的最大点数](https://leetcode.cn/problems/maximum-points-you-can-obtain-from-cards/)（中等）

```python
class Solution:
    def maxScore(self, cardPoints: List[int], k: int) -> int:
        # 解题思路：
        # 只能从数组两端拿 k 张牌，等价于：
        # 数组总和 - 【连续不拿的中间区域的最小和】
        # 不拿的区域长度 = 总长度 - k
        
        # 计算中间不拿的窗口长度
        window_size = len(cardPoints) - k
        
        # 临时和：用于计算滑动窗口的和
        temp_sum = 0
        
        # 第一步：先计算第一个窗口（最左侧）的和
        for i in range(window_size):
            temp_sum += cardPoints[i]
        
        # 记录最小窗口和，初始为第一个窗口的和
        min_sum = temp_sum

        # 第二步：滑动窗口，从左向右移动
        # 每次移动：减去离开窗口的最左元素，加上新进入窗口的右侧元素
        for i in range(window_size, len(cardPoints)):
            temp_sum += cardPoints[i] - cardPoints[i - window_size]
            # 更新最小窗口和
            min_sum = min(min_sum, temp_sum)
        
        # 最终最大得分 = 数组总和 - 中间最小不拿区域的和
        return sum(cardPoints) - min_sum
```

将求取k张卡牌的最大值问题，转化为n-k张卡牌最小值问题，

滑动窗口解决，由于剩余卡牌是连续的，使用一个固定长度为 n−k 的滑动窗口对数组 cardPoints 进行遍历，求出滑动窗口最小值，然后用所有卡牌的点数之和减去该最小值，即得到了拿走卡牌点数之和的最大值。



### 知识：enmerate 和 items（）：

`items()` 是字典专用方法，一次返回 **(键，值)** 一对数据。

for key, count in map.items():

`enumerate()` 只用于**列表、数组这种可索引序列**，作用是：

**返回 (索引，元素)**

```
lst = [10, 20, 30]
for idx, val in enumerate(lst):
    print(idx, val)
# 输出：
# 0 10
# 1 20
# 2 30
```



### [76. 最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/)🔥（困难）

```python
class Solution(object):
    def minWindow(self, s, t):
        """
        :type s: str
        :type t: str
        :rtype: str
        """
         # need 字典：记录 t 中每个字符需要的数量
        need = {}
        # window 字典：记录当前窗口里每个字符的数量
        window = {}
        
        # 滑动窗口的左右指针，左闭右开区间 [left, right)
        left = 0
        right = 0
        
        # valid：记录窗口中【满足 need 数量要求】的字符种类数
        # 当 valid == len(need) 时，说明窗口已经覆盖了 t 所有字符
        valid = 0
        
        # 记录最终结果的起始索引 & 最小长度
        start = 0
        length = float('inf')  # 初始设为无穷大

        # 第一步：统计 t 中每个字符需要多少个，存入 need
        for ch in t:
            need[ch] = need.get(ch, 0) + 1

        # 开始滑动窗口，右指针遍历 s
        while right < len(s):
            # 拿到当前要加入窗口的字符
            ch = s[right]
            # 右指针右移，扩大窗口
            right += 1

            # 如果当前字符是 t 里需要的
            if ch in need:
                # 窗口字典里这个字符数量 +1
                window[ch] = window.get(ch, 0) + 1
                # 如果窗口里这个字符数量 == need 需要的数量
                # 说明这个字符【达标了】，valid +1
                if window[ch] == need[ch]:
                    valid += 1
            
            # 第二步：当窗口满足条件（valid 达标），开始收缩左指针，寻找最小窗口
            while valid == len(need):
                # 现在的窗口比之前记录的更小，更新最小窗口信息
                if right - left < length:
                    length = right - left  # 更新最小长度
                    start = left          # 更新起始位置

                # 要移除窗口的左边字符
                left_ch = s[left]
                # 左指针右移，缩小窗口
                left += 1
            
                # 如果移除的字符是 t 里需要的
                if left_ch in need:
                    # 如果移除前，这个字符刚好达标，移除后就不达标了
                    if window[left_ch] == need[left_ch]:
                        valid -= 1
                    # 窗口字典里这个字符数量 -1
                    window[left_ch] -= 1
        
        # 最后：如果 length 还是无穷大，说明没找到，返回空
        if length == float('inf'):
            return ""
        
        # 截取最小窗口返回
        return s[start : start + length]
```

滑动窗口+哈希表

1.两个哈希表跟着：需求窗口和当前窗口字符

2.左右指针：维护窗口

3.右指针拓展：找到有效窗口

4.左指针收缩：找到最小窗口

[最小覆盖子串 | LeetCode 76_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1m4isBBE69/?spm_id_from=333.337.search-card.all.click&vd_source=9e77deab9cbf476a360f590847f021a1)



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



### [49. 字母异位词分组](https://leetcode.cn/problems/group-anagrams/)🔥（中等）

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



### [1. 两数之和](https://leetcode.cn/problems/two-sum/)🔥（简单）

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



### [128. 最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/)🔥（中等）

4.09

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

注意：循环**`for i in numsset`**：去重，每个数只处理一次 → **O (n) 高效**

而不是**`for i in nums`**：重复遍历重复数字 → **低效，甚至超时**



### [136. 只出现一次的数字](https://leetcode.cn/problems/single-number/)🔥（简单）

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



### [287. 寻找重复数](https://leetcode.cn/problems/find-the-duplicate-number/)🔥（中等）

给定一个包含 `n + 1` 个整数的数组 `nums` ，其数字都在 `[1, n]` 范围内（包括 `1` 和 `n`），可知至少存在一个重复的整数。

假设 `nums` 只有 **一个重复的整数** ，返回 **这个重复的数** 。

你设计的解决方案必须 **不修改** 数组 `nums` 且只用常量级 `O(1)` 的额外空间。

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

哈希表解法

时间复杂度O（n）

空间复杂度O（n）



### [15. 三数之和](https://leetcode.cn/problems/3sum/)🔥（中等）

4.09

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

### [17. 电话号码的字母组合](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/)🔥（中等）

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



### [78. 子集](https://leetcode.cn/problems/subsets/)🔥（中等）

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



### [39. 组合总和](https://leetcode.cn/problems/combination-sum/)🔥（中等）

4.09

```python
class Solution(object):
    def backtracking(self, candidates, target, total, start_index, path, result):
        if total > target:
            return
        if total == target:
            result.append(path[:])
            return
            
        for i in range(start_index, len(candidates)):
            path.append(candidates[i])
            total += candidates[i]
            # 不用i+1了，表示可以重复读取当前的数
            self.backtracking(candidates, target, total, i, path, result)
            total -= candidates[i]
            path.pop()

    def combinationSum(self, candidates, target):
        """
        :type candidates: List[int]
        :type target: int
        :rtype: List[List[int]]
        """
        result = []
        path = []
        self.backtracking(candidates, target, 0, 0, path, result)
        return result
```



### [131. 分割回文串](https://leetcode.cn/problems/palindrome-partitioning/)🔥（中等）

```python
class Solution(object):
    def backtracking(self, s, start_index, path, result):
        if start_index == len(s):
            result.append(path[:])
            return
        
        for i in range(start_index, len(s)):
            # 判断是否回文
            if s[start_index : i + 1] == s[start_index : i + 1][::-1]:
                path.append(s[start_index : i + 1])
                self.backtracking(s, i + 1, path, result)
                path.pop()

    def partition(self, s):
        """
        :type s: str
        :rtype: List[List[str]]
        """
        result = []
        path = []
        self.backtracking(s, 0, path, result)
        return result
```

当 `start_index == len(s)` 时，说明我们已经从字符串的起始位置（0）一直切割到了字符串的末尾（len (s)），意味着完成了一次**完整且有效的切割**（所有切割出的子串都是回文）

此时需要把当前的切割路径（`path`）保存到结果集 `result` 中，然后终止当前递归分支

类似用回溯解决的题目[139. 单词拆分](https://leetcode.cn/problems/word-break/)🔥（中等）但如果用回溯会超时



### [46. 全排列](https://leetcode.cn/problems/permutations/)🔥（中等）

```python
class Solution(object):
    def backtracking(self, nums, used, path, result):
        for i in range(len(nums)):
            if len(path) == len(nums):
                result.append(path[:])
                return

            if used[i] == False:
                path.append(nums[i])
                used[i] = True
                self.backtracking(nums, used, path, result)
                path.pop()
                used[i] = False
            else:
                continue

    def permute(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        used = [False] * len(nums)
        path = []
        result = []
        self.backtracking(nums, used, path, result)
        
        return result
```



### [22. 括号生成](https://leetcode.cn/problems/generate-parentheses/)🔥（中等）

```python
class Solution:
    def backtracking(self, n, left, right, path, result):
        if left + right == 2 * n:
            result.append(path[:])
            return

        if left < n:
            path.append('(')
            self.backtracking(n, left + 1, right, path, result)
            path.pop()

        if right < left:
            path.append(')')
            self.backtracking(n, left, right + 1, path, result)
            path.pop()

    def generateParenthesis(self, n: int) -> List[str]:
        path = []
        result = []
        final_result = []
        self.backtracking(n, 0, 0, path, result)
        
        for path in result:
            temp = ""
            for i in path:
                temp = temp + i
            final_result.append(temp)

        return final_result        
```



### [79. 单词搜索](https://leetcode.cn/problems/word-search/)🔥（中等）

```python
class Solution(object):
    def backtracking(self, board, word, i, j, index):
        if i < 0 or i >= len(board):
            return False
        if j <0 or j >= len(board[0]):
            return False
        if board[i][j] != word[index]:
            return False
    
        if index == len(word) - 1: 
            return True
        # 修改board的状态表明该网格已经被选择过
        board[i][j] = ''
        res = self.backtracking(board, word, i + 1, j, index + 1) or self.backtracking(board, word, i - 1, j, index + 1) or self.backtracking(board, word, i, j + 1, index + 1)or self.backtracking(board, word, i, j - 1, index + 1) 
        board[i][j] = word[index]
        return res


    def exist(self, board, word):
        """
        :type board: List[List[str]]
        :type word: str
        :rtype: bool
        """
        for i in range(len(board)):
            for j in range(len(board[0])):
                if self.backtracking(board, word, i, j, 0):
                    return True
        return False
```



### [51. N 皇后](https://leetcode.cn/problems/n-queens/)🔥（困难）

```python
class Solution(object):
    def solveNQueens(self, n):
        """
        :type n: int
        :rtype: List[List[str]]
        """
        result = []
        chessboard = ['.' * n for _ in range(n)]
        self.backtracking(n, 0, chessboard, result)
        
        final_result = []
        for path in result:
            current_board = []
            for row in path:
                # 将每行的字符列表转为字符串（如 ['.','Q','.','.'] → ".Q.."）
                current_board.append(''.join(row))
            final_result.append(current_board)

        return final_result
    
    def backtracking(self, n, row, chessboard, result):
        # row行 col列
        if row == n:
            result.append(chessboard[:])
            return
        
        for col in range(n):
            if self.isValid(row, col, chessboard):
                chessboard[row] = chessboard[row][:col] + 'Q' + chessboard[row][col + 1 :]
                self.backtracking(n, row + 1, chessboard, result)
                chessboard[row] = chessboard[row][:col] + '.' + chessboard[row][col + 1 :]
    
    def isValid(self, row, col, chessboard):
        for i in range(row):
            if chessboard[i][col] == 'Q':
                return False
        
        i, j = row - 1, col - 1
        while i >= 0 and j >= 0:
            if chessboard[i][j] == 'Q':
                return False
            i -= 1
            j -= 1

        i, j = row - 1, col + 1
        while i >= 0 and j < len(chessboard):
            if chessboard[i][j] == 'Q':
                return False
            i -= 1
            j += 1

        return True
```



## 二分查找

### [35. 搜索插入位置](https://leetcode.cn/problems/search-insert-position/)🔥（简单）

请必须使用时间复杂度为 `O(log n)` 的算法。

```python
class Solution(object):
    def searchInsert(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        left = 0
        right = len(nums) - 1

        while left <= right:
            middle = left + (right - left) // 2
            if nums[middle] == target:
                return middle
            elif nums[middle] < target:
                left = middle + 1
            elif nums[middle] > target:
                right = middle - 1
        return left
```

注意：二分查找middle值用

middle = left + （right - left）// 2

二分查找条件是while left <= right

安全、通用、无溢出



### [704. 二分查找](https://leetcode.cn/problems/binary-search/)

```python
class Solution(object):
    def search(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        left = 0
        right = len(nums) - 1

        while left <= right:
            middle = left + (right - left) // 2
            if nums[middle] == target:
                return middle
            elif nums[middle] < target:
                left = middle + 1
            elif nums[middle] > target:
                right = middle - 1
        return -1
```



### [74. 搜索二维矩阵](https://leetcode.cn/problems/search-a-2d-matrix/)🔥（中等）

```python
class Solution(object):
    def searchMatrix(self, matrix, target):
        """
        :type matrix: List[List[int]]
        :type target: int
        :rtype: bool
        """
        m = len(matrix)
        n = len(matrix[0])
        # 展开成虚拟一维数组
        left = 0
        right = m * n - 1

        while left <= right:
            middle = left + (right - left) // 2
            row = middle // n
            col = middle % n
            if matrix[row][col] == target:
                return True
            elif matrix[row][col] < target:
                left = middle + 1
            elif matrix[row][col] > target:
                right = middle - 1
        return False
```

- 时间复杂度：*O*(log*mn*)，其中 *m* 和 *n* 分别是矩阵的行数和列数。
- 空间复杂度：*O*(1)。



### [240. 搜索二维矩阵 II](https://leetcode.cn/problems/search-a-2d-matrix-ii/)🔥（中等）

```python
class Solution(object):
    def searchMatrix(self, matrix, target):
        """
        :type matrix: List[List[int]]
        :type target: int
        :rtype: bool
        """
        for row in matrix:
            if self.binSearch(row, target):
                return True
        return False

    def binSearch(self, nums, target):
        left = 0
        right = len(nums) - 1

        while left <= right:
            middle = left + (right - left) // 2
            if nums[middle] == target:
                return True
            elif nums[middle] < target:
                left = middle + 1
            elif nums[middle] > target:
                right = middle - 1
        return False
```

- 时间复杂度：*O*(*m*log*n*)。对一行使用二分查找的时间复杂度为 *O*(log*n*)，最多需要进行 *m* 次二分查找。
- 空间复杂度：*O*(1)。



### [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)🔥（中等）

```python
class Solution(object):
    def searchRange(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        leftindex = self.binSearch(nums, target, True)
        rightindex = self.binSearch(nums, target, False)
        return [leftindex, rightindex]
        
    def binSearch(self, nums, target, isLeft):
        left = 0
        right = len(nums) - 1
        index = -1

        while left <= right:
            middle = left + (right - left) // 2
            if nums[middle] == target:
                index = middle
                if isLeft:
                    right = middle - 1
                else:
                    left = middle + 1
            elif nums[middle] < target:
                left = middle + 1
            elif nums[middle] > target:
                right = middle - 1
        
        return index
```



### [33. 搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array/)🔥（中等）

```python
class Solution(object):
    def search(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        left = 0
        right = len(nums) - 1

        while left <= right:
            middle = (left + right) // 2
            if nums[middle] == target:
                return middle

            if nums[left] <= nums[middle]:
                if nums[left] <= target and target < nums[middle]:
                    right = middle - 1
                else:
                    left = middle + 1
            else:
                if nums[middle] < target and target <= nums[right]:
                    left = middle + 1
                else:
                    right = middle - 1
        return -1
```

[【小白都能听懂的算法课】【力扣】【Leetcode33】搜索旋转排序数组 | 二分查找 | 数组_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1tz421r7xC/?spm_id_from=333.337.search-card.all.click&vd_source=9e77deab9cbf476a360f590847f021a1)

将旋转排序后的数组分为四段进行讨论



### [153. 寻找旋转排序数组中的最小值](https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array/)🔥（中等）

```python
class Solution(object):
    def findMin(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        left = 0
        right = len(nums) - 1

        # 注意这里是 < 不是 <=
        while left < right:
            middle = left + (right - left) // 2
            if nums[middle] < nums[right]:
                right = middle
            else:
                left = middle + 1
        
        return nums[left]
```

这道题难以理解，需要背答案



### [4. 寻找两个正序数组的中位数](https://leetcode.cn/problems/median-of-two-sorted-arrays/)🔥（困难）

算法的时间复杂度应该为 `O(log (m+n))`

```python
class Solution(object):
    def findMedianSortedArrays(self, nums1, nums2):
        """
        :type nums1: List[int]
        :type nums2: List[int]
        :rtype: float
        """
        # 确保 nums1 是较短的数组，减少二分次数，同时防止 j 越界
        if len(nums2) < len(nums1):
            nums1, nums2 = nums2, nums1
        
        m, n = len(nums1), len(nums2)
        # 二分查找的左右边界，在 nums1 上找切分点 i
        left, right = 0, m

        while left <= right:
            # 计算 nums1 的切分点 i，防止溢出写法
            i = left + (right - left) // 2
            # 根据总长度计算 nums2 的切分点 j，保证左右两部分总元素相等
            j = (m + n + 1) // 2 - i
            
            # ---------- 处理四个边界值 ----------
            # nums1 左半部分的最大值，i=0 表示 nums1 左半无元素，设为负无穷
            if i > 0:
                left1 = nums1[i - 1]
            else:
                left1 = float('-inf')
            
            # nums1 右半部分的最小值，i=m 表示 nums1 右半无元素，设为正无穷
            if i < m:
                right1 = nums1[i]
            else:
                right1 = float('inf')
            
            # nums2 左半部分的最大值，j=0 表示 nums2 左半无元素，设为负无穷
            if j > 0:
                left2 = nums2[j - 1]
            else:
                left2 = float('-inf')
            
            # nums2 右半部分的最小值，j=n 表示 nums2 右半无元素，设为正无穷
            if j < n:
                right2 = nums2[j]
            else:
                right2 = float('inf')
            
            # ---------- 核心判断：找到正确切分 ----------
            # 满足左边所有数 <= 右边所有数，切分合法
            if left1 <= right2 and left2 <= right1:
                # 总长度奇数：中位数 = 左半部分最大值
                if (m + n) % 2 == 1:
                    return max(left1, left2)
                # 总长度偶数：中位数 = (左半最大 + 右半最小) / 2
                else:
                    return (max(left1, left2) + min(right1, right2)) / 2.0
            
            # ---------- 调整二分范围 ----------
            # nums1 左半部分太大，需要把切分点 i 向左移
            # 相当于是nums1[middle]>target，left = middle， right = middle - 1
            elif left1 > right2:
                right = i - 1
            # nums2 左半部分太大，需要把切分点 i 向右移
            elif left2 > right1:
                left = i + 1
```

题目要求时间复杂度为logn级别的，考虑二分查找

[寻找两个正序数组的中位数 | LeetCode 4_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV162PVzPECg/?spm_id_from=333.337.search-card.all.click)



## 栈

### [232. 用栈实现队列](https://leetcode.cn/problems/implement-queue-using-stacks/)

```python
class MyQueue(object):

    def __init__(self):
        self.stack_in = []
        self.stack_out = []

    def push(self, x):
        """
        :type x: int
        :rtype: None
        """
        self.stack_in.append(x)
        

    def pop(self):
        """
        :rtype: int
        """
        if self.empty():
            return None
        
        if self.stack_out:
            return self.stack_out.pop()
        else:
            for i in range(len(self.stack_in)):
                self.stack_out.append(self.stack_in.pop())
            return self.stack_out.pop()
        

    def peek(self):
        """
        :rtype: int
        """
        res = self.pop()
        self.stack_out.append(res)
        return res

    def empty(self):
        """
        :rtype: bool
        """
        return not (self.stack_in or self.stack_out)
        


# Your MyQueue object will be instantiated and called as such:
# obj = MyQueue()
# obj.push(x)
# param_2 = obj.pop()
# param_3 = obj.peek()
# param_4 = obj.empty()
```



### [20. 有效的括号](https://leetcode.cn/problems/valid-parentheses/)🔥（简单）

```python
class Solution(object):
    def isValid(self, s):
        """
        :type s: str
        :rtype: bool
        """
        stack = []

        for item in s:
            if item == '(':
                stack.append(')')
            elif item == '[':
                stack.append(']')
            elif item == '{':
                stack.append('}')
            # 注意要先判断栈是否为空，再判断栈顶元素是否和当前元素相等
            # 右括号必须匹配栈顶，否则直接不合格！
            elif not stack or stack[-1] != item:
                return False
            else:
                stack.pop()
            
        return True if not stack else False
```



### [32. 最长有效括号](https://leetcode.cn/problems/longest-valid-parentheses/)🔥（困难）

```python
class Solution(object):
    def longestValidParentheses(self, s):
        """
        :type s: str
        :rtype: int
        """
        stack = [] 
        mark = [False] * len(s)

        for i in range(len(s)):
            if s[i] == '(':
                # 栈里只存左括号 ( 的下标
                stack.append(i)
            # 只要栈是非空的，就是合法匹配，非法的括号直接跳过去了
            elif stack:
                mark[stack[-1]] =  True
                mark[i] = True
                stack.pop()
        
        temp = 0
        res = 0
        for i in range(len(mark)):
            if mark[i]:
                temp += 1
                res = max(res, temp)
            else:
                temp = 0

        return res
```



### [71. 简化路径](https://leetcode.cn/problems/simplify-path/)（中等）

```python
class Solution(object):
    def simplifyPath(self, path):
        """
        :type path: str
        :rtype: str
        """ 
        names = path.split("/")
        stack = []
        
        for x in names:
            # 如果片段是 .. 表示回到上一级目录
            if x == "..":
                # 只有栈不为空时，才弹出栈顶（回到上一级）
                if stack:
                    stack.pop()
            # 如果片段 非空 且 不是 .（当前目录），说明是有效目录名
            elif x and x != ".":
                # 把有效目录名加入栈
                stack.append(x)

        # 最终用 / 连接栈里的目录，并在最前面加根目录 /
        return "/" + "/".join(stack)
```



### [155. 最小栈](https://leetcode.cn/problems/min-stack/)🔥（中等）

```python
class MinStack(object):

    def __init__(self):
        self.stack = []
        self.stackmin = []

    def push(self, val):
        """
        :type val: int
        :rtype: None
        """
        self.stack.append(val)
        if self.stackmin:
            temp = min(val, self.stackmin[-1])
            self.stackmin.append(temp)
        else:
            self.stackmin.append(val)
        
    def pop(self):
        """
        :rtype: None
        """
        self.stack.pop()
        self.stackmin.pop()

    def top(self):
        """
        :rtype: int
        """
        return self.stack[-1]

    def getMin(self):
        """
        :rtype: int
        """
        return self.stackmin[-1]


# Your MinStack object will be instantiated and called as such:
# obj = MinStack()
# obj.push(val)
# obj.pop()
# param_3 = obj.top()
# param_4 = obj.getMin()
```



### [394. 字符串解码](https://leetcode.cn/problems/decode-string/)🔥（中等）

```python
class Solution(object):
    def decodeString(self, s):
        """
        :type s: str
        :rtype: str
        """
        # 初始化栈，用来存储数字、字母、括号
        stack = []
        
        # 遍历字符串里的每一个字符
        for ch in s:
            # 只要不是 】，就直接入栈
            if ch != ']': 
                stack.append(ch)
            
            # 遇到 】，开始解码
            else:
                # 1. 先弹出 【 前面的所有字母，拼成要重复的字符串
                temp = ""
                while stack[-1] != '[':
                    temp = stack.pop() + temp
                
                # 2. 把 【 弹出去（没用了）
                stack.pop() 
                
                # 3. 弹出【前面的所有数字（可能是多位数）
                num = ''
                while stack and stack[-1].isdigit():
                    num = stack.pop() + num
                
                # 4. 字符串 × 数字，结果放回栈
                stack.append(temp * int(num))
        
        # 把栈里所有内容拼起来就是最终结果
        res = ''
        while stack:
            res = stack.pop() + res

        return res       
```



### [739. 每日温度](https://leetcode.cn/problems/daily-temperatures/)🔥（中等）

```python
class Solution(object):
    def dailyTemperatures(self, temperatures):
        """
        :type temperatures: List[int]
        :rtype: List[int]
        """
        res = [0] * len(temperatures)
        stack = [0] # 存入元素对应的下标
        for i in range(1, len(temperatures)):
            if temperatures[i] <= temperatures[stack[-1]]:
                stack.append(i)
            else:
                while stack and temperatures[i] > temperatures[stack[-1]]:
                    res[stack[-1]] = i - stack[-1]
                    stack.pop()
                stack.append(i)

        return res
```

这里我们要使用递增循序（再强调一下是指从栈头到栈底的顺序），因为只有递增的时候，栈里要加入一个元素i的时候，才知道栈顶元素在数组中右面第一个比栈顶元素大的元素是i。

递增顺序是找比当前元素大的



### [42. 接雨水](https://leetcode.cn/problems/trapping-rain-water/)🔥（困难）

```python
class Solution(object):
    def trap(self, height):
        """
        :type height: List[int]
        :rtype: int
        """
        stack = [0]
        res = 0

        for i in range(1,  len(height)):
            while stack and height[i] >= height[stack[-1]]:
                mid_height = stack.pop()
                if stack:
                    # 雨水高度是 min(凹槽左侧高度, 凹槽右侧高度) - 凹槽底部高度
                    h = min(height[i], height[stack[-1]]) - height[mid_height]
                    w = i - stack[-1] - 1
                    res += h * w
            stack.append(i)

        return res
```

单调栈中为递增序列，求左右比当前更大的元素



### [84. 柱状图中最大的矩形](https://leetcode.cn/problems/largest-rectangle-in-histogram/)🔥（困难）

```python
class Solution(object):
    def largestRectangleArea(self, heights):
        """
        :type heights: List[int]
        :rtype: int
        """
        stack = [0]
        res = 0
        heights.insert(0,0)
        heights.append(0) 
        # 也可以写成[0] + heights + [0]

        for i in range(1, len(heights)):
            while stack and heights[i] <= heights[stack[-1]]:
                mid_height = stack.pop()
                if stack:
                    h = heights[mid_height]
                    w = i - stack[-1] - 1
                    res = max(res, h * w)
            stack.append(i)
        
        return res
```

单调栈中为递减序列，求左右比当前更小的元素

首先来说末尾为什么要加元素0？

[84.柱状图中最大的矩形 | 代码随想录](https://programmercarl.com/0084.柱状图中最大的矩形.html#思路)



### 知识：queue（列表模拟）和 deque（双端队列）

**列表 list = 栈（后进先出）**

**deque = 队列（先进先出）**

queue = [] 

queue.pop()  

删最后一个 → 栈 → DFS（深度优先）



q = collections.deque()

q.popleft()  # 删第一个 → 队列 → BFS（广度优先）

| 方式      | 命令      | 结构 | 适合   |
| --------- | --------- | ---- | ------ |
| 列表 list | pop()     | 栈   | DFS    |
| 列表 list | pop(0)    | 队列 | 小数据 |
| deque     | popleft() | 队列 | BFS    |

有时可以用[] + pop(0)模拟队列，但是list.pop(0) → 慢到爆炸 O (n) deque.popleft() → 飞快 O (1)



## 堆

### [215. 数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/)🔥（中等）

4.09

你必须设计并实现时间复杂度为 `O(n)` 的算法解决此问题。

```python
class Solution(object):
    def findKthLargest(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        pivot = random.choice(nums)

        left = [x for x in nums if x > pivot]
        mid = [x for x in nums if x == pivot]
        right = [x for x in nums if x < pivot]

        if k <= len(left):
            return self.findKthLargest(left, k)
        elif k <= len(left) + len(mid):
            return pivot
        else:
            return self.findKthLargest(right, k - len(left) - len(mid))
```

快速选择，时间复杂度O（n），空间复杂度O（n），方法使用了额外的数组空间，如果原地创建分区，可将空间复杂度降至O（1）

[数组中的第k个最大元素 | LeetCode 215_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1hTfBBuE4T/?spm_id_from=333.337.search-card.all.click&vd_source=9e77deab9cbf476a360f590847f021a1)

注意：这里left中的元素是放比pivot大的元素，right中放的是比pivot小的元素



### [347. 前 K 个高频元素](https://leetcode.cn/problems/top-k-frequent-elements/)🔥（中等）

```python
class Solution(object):
    def topKFrequent(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: List[int]
        """
    # 第一步：统计每个数字出现的频率，用字典存储
        nums_map = {}
        # 遍历数组中的每一个数字
        for index in range(len(nums)):
            # key=数字，value=出现次数
            nums_map[nums[index]] = nums_map.get(nums[index], 0) + 1
        
        # 第二步：构建一个大小为k的小顶堆
        pri_que = []

        # 遍历频率字典中的每个数字和它的频率
        for key, count in nums_map.items():
            # 把（频率，数字）推入小顶堆
            # 堆会自动按频率排序，堆顶 = 频率最小
            heapq.heappush(pri_que, (count, key))
            
            # 只要堆的大小超过k，就弹出堆顶（最小频率）
            # 这样能保证堆里永远只保留频率最大的k个
            if len(pri_que) > k:
                heapq.heappop(pri_que)
        
        # 第三步：把堆中剩下的k个元素取出，倒序存入结果
        # 因为小顶堆弹出顺序是：从小到大
        # 我们要从后往前放，让结果从大到小显示
        res = [0] * k
        for i in range(k - 1, -1, -1):
            # [0] 位置 = 频率 [1] 位置 = 数字本身
            res[i] = heapq.heappop(pri_que)[1]
        
        # 返回最终结果
        return res
```

小顶堆，对应的python数据结构为

import heapq

初始化堆：

pri_que = []

heapq.heappush(堆(pri_queue), 要加的元素)

```python
import heapq

heap = []        # 空堆
heapq.heappush(heap, 5)
heapq.heappush(heap, 2)
heapq.heappush(heap, 8)

print(heap)  # [2,5,8]  最小的 2 自动跑到最上面
```

heapq.heappop

```python
import heapq

heap = [2,5,8]
print(heapq.heappop(heap))  # 弹出 2
print(heapq.heappop(heap))  # 弹出 5
print(heap)  # [8]
```

Python **没有直接大顶堆**

**大顶堆 = 小顶堆 + 存负数**



## 贪心算法

### [55. 跳跃游戏](https://leetcode.cn/problems/jump-game/)🔥（中等）

```python
class Solution(object):
    def canJump(self, nums):
        """
        :type nums: List[int]
        :rtype: bool
        """
        cover = 0
        if len(nums) == 1:
            return True

        i = 0
        while i <= cover:
            cover = max(i + nums[i], cover)
            if cover >= len(nums) - 1:
                return True
            i += 1
        
        return False
```

贪心算法局部最优解：每次取最大跳跃步数（取最大覆盖范围），整体最优解：最后得到整体最大覆盖范围，看是否能到终点。



### [45. 跳跃游戏 II](https://leetcode.cn/problems/jump-game-ii/)🔥（中等）

```python
class Solution(object):
    def jump(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        if len(nums)==1:  # 如果数组只有一个元素，不需要跳跃，步数为0
            return 0
        
        i = 0  # 当前位置
        count = 0  # 步数计数器
        cover = 0  # 当前能够覆盖的最远距离
        
        while i <= cover:  # 当前位置小于等于当前能够覆盖的最远距离时循环
            for i in range(i, cover+1):  # 遍历从当前位置到当前能够覆盖的最远距离之间的所有位置
                cover = max(nums[i]+i, cover)  # 更新当前能够覆盖的最远距离
                if cover >= len(nums)-1:  # 如果当前能够覆盖的最远距离达到或超过数组的最后一个位置，直接返回步数+1
                    return count+1
            count += 1  # 每一轮遍历结束后，步数+1
```

贪心的思路，局部最优：当前可移动距离尽可能多走，如果还没到终点，步数再加一。整体最优：一步尽可能多走，从而达到最少步数。



### [763. 划分字母区间](https://leetcode.cn/problems/partition-labels/)🔥（中等）

```python
class Solution(object):
    def partitionLabels(self, s):
        """
        :type s: str
        :rtype: List[int]
        """
        # 存储每个字符最后出现的位置
        last_occurence = {}
        for index, ch in enumerate(s):
            last_occurence[ch] = index
        
        res = []
        start = 0
        end = 0
        for index, ch in enumerate(s):
            # 找到当前字符出现的最远位置
            end = max(end, last_occurence[ch])
            # 如果当前位置是最远位置，表示可以分割出一个区间
            if index == end:
                res.append(end - start + 1)
                start = index + 1
        
        return res
```



## 普通数组



### [56. 合并区间](https://leetcode.cn/problems/merge-intervals/)🔥（中等）

```python
class Solution(object):
    def merge(self, intervals):
        """
        :type intervals: List[List[int]]
        :rtype: List[List[int]]
        """
        result = []
        if len(intervals) == 0:
            return []
        # 按照区间的左边界进行排序
        intervals.sort(key=lambda x: x[0])

        result.append(intervals[0])
        for i in range(1, len(intervals)):
            if intervals[i][0] <= result[-1][1]:
                # 合并区间，只需要更新结果集最后一个区间的右边界，因为根据排序，左边界已经是最小的
                result[-1][1] = max(result[-1][1], intervals[i][1])
            else:
                result.append(intervals[i])
        
        return result
```

本质上就是贪心算法，类似于55. 跳跃游戏



### [986. 区间列表的交集](https://leetcode.cn/problems/interval-list-intersections/)（中等）

```python
class Solution:
    def intersection(self, List1, List2):
        left_1, right_1 = List1[0], List1[1]
        left_2, right_2 = List2[0], List2[1]

        # 两个区间交集的左边界：取两个左端点的较大值
        left = max(left_1, left_2)
        # 两个区间交集的右边界：取两个右端点的较小值
        right = min(right_1, right_2)

        # 如果左边界 > 右边界，说明无交集，返回空
        if left > right:
            return []
        else:
            return [left, right]

    # 主函数：求两个有序区间列表的所有交集
    def intervalIntersection(self, firstList: List[List[int]], secondList: List[List[int]]) -> List[List[int]]:
        i, j = 0, 0
        res = []

        # 双指针遍历，只要任意一个列表遍历完就结束
        while i < len(firstList) and j < len(secondList):
            # 计算当前两个指针对应区间的交集
            path = self.intersection(firstList[i], secondList[j])
            # 如果有交集，加入结果集
            if len(path) != 0:
                res.append(path)

            # 移动指针策略：
            # 哪个区间的右端点更小，就移动哪个指针（因为它不可能再和后面的区间相交）
            if firstList[i][1] < secondList[j][1]:
                i += 1
            else:
                j += 1

        return res
```

双指针，本题亦有归并排序解法



### [88. 合并两个有序数组](https://leetcode.cn/problems/merge-sorted-array/)（简单）

给你两个按 **非递减顺序** 排列的整数数组 `nums1` 和 `nums2`，另有两个整数 `m` 和 `n` ，分别表示 `nums1` 和 `nums2` 中的元素数目。

请你 **合并** `nums2` 到 `nums1` 中，使合并后的数组同样按 **非递减顺序** 排列。

**注意：**最终，合并后数组不应由函数返回，而是存储在数组 `nums1` 中。为了应对这种情况，`nums1` 的初始长度为 `m + n`，其中前 `m` 个元素表示应合并的元素，后 `n` 个元素为 `0` ，应忽略。`nums2` 的长度为 `n` 。

```python
class Solution:
    def merge(self, nums1: List[int], m: int, nums2: List[int], n: int) -> None:
        """
        Do not return anything, modify nums1 in-place instead.
        """
        res = []
        p1 = 0
        p2 = 0
        while p1 < m or p2 < n:
            if p1 == m:
                res.append(nums2[p2])
                p2 += 1
            elif p2 == n:
                res.append(nums1[p1])
                p1 += 1
            elif nums1[p1] < nums2[p2]:
                res.append(nums1[p1])
                p1 += 1
            else:
                res.append(nums2[p2])
                p2 += 1
        
        nums1[:] = res
```

解法1：双指针

面试中使用了额外的数组空间

时间复杂度：O(m+n)。
指针移动单调递增，最多移动 m+n 次，因此时间复杂度为 O(m+n)。

空间复杂度：O(m+n)。
需要建立长度为 m+n 的中间数组res。



```python
# 指针 p1 指向 nums1 有效元素的最后一位
        p1 = m - 1
        # 指针 p2 指向 nums2 有效元素的最后一位
        p2 = n - 1
        # tail 指向合并后 nums1 总长度的最后一位（从后往前放数据）
        tail = m + n - 1
        
        # 只要还有元素没处理完就继续循环
        while p2 >= 0 or p1 >= 0:
            if p1 == -1:
                # nums1 已经全部处理完，直接把 nums2 剩下的元素放进去
                nums1[tail] = nums2[p2]
                p2 -= 1
            elif p2 == -1:
                # nums2 已经全部处理完，直接把 nums1 剩下的元素放进去
                nums1[tail] = nums1[p1]
                p1 -= 1
            elif nums1[p1] < nums2[p2]:
                # 谁大就把谁放在当前 tail 位置（从大到小放）
                nums1[tail] = nums2[p2]
                p2 -= 1
            else:
                # nums1 当前元素更大，放入当前位置
                nums1[tail] = nums1[p1]
                p1 -= 1
            
            # 放完一个元素，指针向前移动一位
            tail -= 1
```

解法二：双指针+倒序

那么如何直接避免覆盖 nums 1中的元素呢？观察可知，nums 1 的后半部分是空的，可以直接覆盖而不会影响结果。因此可以指针设置为从后向前遍历，每次取两者之中的较大者放进 nums 1的最后面。

时间复杂度：O(m+n)。
指针移动单调递减，最多移动 m+n 次，因此时间复杂度为 O(m+n)。

空间复杂度：O(1)。
直接对数组 nums 1原地修改，不需要额外空间。



### [189. 轮转数组](https://leetcode.cn/problems/rotate-array/)🔥（中等）

```python
class Solution(object):
    def rotate(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: None Do not return anything, modify nums in-place instead.
        """
        k = k % len(nums)  
        nums[:] = nums[-k:] + nums[:-k]
```

这样会使用额外的空间

```python
class Solution(object):
    def reverse(self, nums, i, j):
        # i从数组开头向后遍历，j从数组结尾向前遍历
        while i < j:
            nums[i], nums[j] = nums[j], nums[i]
            i += 1
            j -= 1

    def rotate(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: None Do not return anything, modify nums in-place instead.
        """
        k = k % len(nums)
        n = len(nums)
        self.reverse(nums, 0, n - 1)
        self.reverse(nums, 0, k - 1)
        self.reverse(nums, k, n - 1)
```

- 时间复杂度：O(*n*)，其中 *n* 是 *nums* 的长度。
- 空间复杂度：O(1)。

1 2 3 4 5 6 7 k=3

7 6 5 4 3 2 1 先反转整个数组

5 6 7 4 3 2 1再反转前k个

5 6 7 1 2 3 4反转后n-k个

题解详见https://leetcode.cn/problems/rotate-array/solutions/2784427/tu-jie-yuan-di-zuo-fa-yi-tu-miao-dong-py-ryfv



### [238. 除了自身以外数组的乘积](https://leetcode.cn/problems/product-of-array-except-self/)🔥（中等）

**题目要求不要使用除法，且在 `O(n)` 时间复杂度内完成此题**

```python
class Solution(object):
    def productExceptSelf(self, nums):
        """
        :type nums: List[int]
        :rtype: List[int]
        """
        n = len(nums)
        left = [1] * n
        right = [1] * n
        result = [1] * n

        # 求元素i的前缀
        for i in range(1, n):
            left[i] = left[i - 1] * nums[i - 1]

        # 求元素i的后缀
        for i in range(n - 2, -1, -1):
            right[i] = right[i + 1] * nums[i + 1]

        for i in range(n):
            result[i] = left[i] * right[i]
        
        return result
```



### [41. 缺失的第一个正数](https://leetcode.cn/problems/first-missing-positive/)🔥（困难）

**请你实现时间复杂度为 `O(n)` 并且只使用常数级别额外空间的解决方案。**

```python
class Solution(object):
    def firstMissingPositive(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        n = len(nums)
        for i in range(n):
            while 1 <= nums[i] <= n and nums[i] != nums[nums[i] - 1]:
                # 原地哈希交换：必须先换目标位置，再换 i 位置！
                # 如果写成 nums[i], nums[nums[i] - 1] = nums[nums[i] - 1], nums[i]会超时
                nums[nums[i] - 1], nums[i] = nums[i], nums[nums[i] - 1]
        
        for i in range(n):
            if nums[i] != i + 1:
                return i + 1
        
        return n + 1
```



## 矩阵

### [73. 矩阵置零](https://leetcode.cn/problems/set-matrix-zeroes/)🔥（中等）

```python
class Solution(object):
    def setZeroes(self, matrix):
        """
        :type matrix: List[List[int]]
        :rtype: None Do not return anything, modify matrix in-place instead.
        """
        m = len(matrix)
        n = len(matrix[0])
        row = [False] * m
        col = [False] * n

        for i in range(m):
            for j in range(n):
                if matrix[i][j] == 0:
                    row[i] = True
                    col[j] = True
        
        for i in range(m):
            for j in range(n):
                if row[i] or col[j]:
                    matrix[i][j] = 0
```



### [48. 旋转图像](https://leetcode.cn/problems/rotate-image/)🔥（中等）

你必须在**[ 原地](https://baike.baidu.com/item/原地算法)** 旋转图像，这意味着你需要直接修改输入的二维矩阵。**请不要** 使用另一个矩阵来旋转图像。

```python
class Solution(object):
    def rotate(self, matrix):
        """
        :type matrix: List[List[int]]
        :rtype: None Do not return anything, modify matrix in-place instead.
        """
        n = len(matrix)
        # 水平翻转
        for i in range(n // 2):
            for j in range(n):
                matrix[i][j], matrix[n - i - 1][j] = matrix[n - i - 1][j], matrix[i][j]

        # 对角线翻转
        for i in range(n):
            for j in range(i):
                matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
```

1.先水平翻转
$$
\begin{bmatrix}
5 & 1 & 9 & 11 \\
2 & 4 & 8 & 10 \\
13 & 3 & 6 & 7 \\
15 & 14 & 12 & 16
\end{bmatrix}
\xrightarrow{\text{水平翻转}}
\begin{bmatrix}
15 & 14 & 12 & 16 \\
13 & 3 & 6 & 7 \\
2 & 4 & 8 & 10 \\
5 & 1 & 9 & 11
\end{bmatrix}
$$
2.再对角线翻转
$$
\begin{bmatrix}
15 & 14 & 12 & 16 \\
13 & 3 & 6 & 7 \\
2 & 4 & 8 & 10 \\
5 & 1 & 9 & 11
\end{bmatrix}
\xrightarrow{\text{主对角线翻转}}
\begin{bmatrix}
15 & 13 & 2 & 5 \\
14 & 3 & 4 & 1 \\
12 & 6 & 8 & 9 \\
16 & 7 & 10 & 11
\end{bmatrix}
$$


### [54. 螺旋矩阵](https://leetcode.cn/problems/spiral-matrix/)🔥（中等）

```python
class Solution(object):
    def spiralOrder(self, matrix):
        """
        :type matrix: List[List[int]]
        :rtype: List[int]
        """
        m = len(matrix)
        n = len(matrix[0])
        left, right, up, down = 0, n - 1, 0, m - 1
        res = []

        while len(res) < m * n:
            # 向右走
            for i in range(left, right + 1):
                if len(res) == m * n:
                    return res
                res.append(matrix[up][i]) 
            up += 1

            # 向下走
            for i in range(up, down + 1):
                if len(res) == m * n:
                    return res
                res.append(matrix[i][right]) 
            right -= 1

             # 向左走
            for i in range(right, left - 1, -1):
                if len(res) == m * n:
                    return res
                res.append(matrix[down][i]) 
            down -= 1

            
             # 向上走
            for i in range(down, up - 1, -1):
                if len(res) == m * n:
                    return res
                res.append(matrix[i][left]) 
            left += 1

        return res
```

依次按顺序模拟右下左上



## 动态规划

### [70. 爬楼梯](https://leetcode.cn/problems/climbing-stairs/)🔥（简单）

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



### [118. 杨辉三角](https://leetcode.cn/problems/pascals-triangle/)🔥（简单）

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



### [198. 打家劫舍](https://leetcode.cn/problems/house-robber/)🔥（中等）

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

### [416. 分割等和子集](https://leetcode.cn/problems/partition-equal-subset-sum/)🔥（中等）

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





### [322. 零钱兑换](https://leetcode.cn/problems/coin-change/)🔥（中等）

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



### [279. 完全平方数](https://leetcode.cn/problems/perfect-squares/)🔥（中等）

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



### [139. 单词拆分](https://leetcode.cn/problems/word-break/)🔥（中等）

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

```python
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        wordSet = set(wordDict)
        return self.backtracking(s, wordSet, 0)

    def backtracking(self, s, wordSet, start_index):
        if start_index == len(s):
            return True
        
        for i in range(start_index, len(s)):
            word = s[start_index : i + 1]
            if word in wordSet and self.backtracking(s, wordSet, i + 1):
                return True
        
        return False
```

如果用回溯算法，不进行记忆化搜索，会超时



### [300. 最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/)🔥（中等）

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



### [121. 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)🔥（简单）

```python
class Solution(object):
    def maxProfit(self, prices):
        """
        :type prices: List[int]
        :rtype: int
        """
        length = len(prices)
        if length == 0:
            return 0
        dp = [[0] * 2 for _ in range(length)]
        dp[0][0] = -prices[0]
        dp[0][1] = 0
        for i in range(1, length):
            # 前1天就持有股票，保持；之前不持有股票，第i天购入股票
            dp[i][0] = max(dp[i-1][0], -prices[i])
            # 前1天就不持有股票，保持；之前持有股票，第i天卖出股票
            dp[i][1] = max(dp[i-1][1], prices[i] + dp[i-1][0])
        return dp[-1][1]
```

dp[i] [0]第i天持有股票最大值

dp[i] [1]第i天不持有股票最大值



### [53. 最大子数组和](https://leetcode.cn/problems/maximum-subarray/)🔥（中等）

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



### [152. 乘积最大子数组](https://leetcode.cn/problems/maximum-product-subarray/)🔥（中等）

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



## 多维动态规划

### [62. 不同路径](https://leetcode.cn/problems/unique-paths/)🔥（中等）

```python
class Solution(object):
    def uniquePaths(self, m, n):
        """
        :type m: int
        :type n: int
        :rtype: int
        """
        dp = [[0] * n for i in range(m)]

        # 初始化dp数组，让第一列和第一行的路径数均为1
        for i in range(m):
            dp[i][0] =  1
        for j in range(n):
            dp[0][j] = 1

        for i in range(1, m):
            for j in range(1, n):
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1]
        
        return dp[m - 1][n - 1]
```

dp【i】【j】 ：表示从（0 ，0）出发，到(i, j) 有dp【i】【j】条不同的路径。



### [63. 不同路径 II](https://leetcode.cn/problems/unique-paths-ii/)（中等）

```python
class Solution(object):
    def uniquePathsWithObstacles(self, obstacleGrid):
        """
        :type obstacleGrid: List[List[int]]
        :rtype: int
        """
        m = len(obstacleGrid)
        n = len(obstacleGrid[0])

        dp = [[0] * n for i in range(m)]

        if obstacleGrid[0][0] == 1 or obstacleGrid[m - 1][n - 1] == 1:
            return 0
        
        for i in range(m):
            if obstacleGrid[i][0] == 0:
                dp[i][0] = 1
            else:
                break
        
        for j in range(n):
            if obstacleGrid[0][j] == 0:
                dp[0][j] = 1
            else:
                break
        
        for i in range(1, m):
            for j in range(1, n):
                if obstacleGrid[i][j] == 1:
                    continue
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1]

        return dp[m - 1][n - 1] 
```

dp【i】【j】 ：表示从（0 ，0）出发，到(i, j) 有dp【i】【j】条不同的路径。



### [64. 最小路径和](https://leetcode.cn/problems/minimum-path-sum/)🔥（中等）

```python
class Solution(object):
    def minPathSum(self, grid):
        """
        :type grid: List[List[int]]
        :rtype: int
        """
        m = len(grid)
        n = len(grid[0])

        dp = [[0] * n for i in range(m)]

        temp_m = 0
        for i in range(m):
            temp_m += grid[i][0]
            dp[i][0] = temp_m 
        
        temp_n = 0
        for j in range(n):
            temp_n += grid[0][j]
            dp[0][j] = temp_n

        for i in range(1, m):
            for j in range(1, n): 
                dp[i][j] = min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j]
        
        return dp[m - 1][n - 1]
```

dp【i】【j】表示从（0 ，0）出发，到坐标（i，j）路径上的数字总和的最小值为dp【i】【j】



### [647. 回文子串](https://leetcode.cn/problems/palindromic-substrings/)（中等）

4.09

```python
class Solution(object):
    def countSubstrings(self, s):
        """
        :type s: str
        :rtype: int
        """
        dp = [[False] * len(s) for _ in range(len(s))]
        result = 0

        for i in range(len(s)-1, -1, -1): #注意遍历顺序
            for j in range(i, len(s)):
                if s[i] == s[j]:
                    if j - i <= 1: #情况一 和 情况二
                        result += 1
                        dp[i][j] = True
                    elif dp[i+1][j-1]: #情况三
                        result += 1
                        dp[i][j] = True
                        
        return result
```

dp【i】【j】：表示区间范围[i,j] （注意是左闭右闭）的子串是否是回文子串，如果是dp【i】【j】为true，否则为false

- 情况一：下标i 与 j相同，同一个字符例如a，当然是回文子串
- 情况二：下标i 与 j相差为1，例如aa，也是回文子串
- 情况三：下标：i 与 j相差大于1的时候，例如cabac，此时s[i]与s[j]已经相同了，我们看i到j区间是不是回文子串就看aba是不是回文就可以了，那么aba的区间就是 i+1 与 j-1区间，这个区间是不是回文就看dp[i + 1][j - 1]是否为true。



### [5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)🔥（中等）

4.09

```python
class Solution(object):
    def longestPalindrome(self, s):
        """
        :type s: str
        :rtype: str
        """
        dp = [[False] * len(s) for _ in range(len(s))]
        result = ""
        index_i = 0
        index_j = 0

        for i in range(len(s)-1, -1, -1): #注意遍历顺序
            for j in range(i, len(s)):
                if s[i] == s[j]:
                    if j - i <= 1: #情况一 和 情况二
                        dp[i][j] = True
                        if j - i > index_j - index_i:
                            index_j = j
                            index_i = i
                    elif dp[i + 1][j - 1]: #情况三
                        dp[i][j] = True
                        if j - i > index_j - index_i:
                            index_j = j
                            index_i = i
        
        for i in range(index_i, index_j + 1):
            result += s[i]
                        
        return result
```

dp【i】【j】：表示区间范围[i,j] （注意是左闭右闭）的子串是否是回文子串，如果是dp【i】【j】为true，否则为false



### [72. 编辑距离](https://leetcode.cn/problems/edit-distance/)🔥（中等）

4.09

```python
class Solution(object):
    def minDistance(self, word1, word2):
        """
        :type word1: str
        :type word2: str
        :rtype: int
        """
        # 创建 DP 表
        # dp[i][j] 表示：把 word1 的前 i 个字符 → 变成 word2 的前 j 个字符，最少需要几步
        # +1 是为了预留 空字符串 的位置（第0行、第0列）
        dp = [[0] * (len(word2) + 1) for _ in range(len(word1) + 1)]

        # 初始化第一列：word2 为空，把 word1 前 i 个字符删光 → 需要 i 步
        for i in range(len(word1) + 1):
            dp[i][0] = i

        # 初始化第一行：word1 为空，变成 word2 前 j 个字符 → 需要加 j 个字符
        for j in range(len(word2) + 1):
            dp[0][j] = j

        # 开始填 DP 表
        for i in range(1, len(word1) + 1):
            for j in range(1, len(word2) + 1):
                # 如果当前两个字符相等 → 不用操作，直接继承左上角的值
                if word1[i - 1] == word2[j - 1]:
                    dp[i][j] = dp[i - 1][j - 1]
                # 如果字符不相等 → 三种操作选最小的：
                # 1. dp[i-1][j]+1   删除 word1 的第 i 个字符
                # 2. dp[i][j-1]+1   在 word1 中插入字符
                # 3. dp[i-1][j-1]+1 替换字符
                else:
                    dp[i][j] = min(dp[i - 1][j] + 1, dp[i][j - 1] + 1, dp[i - 1][j - 1] + 1)
        
        # 右下角就是最终答案：把整个 word1 → word2 的最小步数
        return dp[-1][-1]
```

dp【i】【j】:表示以下标i-1为结尾的字符串word1，和以下标为j-1为结尾的字符串word2，最近的编辑距离为dp【i】【j】。

len（word1）+1 和 len（word2）+ 1是为了第一行第一列的空字符串留空间

第 0 行（空字符串）

第 0 列（空字符串）



### [1143. 最长公共子序列](https://leetcode.cn/problems/longest-common-subsequence/)🔥（中等）

4.09

```python
class Solution(object):
    def longestCommonSubsequence(self, text1, text2):
        """
        :type text1: str
        :type text2: str
        :rtype: int
        """
    # 创建 DP 表：(len(text1)+1)行 × (len(text2)+1)列
    # dp[i][j] 表示：text1前i个字符 和 text2前j个字符 的最长公共子序列长度
    # +1 是为了包含 空字符串 的情况（第0行、第0列）
    dp = [[0] * (len(text2) + 1) for _ in range(len(text1) + 1)]
	
    for i in range(len(text1) + 1):
        dp[i][0] = 0
        
    for j in range(len(text2) + 1):
        dp[0][j] = 0
        
    # 遍历填充 DP 表
    for i in range(1, len(text1) + 1):
        for j in range(1, len(text2) + 1):
            # 如果当前字符相等（注意字符串从0开始，所以用i-1/j-1）
            if text1[i - 1] == text2[j - 1]:
                # 相等 → 公共长度 = 左上角值 + 1（两个字符串同时往前挪一位）
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:
                # 不相等 → 取两种情况最大值：
                # 1. 舍弃 text1 当前字符：dp[i-1][j]
                # 2. 舍弃 text2 当前字符：dp[i][j-1]
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    
    # 右下角的值就是整个 text1 和 text2 的最长公共子序列长度
    return dp[-1][-1]
```

有点类似72. 编辑距离

dp【i】【j】：长度为[0, i - 1]的字符串text1与长度为[0, j - 1]的字符串text2的最长公共子序列长度为dp【i】【j】



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



### [19. 删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)🔥（中等）

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



### [21. 合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)🔥（简单）

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



### [23. 合并 K 个升序链表](https://leetcode.cn/problems/merge-k-sorted-lists/)🔥（困难）

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution(object):
    def mergeKLists(self, lists):
        """
        :type lists: List[Optional[ListNode]]
        :rtype: Optional[ListNode]
        """
        if not lists:
            return None

        res = lists[0]
        for i in range(1, len(lists)):
            res = self.mergeTwoLists(res, lists[i])
        return res
    
    def mergeTwoLists(self, list1, list2):
        if list1 is None:
            return list2
        elif list2 is None:
            return list1
        elif list1.val < list2.val:
            list1.next = self.mergeTwoLists(list1.next, list2)
            return list1
        else:
            list2.next = self.mergeTwoLists(list2.next, list1)
            return list2 
```

用合并两个有序链表的思路写会超时

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution(object):
    def mergeKLists(self, lists):
        """
        :type lists: List[Optional[ListNode]]
        :rtype: Optional[ListNode]
        """
        dummy = ListNode(0)
        current = dummy
        heap = []

        for index, node in enumerate(lists):
            if node:
                heapq.heappush(heap, (node.val, index, node))

        while heap:
            val, i, node = heapq.heappop(heap)
            current.next = node
            current = current.next
            if node.next:
                heapq.heappush(heap, (node.next.val, index, node.next))

        return dummy.next
```

维护一个最小堆



### [160. 相交链表🔥](https://leetcode.cn/problems/intersection-of-two-linked-lists/)（简单）

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



### [206. 反转链表](https://leetcode.cn/problems/reverse-linked-list/)🔥（简单）

4.09

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

注意：最后要return pre，因为最后cur已经指向temp，为None



### [24. 两两交换链表中的节点](https://leetcode.cn/problems/swap-nodes-in-pairs/)🔥（中等）

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



### [25. K 个一组翻转链表](https://leetcode.cn/problems/reverse-nodes-in-k-group/)🔥（困难）

```

```

[Reverse Nodes in K-Group 可视化](https://jiaxuan-yue.github.io/projects/reverse-k-group/)



### [234. 回文链表](https://leetcode.cn/problems/palindrome-linked-list/)🔥（简单）

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



### [141. 环形链表](https://leetcode.cn/problems/linked-list-cycle/)🔥（简单）

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



### [142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/)🔥（中等）

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



### [2. 两数相加](https://leetcode.cn/problems/add-two-numbers/)🔥（中等）

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



### [138. 随机链表的复制](https://leetcode.cn/problems/copy-list-with-random-pointer/)🔥（中等）

```python
"""
# Definition for a Node.
class Node:
    def __init__(self, x, next=None, random=None):
        self.val = int(x)
        self.next = next
        self.random = random
"""

class Solution(object):
    def copyRandomList(self, head):
        """
        :type head: Node
        :rtype: Node
        """
        # 原节点 -> 新节点 的映射字典
        old_to_new = {}
        
        # 第一步：创建所有新节点，建立映射
        current = head
        while current:
            new_node = Node(current.val)
            old_to_new[current] = new_node
            current = current.next

        # 第二步：设置新节点的 next 和 random 指针
        current = head
        while current:
            new_node = old_to_new[current]
            new_node.next = old_to_new.get(current.next)
            new_node.random = old_to_new.get(current.random)
            current = current.next

        # 返回新链表的头节点
        return old_to_new.get(head)
```

先把所有节点复制一遍存起来，再回头统一连指针

`old_to_new` 是一个**Python 字典**，`get()` 是字典自带的**安全取值函数**。



### [148. 排序链表](https://leetcode.cn/problems/sort-list/)🔥（中等）

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution(object):
    def sortList(self, head):
        """
        :type head: Optional[ListNode]
        :rtype: Optional[ListNode]
        """
         # 递归终止条件：空链表或只有一个节点，本身已有序
        if head is None or head.next is None:
            return head
        
        # 快慢指针找中点（slow最终指向链表前半段的最后一个节点）
        slow = head
        fast = head.next
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
        
        # 分割链表为两部分
        first = head
        second = slow.next
        slow.next = None

        # 递归排序左右两部分
        first = self.sortList(first)
        second = self.sortList(second)

        return self.mergeTwoLists(first, second)
    
    
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

**时间复杂度**：O (n log n)，归并排序的标准时间复杂度。

合并两个有序链表和[21. 合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)🔥（简单）一致



### [146. LRU 缓存](https://leetcode.cn/problems/lru-cache/)🔥（中等）

```python
class ListNode:
    def __init__(self, key = 0, val=0):
        # 因为要满足get和put函数时间复杂度为O(1)所以使用双向链表
        self.key = key
        self.val = val
        self.next = None
        self.prev = None

class LRUCache(object):

    def __init__(self, capacity):
        """
        :type capacity: int
        """
        self.capacity = capacity
        # 哈希表：key -> 双向链表节点，O(1) 查找
        self.cache = {}
        self.head = ListNode()
        self.tail = ListNode()
        self.head.next = self.tail
        self.tail.prev = self.head
    
    def remove(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev
    
    def add_to_head(self, node):
        node.next = self.head.next
        node.prev = self.head
        self.head.next.prev = node
        self.head.next = node

    def get(self, key):
        """
        :type key: int
        :rtype: int
        """
        if key not in self.cache:
            return -1
        
        # 从哈希表拿到对应的节点
        node = self.cache[key]
        # 先把节点从原来位置删除
        self.remove(node)
        # 再把节点移到头部（标记为最近使用）
        self.add_to_head(node)
        return node.val

    def put(self, key, value):
        """
        :type key: int
        :type value: int
        :rtype: None
        """
        if key in self.cache:
            self.remove(self.cache[key])
            self.cache.pop(key)
        node = ListNode(key, value)
        self.cache[key] = node
        self.add_to_head(node)

        # 如果缓存超过最大容量，删除最久未使用的节点
        if len(self.cache) > self.capacity:
            last_use = self.tail.prev
            self.remove(last_use)
            self.cache.pop(last_use.key)

# Your LRUCache object will be instantiated and called as such:
# obj = LRUCache(capacity)
# param_1 = obj.get(key)
# obj.put(key,value)
```

双向链表，面试常考，建议直接背诵



## 二叉树DFS

前序遍历：中左右

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



### [94. 二叉树的中序遍历](https://leetcode.cn/problems/binary-tree-inorder-traversal/)🔥（简单）

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

中序遍历：左中右



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

后序遍历：左右中



### 知识：二叉搜索树BST

**有效** 二叉搜索树定义如下：

- 节点的左子树只包含 **严格小于** 当前节点的数。
- 节点的右子树只包含 **严格大于** 当前节点的数。
- 所有左子树和右子树自身必须也是二叉搜索树。
- 二叉搜索树的中序遍历是**严格递增**序列



### [98. 验证二叉搜索树](https://leetcode.cn/problems/validate-binary-search-tree/)🔥（中等）

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def isValidBST(self, root):
        """
        :type root: Optional[TreeNode]
        :rtype: bool
        """
        def dfs(node, vec):
            if node is None:
                return
            # 中序遍历，将二叉搜索树转换为有序数组
            dfs(node.left, self.vec)
            self.vec.append(node.val)
            dfs(node.right, self.vec)

        self.vec = []
        dfs(root, self.vec)
        for i in range(0, len(self.vec) - 1):
            #注意要大于等于，搜索树里不能有相同元素
            if self.vec[i] >= self.vec[i + 1]:
                return False
            
        return True
```

DFS，二叉搜索树（BST）的中序遍历结果一定是「严格递增数组」



### [108. 将有序数组转换为二叉搜索树](https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree/)🔥（简单）

4.09 4.09

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def sortedArrayToBST(self, nums):
        """
        :type nums: List[int]
        :rtype: Optional[TreeNode]
        """
        def dfs(nums, left, right):
            # 定义函数在nums上[left, right]左闭右闭区间内寻找根节点
            if left > right:
                return
            
            # 核心永远选区间中间的数作为根节点，递归构建左右子树
            mid = left + (right - left) // 2
            root = TreeNode(nums[mid])
            root.left = dfs(nums, left, mid - 1)
            root.right = dfs(nums, mid + 1, right)
            return root

        root = dfs(nums, 0, len(nums) - 1)
        return root
```

left > right` = 区间为空 = 没有节点可建 = 返回 `

DFS，构建二叉搜索树永远选区间中间的数作为根节点，递归构建左右子树



### [230. 二叉搜索树中第 K 小的元素](https://leetcode.cn/problems/kth-smallest-element-in-a-bst/)🔥（中等）

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def kthSmallest(self, root, k):
        """
        :type root: Optional[TreeNode]
        :type k: int
        :rtype: int
        """
        self.res = 0
        self.count = 0

        def dfs(node):
            if node is None:
                return
            
            dfs(node.left)
            self.count += 1
            if self.count == k:
                self.res = node.val
                return
            dfs(node.right)
        
        dfs(root)
        return self.res
```

DFS，二叉搜索树的中序遍历为严格递增序列



### [114. 二叉树展开为链表](https://leetcode.cn/problems/flatten-binary-tree-to-linked-list/)🔥（中等）

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



### [543. 二叉树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/)🔥（简单）

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def diameterOfBinaryTree(self, root):
        """
        :type root: Optional[TreeNode]
        :rtype: int
        """

        def dfs(node, res, height):
            if node is None:
                return 0
            left_height = 0
            L_depth = dfs(node.left, res, left_height)
            right_height = 0
            R_depth = dfs(node.right, res, right_height)
            self.res = max(self.res, L_depth + R_depth)
            # 递归返回给父节点时，必须把自己算进去，父节点才能算出正确高度
            return max(L_depth, R_depth) + 1
        
        self.res = 0
        height = 0
        dfs(root, self.res, height)
        return self.res
```

任意节点的【左子树高度 + 右子树高度】的最大值



### [236. 二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/)🔥（中等）

最近公共祖先的定义为：“对于有根树 T 的两个节点 p、q，最近公共祖先表示为一个节点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（**一个节点也可以是它自己的祖先**）。”

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def lowestCommonAncestor(self, root, p, q):
        """
        :type root: TreeNode
        :type p: TreeNode
        :type q: TreeNode
        :rtype: TreeNode
        """
        # 递归终止条件：
        # 1. 走到空节点，返回 None
        # 2. 当前节点就是 p 或 q，直接返回当前节点（找到了）
        if root is None or root == p or root == q:
            return root

        # 递归去左子树找 p 和 q
        left = self.lowestCommonAncestor(root.left, p, q)
        # 递归去右子树找 p 和 q
        right = self.lowestCommonAncestor(root.right, p, q)

        # 情况1：左边没找到，右边找到了 → 答案就是右边
        if left is None and right is not None:
            return right
        # 情况2：右边没找到，左边找到了 → 答案就是左边
        elif left is not None and right is None:
            return left
        # 情况3：左右都没找到 → 返回空
        elif left is None and right is None:
            return None
        # 情况4：左右两边都找到了 → 当前节点就是最近公共祖先！
        elif left is not None and right is not None:
            return root
```

DFS，采用后序遍历（左→右→根），自底向上找最近公共祖先：

如果一个节点的左边找到 p、右边找到 q，那它就是答案！



### [105. 从前序与中序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)🔥（中等）

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def buildTree(self, preorder, inorder):
        """
        :type preorder: List[int]
        :type inorder: List[int]
        :rtype: Optional[TreeNode]
        """
        if len(preorder) == 0 and len(inorder) == 0:
            return None
        
        # 前序遍历的第一个节点就是根节点
        root = TreeNode(preorder[0])

        # 寻找在中序遍历inorder中root的index
        index = 0
        for i in range(len(inorder)):
            if inorder[i] == preorder[0]:
                index = i
                break
        
        preorder_left = preorder[1:1 + index]
        inorder_left = inorder[:index]
        root.left = self.buildTree(preorder_left, inorder_left)
        preorder_right = preorder[index + 1:]
        inorder_right = inorder[index + 1:]
        root.right = self.buildTree(preorder_right, inorder_right)

        return root
```

利用前序和中序遍历的特性，递归地把数组切成「根、左子树、右子树」三部分，不断重复直到树构建完成。



### [106. 从中序与后序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)（中等）

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def buildTree(self, inorder, postorder):
        """
        :type inorder: List[int]
        :type postorder: List[int]
        :rtype: Optional[TreeNode]
        """
        if len(inorder) == 0 and len(postorder) == 0:
            return None
        
        # 后序遍历的最后一个节点是根节点
        root = TreeNode(postorder[-1])
        
        index = 0
        # 在中序遍历序列中找到根节点对应的index
        for i in range(len(inorder)):
            if inorder[i] == postorder[-1]:
                index = i
                break
        
        inorder_left = inorder[:index]
        postorder_left = postorder[:index]
        root.left = self.buildTree(inorder_left, postorder_left)
        inorder_right = inorder[index + 1:]
        postorder_right = postorder[index:-1]
        root.right = self.buildTree(inorder_right, postorder_right)

        return root
```



### [112. 路径总和](https://leetcode.cn/problems/path-sum/)（简单）

返回是否有  **从根节点到叶子节点** 路径总和等于给定目标和

```
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def hasPathSum(self, root, targetSum):
        """
        :type root: Optional[TreeNode]
        :type targetSum: int
        :rtype: bool
        """
        if root is None:
            return False

        return self.dfs(root, targetSum - root.val)
    
    def dfs(self, current, target):
        if current.left is None and current.right is None and target == 0:
            return True
        
        if current.left is None and current.right is None and target != 0:
            return False

        if current.left:
            target -= current.left.val
            if self.dfs(current.left, target) == True:
                return True
            target += current.left.val
        
        if current.right:
            target -= current.right.val
            if self.dfs(current.right, target) == True:
                return True
            target += current.right.val
        
        return False
```



### [113. 路径总和 II](https://leetcode.cn/problems/path-sum-ii/)（中等）

找出所有 **从根节点到叶子节点** 路径总和等于给定目标和的路径

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def pathSum(self, root, targetSum):
        """
        :type root: Optional[TreeNode]
        :type targetSum: int
        :rtype: List[List[int]]
        """
        self.res = []
        self.path = []

        if root is None:
            return []

        self.dfs(root, targetSum - root.val)

        return self.res
        
    def dfs(self, current, target):
        self.path.append(current.val)

        if current.left is None and current.right is None and target == 0:
            self.res.append(self.path[:])
            return
        
        if current.left is None and current.right is None and target != 0:
            return

        if current.left:
            target -= current.left.val
            self.dfs(current.left, target)
            target += current.left.val
            self.path.pop()
        
        if current.right:
            target -= current.right.val
            self.dfs(current.right, target)
            target += current.right.val
            self.path.pop()
        
        return
```



### [437. 路径总和 III](https://leetcode.cn/problems/path-sum-iii/)🔥（中等）

求该二叉树里节点值之和等于 `targetSum` 的 **路径** 的数目，注意：此题不一定是根节点到子节点

```

```



## 二叉树BFS

### [102. 二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/)🔥（中等）

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

BFS



### [104. 二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/)🔥（简单）

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



### [226. 翻转二叉树](https://leetcode.cn/problems/invert-binary-tree/)🔥（简单）

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



### [101. 对称二叉树](https://leetcode.cn/problems/symmetric-tree/)🔥（简单）

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



### [199. 二叉树的右视图](https://leetcode.cn/problems/binary-tree-right-side-view/)🔥（中等）

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def rightSideView(self, root):
        """
        :type root: Optional[TreeNode]
        :rtype: List[int]
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
            res.append(level[-1])
        
        return res
```

BFS，和[102. 二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/)几乎完全一致，res中保存level的最后一个元素即可



## 图论

### [200. 岛屿数量](https://leetcode.cn/problems/number-of-islands/)🔥（中等）

4.09

```python
class Solution(object):
    # 四个方向：上、右、下、左
    directions = [[0, 1], [1, 0], [0, -1], [-1, 0]]

    def numIslands(self, grid):
        """
        :type grid: List[List[str]]
        :rtype: int
        """
        res = 0
        m = len(grid)
        n = len(grid[0])
        visited = [[False] * n for _ in range(m)]

        for i in range(m):
            for j in range(n):
                # 判断：如果当前节点是陆地，res+1并标记访问该节点，使用深度搜索标记相邻陆地。
                if grid[i][j] == "1" and visited[i][j] == False:
                    res += 1
                    self.dfs(visited, grid, i, j)

        return res
    
    def dfs(self, visited, grid, x, y):
        # x判断边界条件是和行数比较，y判断边界条件是和列数比较
            if x < 0 or y < 0 or x >= len(grid) or y >= len(grid[0]):
                return
            # 注意这里是or判断
            if visited[x][y] == True or grid[x][y] == "0":
                return

            visited[x][y] = True

            for i, j in self.directions:
                next_x = x + i
                next_y = y + j
                self.dfs(visited, grid, next_x, next_y)
```



### [994. 腐烂的橘子](https://leetcode.cn/problems/rotting-oranges/)🔥（中等）

```python
class Solution(object):
    def orangesRotting(self, grid):
        """
        :type grid: List[List[int]]
        :rtype: int
        """
        queue = collections.deque()
        # queue = []
        time = 0
        fresh = 0
        for i in range(len(grid)):
            for j in range(len(grid[0])):
                if grid[i][j] == 1:
                    fresh += 1
                # 将腐烂橘子的坐标放到队列里
                elif grid[i][j] == 2:
                    queue.append((i, j))
        
        direction = [[0, 1], [1, 0], [0, -1], [-1, 0]]

        while queue and fresh > 0:
            for i in range(len(queue)):
                current = queue.popleft()
                # current = queue.pop(0)
                # 遍历腐烂橘子的四个方向
                for p, q in direction:
                    current_x = current[0] + p
                    current_y = current[1] + q

                    if current_x < 0 or current_x >= len(grid) or current_y < 0 or current_y >= len(grid[0]) or grid[current_x][current_y] != 1:
                        continue
                    # 将周围新鲜橘子腐烂，将坐标加入队列
                    grid[current_x][current_y] = 2
                    queue.append((current_x, current_y))
                    fresh -= 1
            time += 1
        
        if fresh != 0:
            return -1
        else:
            return time
```

BFS



### [207. 课程表](https://leetcode.cn/problems/course-schedule/)🔥（中等）

```python
class Solution(object):
    def canFinish(self, numCourses, prerequisites):
        """
        :type numCourses: int
        :type prerequisites: List[List[int]]
        :rtype: bool
        """
        inDegree = [0] * numCourses
        umap = collections.defaultdict(list)

        # info[1] = 先修课 info[0] = 后修课
        # umap[先修课].append(后修课)
        for info in prerequisites:
            umap[info[1]].append(info[0])
            inDegree[info[0]] += 1

        q = collections.deque([i for i in range(numCourses) if inDegree[i] == 0])
        visited = 0  # 记录已经学完的课程数量

        while q:
            current = q.popleft()   # 拿出一门可以学的课
            visited += 1            # 学完了！计数+1

            # 学完 current 课后，它的所有后续课 先修要求-1
            for i in umap[current]:
                inDegree[i] -= 1

                # 如果这门课 先修课都学完了（入度=0），可以学了
                if inDegree[i] == 0:
                    q.append(i)
        
        return visited == numCourses
```

BFS找有向图是否成环



### [208. 实现 Trie (前缀树)](https://leetcode.cn/problems/implement-trie-prefix-tree/)🔥（中等）

```python
class Trie(object):

    def __init__(self):
        self.children = [None] * 26
        self.isEnd = False
        

    def insert(self, word):
        """
        :type word: str
        :rtype: None
        """
        node = self
        for ch in word:
            ch = ord(ch) - ord("a")
            if not node.children[ch]:
                node.children[ch] =  Trie()
            node = node.children[ch]
        node.isEnd = True

    def search(self, word):
        """
        :type word: str
        :rtype: bool
        """
        node = self
        for ch in word:
            ch = ord(ch) - ord("a")
            if not node.children[ch]:
                return False
            node = node.children[ch]
        if node.isEnd == True:
            return True
        else:
            return False

    def startsWith(self, prefix):
        """
        :type prefix: str
        :rtype: bool
        """
        node = self
        for ch in prefix:
            ch = ord(ch) - ord("a")
            if not node.children[ch]:
                return False
            node = node.children[ch]
        return True
        

# Your Trie object will be instantiated and called as such:
# obj = Trie()
# obj.insert(word)
# param_2 = obj.search(word)
# param_3 = obj.startsWith(prefix)
```

https://www.bilibili.com/video/BV1wsCJY6ESK/?spm_id_from=333.337.search-card.all.click



## 技巧

### [169. 多数元素](https://leetcode.cn/problems/majority-element/)🔥（简单）

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



### [31. 下一个排列](https://leetcode.cn/problems/next-permutation/)🔥（中等）

必须**[ 原地 ](https://baike.baidu.com/item/原地算法)**修改，只允许使用额外常数空间。

```python
class Solution(object):
    def nextPermutation(self, nums):
        """
        :type nums: List[int]
        :rtype: None Do not return anything, modify nums in-place instead.
        """
        # 第一步：从后往前找 第一个 变小的位置 i
        # 也就是找 nums[i] < nums[i+1]
        i = len(nums) - 2  # 从倒数第二个开始，避免 i+1 越界

        # 只要当前数 >= 后面的数，就继续往前找
        while i >= 0 and nums[i] >= nums[i + 1]:
            i -= 1
        
        # 第二步：如果找到了 i（不是完全降序）
        if i >= 0:
            # 从后往前找 第一个 比 nums[i] 大的数 j
            j = len(nums) - 1
            while j >= 0 and nums[i] >= nums[j]:
                j -= 1
            # 交换 i 和 j，让前面的数刚好变大一点点
            nums[i], nums[j] = nums[j], nums[i]

        # 第三步：把 i 后面的所有数 反转
        # 因为 i 后面一定是降序，反转后变成升序（最小）
        left = i + 1
        right = len(nums) - 1
        while left < right:
            # 交换左右指针，实现反转
            nums[left], nums[right] = nums[right], nums[left]
            left += 1
            right -= 1
```

难以理解，直接背诵
