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