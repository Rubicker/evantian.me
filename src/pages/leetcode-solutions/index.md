---
title: Leetcode Solutions(Python & JavaScript version)
date: '2020-05-11'
spoiler: "The best preparation for tomorrow is to do your best today"
---

### Two Sum

[address](https://leetcode.com/problems/two-sum/)

> Given an array of integers, return indices of the two numbers such that they add up to a specific target.
>
> You may assume that each input would have exactly one solution, and you may not use the same element twice.

```py
class Solution:
  def twoSum(self, num: List[int], target: int) -> List[int]:
    dic = {}

    for i, num in enumerate(nums):
      n = target - num
      if n in h:
        return [dic[n], i]
      else:
        dic[num] = i

    return []
```

```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
const twoSum = (nums, target) => {
  let dic = new Map()

  for (let i = 0; i < nums.length; i++) {
    const n = target - nums[i]
    if (dic.has(n)) {
      return [dic.get(n), i]
    } else {
      dic.set(nums[i], i)
    }
  }

  return []
}
```

### Add Two Numbers

[source](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)

> You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.
>
> You may assume the two numbers do not contain any leading zero, except the number 0 itself.
> 
> Example:
> 
> Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
>
> Output: 7 -> 0 -> 8
>
> Explanation: 342 + 465 = 807.

```py
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
def addTwoNumbers(self, l1: ListNode, l2: ListNode) -> ListNode:
  carry = 0
  root = curr = ListNode(0)

  while l1 or l2 or carry:
    if l1: carry += l1.val; l1 = l1.next
    if l2: carry += l2.val; l2 = l2.next
    curr.next = curr = ListNode(carry % 10)
    carry //= 10

  return root.next
```

```js
/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
/**
 * @param {ListNode} l1
 * @param {ListNode} l2
 * @return {ListNode}
 */
const addTwoNumbers = (l1, l2) => {
  let carry = 0
  let root, current; 
  root = current = new ListNode(0)
  
  while (l1 || l2 || carry > 0) {
    if (l1) {
      carry += l1.val
      l1 = l1.next
    } 
    if (l2) {
      carry += l2.val
      l2 = l2.next
    }
    current.next = current = new ListNode(carry % 10)
    carry = parseInt(carry / 10)
  }
  
  return root.next
};
```



## Hashtable

### [1424. Diagonal Traverse II](https://leetcode.com/problems/diagonal-traverse-ii/)

```js
/*
 *  Algorithm:
 *  1. insert all diagonals elements into the same position of an array
 *      eg. nums[i][0], nums[i-1][1],...nums[0][i] into m[i]
 *  2. flatten m and output  
 *  Time complexity: O(n)
 *  Space Complexity: O(n)
 */
const findDiagonalOrder = nums => {  
  let m = []
  
  nums.forEach((row, i) => {
    row.forEach((num, j) => {
      if (i + j >= m.length) m.push([])
      m[i + j].unshift(num)
    })
  })
  
  return m.flat(1);
};
```

```py
class Solution:
  def findDiagonalOrder(self, nums: List[List[int]]) -> List[int]:
    m = []

    for i, row in enumerate(nums):
      for j, v in enumerate(row):
        if i + j >= len(m): m.append([])
        m[i + j].append(v)

    return [v for d in m for v in reversed(d)]
```

### [1371. Find the Longest Substring Containing Vowels in Even Counts](https://leetcode.com/problems/find-the-longest-substring-containing-vowels-in-even-counts/)

Use a hashtable to store the first index of a given state. If the same state occurs again(i -> j), which means we found a subarray (i + 1 -> j) that all vowels occur even times. Length = j - (i + 1) + 1 = j - i

```py
# prefix sum => prefix freq: 
# whether each vowel occurs evan(0) or odd(1) time(s)
# when a vowel occurs we just flip the bit
# e.g. 01110 = uoiea(o, i, e even times) => i => 01010(o, e even times)
#
# Time complexity: O(n)
# Space complexity: O(32) => 2**5

class Solution:
  def findTheLongestSubstring(self, s: str) -> int:
    idx = { 0 : -1 }
    vowels = 'aeiou'
    state = 0
    ans = 0
    
    for i in range(len(s)):
      j = vowels.find(s[i])
      if j >= 0: state ^= 1 << j
      if state not in idx:
        idx[state] = i
      ans = max(ans, i - idx[state])
    
    return ans
```

```js
/**
 * @param {string} s
 * @return {number}
 */
const findTheLongestSubstring = s =>  {
  let idx = { 0: -1 }
  const vowels = 'aeiou'
  let state = 0
  let max = 0
  
  for (let i = 0; i < s.length; i++) {
    let j = vowels.indexOf(s[i])
    if (j > -1) {
      state ^= 1 << j
    } 
    if (!(state in idx)) {
      idx[state] = i
    }
    max = Math.max(max, i - idx[state])
  }
  
  return max
};
```

### [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

```py
class Solution:
  def lengthOfLongestSubstring(self, s: str) -> int:
    used = {}
    max_length = start = 0
    for i, c in enumerate(s):
      if c in used and start <= used[c]:
        start = used[c] + 1
      else: max_length = max(max_length, i - start + 1)
        
      used[c] = i
      
    return max_length
```

> [How to Solve Sliding Window Problems](https://medium.com/outco/how-to-solve-sliding-window-problems-28d67601a66)