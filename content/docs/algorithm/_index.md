---
weight: 7
bookFlatSection: true
title: "算法"
---

### 面试题
- 题目：
- []()
- 思路：
- 答案：
```java

```


// // /**
// //  * 反转一个 32bit 无符号 int
// //  * --
// //  * Example 1:
// //  * input : 0
// //  * output : 0
// //  *
// //  * 00000000000000000000000000000000
// //  * 00000000000000000000000000000000
// //  * --
// //  * Example 2:
// //  * input : 3
// //  * output : 3221225472
// //  *
// //  * 00000000000000000000000000010011
// //  * 11001000000000000000000000000000
// //  */
// // public long reverse(long input) {
// //   long res = 0;
// //   for (int i = 0; i < 32; i++) {
// //     res = res | (input & 1) << (31 - i);
// //     if (input == 0) {
// //       break;
// //     }
// //     input = input >> 1;
// //   }
// //   return res;
// // }

// // 我们的有一堆文件，现在需要你检查一下是否有文件名重复的文件。如果有返回 true 否则返回 false
// // Example 1:
// // Input: ["a.txt","b.txt","a.jpg","a.txt"]
// // Output: true
// // Example 2:
// // Input: ["a.txt","b.txt","c.txt","d.txt"]
// // Output: false

// public boolean checkDuplicate(String[] input) {
// 	// write your code here
//   Map<Integer, Integer> map = new HashMap<>();
//   for (int i = 0; i < input.length; i++) {
//     int hashCode = input[i].hashCode();
//     if (map.contains(hashCode)) {
//       int h = map.get(hashCode);
//       if (input[h] == input[i]) {
//         return true;
//       }
//     }
//     map.put(hashCode, i);
//   }
//   return false;
// }


/**
 * 给你一个链表，请删除倒数第 K 个节点并返回头节点。
 *
 * Example:
 * Input: Linked List: 1->2->3->4->5, K = 2.
 * Output: 1->2->3->5.
 *
 * 链表数据结构：
 * class ListNode {
 *     public int val;
 *     public ListNode next;
 *     ListNode(int x) { val = x; next = null; }
 * }
 */
public ListNode removeNthFromEnd(ListNode A, int K) {
	// write your code here

  ListNode p1 = A; // 1
  ListNode p2 = A; // 1
  for (int i = 0; i < K + 1; i ++) {
    if (p2 == null) {
      return A;
    }
    p2 = p2.next; // 2 3 4
  }
  if (p2 == null) {
    return A.next;
  }
  // p1 1 p 2
  while (p2 != null) {
    p1 = p1.next; // 2 3
    p2 = p2.next; // 5 null
  }
  p1.next = p1.next.next; //
  return A; //1235
}
