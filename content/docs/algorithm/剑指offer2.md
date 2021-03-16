---
title: '剑指offer - 高质量的代码'
date: 2021-02-28
weight: 2
categories: ["剑指offer","算法"]
---
# 剑指offer中的算法题
> [剑指Offer](/books/Offer.pdf)

## 高质量的代码

### 面试题11：数值的整次方
- 题目：实现 pow(x, n) ，即计算 x 的 n 次幂函数（即，xn）。不得使用库函数，同时不需要考虑大数问题。
- [https://leetcode-cn.com/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/](https://leetcode-cn.com/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/)
- 思路：
![kantu](/images/offer12.png)
- 答案：
```java
class Solution {
    public double myPow(double x, int n) {
        if (x == 0) {
            return 0;
        }
       long b = n;
       if (n < 0) {
           x = 1 / x;
           b = -b;
       }
       double r = 1.0;
       while (b > 0) {
           if ((b & 1) == 1) {
               r = r * x;
           }
           x *= x;
           b = b>>1;
       }
       return r;
    }
}
```

### 面试题12：打印1到最大的n位数
- 题目：输入数字 n，按顺序打印出从 1 到最大的 n 位十进制数。比如输入 3，则打印出 1、2、3 一直到最大的 3 位数 999。
- [https://leetcode-cn.com/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/](https://leetcode-cn.com/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/)
- 思路：考察在大数字的情况下如何输出。应该改用String，然后递归使用。
- 答案：
```java
class Solution {
    public int[] printNumbers(int n) {
        int end = (int)Math.pow(10, n) - 1;
        int[] res = new int[end];
        for(int i = 0; i < end; i++)
            res[i] = i + 1;
        return res;
    }
}

```

### 面试题13：删除链表结点
- 题目：给定单向链表的头指针和一个要删除的节点的值，定义一个函数删除该节点。返回删除后的链表的头节点。
- [https://leetcode-cn.com/problems/shan-chu-lian-biao-de-jie-dian-lcof/](https://leetcode-cn.com/problems/shan-chu-lian-biao-de-jie-dian-lcof/)
- 思路：顺着链表往下删除，找到要删除的节点把他的next 接过来，删除节点
- 答案：
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode deleteNode(ListNode head, int val) {
        ListNode p = head;
        int var = val;
        while (p.next != null) {
            if (p.val == var) {
                p.val = p.next.val;
                var = p.val;
            }
            if (p.next != null && p.next.next == null) {
                break;
            }
            p = p.next;
        }
        if (p.next != null && p.next.val == var) {
            p.next = null;
        }
        return head;
    }
}
```

### 面试题14：调整数组顺序使奇数位于偶数前面。
- 题目：输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有奇数位于数组的前半部分，所有偶数位于数组的后半部分。
- [https://leetcode-cn.com/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/](https://leetcode-cn.com/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/)
- 思路：双指针从两边开始循环。
- 答案：
```java
class Solution {
    public int[] exchange(int[] nums) {
        int start = 0;
        int end = nums.length - 1;
        while (start <= end) {
            if ((nums[start] & 1) != 0) {
                start++;
                continue;
            }
            if ((nums[end] & 1) == 0) {
                end--;
                continue;
            }
            swap(nums, start, end);

        }
        return nums;

    }

    private void swap(int[] nums, int a, int b) {

        nums[a] = nums[b] + nums[a];
        nums[b] = nums[a] - nums[b];
        nums[a] = nums[a] - nums[b];
    }
}
```

### 面试题15：链表中倒数第k个节点
- 题目：输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。
- [https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)
- 思路：双指针一个先跑k步，之后两个一起跑，当先跑的指针到达终点之后，后跑的指针所在的地点就是倒数第几步。
- 答案：
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode getKthFromEnd(ListNode head, int k) {
        ListNode start = head;
        ListNode end = head;
        for (int i = 0 ;i < k; i++) {
            end = end.next;
        }
        while (end != null) {
            end = end.next;
            start = start.next;
        }
        return start;
    }
}
```

### 面试题16：反转链表
- 题目：定义一个函数，输入一个链表的头节点，反转该链表并输出反转后链表的头节点。
- [https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/](https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/)
- 思路：多定义一个指针保存当前的额下一个指针，即可一遍循环完成。难点在于递归实现。
- 答案：
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
 // 1 2 null
class Solution {
    public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        ListNode newHead = reverseList(head.next);
        head.next.next = head;
        head.next = null;
        return newHead;

    }
}
```

### 面试题17：合并两个排序的链表
- 题目：输入两个递增排序的链表，合并这两个链表并使新链表中的节点仍然是递增排序的。
- [https://leetcode-cn.com/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/submissions/](https://leetcode-cn.com/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/submissions/)
- 思路：两个节点比较，这里考察的是代码的鲁棒性。
- 答案：
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) {
            return l2;
        }
        if (l2 == null) {
            return l1;
        }
        ListNode newHead;
        if (l1.val > l2.val) {
            newHead = l2;
            l2 = l2.next;
        } else {
            newHead = l1;
            l1 = l1.next;
        }
        ListNode t = newHead;
        while (l1 != null && l2 != null) {
            if (l1.val > l2.val) {
                t.next = l2;
                l2 = l2.next;
            } else {
                t.next = l1;
                l1 = l1.next;
            }
            t = t.next;
        }
        if (l1 == null) {
            t.next = l2;
        }
        if (l2 == null) {
            t.next = l1;
        }

        return newHead;

    }
}
```
### 面试题18：树的子结构
- 题目：输入两棵二叉树A和B，判断B是不是A的子结构。(约定空树不是任意一个树的子结构)。B是A的子结构， 即 A中有出现和B相同的结构和节点值。
- [https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)
- 思路：只是简单的考核代码的鲁棒性，递归实现即可。
- 答案：
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public boolean isSubStructure(TreeNode A, TreeNode B) {
        if (B == null || A == null) {
            return false;
        }
        return isSame(A, B) || isSubStructure(A.left, B) || isSubStructure(A.right, B);
    }

    private boolean isSame(TreeNode a, TreeNode b) {
        if (a == null && b != null) {
            return false;
        } else {
            if (a != null && b != null) {
                return a.val == b.val && isSame(a.left, b.left) && isSame(a.right, b.right);
            } else {
                return true;
            }
        }

    }
}
```
