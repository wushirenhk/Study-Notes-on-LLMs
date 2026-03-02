# Leetcode hot100

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

