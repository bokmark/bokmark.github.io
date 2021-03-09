---
title: '剑指offer - 数据结构'
date: 2021-02-28
weight: 1
categories: ["剑指offer","算法"]
---

# 剑指offer中的算法题
> [剑指Offer](/books/Offer.pdf)

## 数据结构
### 面试题3：二维数组中的查找
- 题目：在一个二维数组中，每一行都按照从左到右的递增顺序排序，每一列都按照从上到下递增的顺序排序，请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。
- [https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)
- 思路： 先拿出一个例子找出规律，然后再实现成代码。我们在这里从右上角开始往前找，如果找到的数大于target这前移一位，如果小于这下移一位。重复这歌步骤直到x 或者y超出边界。
- 答案：
```java
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
            return false;
        }
        int y = matrix[0].length - 1, x = 0;
        for (;x < matrix.length && y >= 0;) {
            if (matrix[x][y] == target) {
                return true;
            }
            if (matrix[x][y] > target) {
                y --;
            } else {
                x ++;
            }
        }
        return false;
    }
}
```
### 面试题4：替换空格
- 题目：请实现一个函数，把字符串 s 中的每个空格替换成"%20"。
- [https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/](https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/)
- 思路： 这里考察的是 空格一个char 替换为“%20” 三个char。如果可以新开一个string。那么就新开。这种方法比较简单，如果要在原来的string执行（当然Java不存在这个问题。），那么从后面开始填充字符。
- 答案：
```java
class Solution {
    public String replaceSpace(String s) {
        char[] carr = s.toCharArray();
        int max = 0;
        for (char c : carr) {
            if (c == ' ') {
                max += 2;
            }
            max++;
        }
        StringBuilder sb = new StringBuilder(max);
        for (char c : carr) {
            if (c == ' ') {
                sb.append('%');
                sb.append('2');
                sb.append('0');
            } else {
                sb.append(c);
            }
        }
        return sb.toString();
    }
}
```
### 面试题5：从尾到头打印链表
- 题目：输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。
- [https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)
- 思路：这个问题第一想法就是使用栈来实现。但是栈需要另外开辟一块空间，有更优的解法嘛？递归? Java 这个貌似不好弄。且看我这种解法。
- 答案：
```Java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public int[] reversePrint(ListNode head) {
        int size = 0;
        ListNode p = head;
        while(p != null) {
            size++;
            p = p.next;
        }
        int r[] = new int[size];
        size --;
        while (head != null) {
            r[size] = head.val;
            head = head.next;
            size--;
        }
        return r;
    }

}
```
### 面试题6：重建二叉树
- 题目：输入某二叉树的前序遍历和中序遍历的结果，请重建该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。
- [https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/)
- 思路，递归实现，弄清楚前序遍历 和中序遍历即可。
- 答案：
```Java
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
    private Map<Integer, Integer> map = new HashMap<>();
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        for (int i = 0; i < inorder.length; i++) {
            map.put(inorder[i], i);
        }
        return myBuildTree(preorder, inorder, 0, preorder.length - 1, 0, inorder.length - 1);
    }

    public TreeNode myBuildTree(int[] preorder, int[] inorder, int preorder_left, int preorder_right, int inorder_left, int inorder_right) {

        if (inorder_left > inorder_right) {
            return null;
        }
        TreeNode root = new TreeNode(preorder[preorder_left]); // 3
        int centerInoder = map.get(preorder[preorder_left]);   // 3
        int leftpreSize = centerInoder - inorder_left;         // 3
        root.left = myBuildTree(preorder, inorder,
        preorder_left + 1, preorder_left + leftpreSize,
        inorder_left, centerInoder - 1);

        root.right = myBuildTree(preorder, inorder,  
        preorder_left + leftpreSize + 1, preorder_right,
        centerInoder + 1, inorder_right);

        return root;
    }
}
```
### 面试题7： 用两个栈实现队列
- 题目：用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 appendTail 和 deleteHead ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，deleteHead 操作返回 -1 )
- 地址：[https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)
- 思路：双栈实现队列，关键在于 两个栈倒来倒去。但是在这里，其实可以分两部分，tail的队列和head的队列，只有当head为空的时候 删除head 才需要把tail的队列倒到head中。
- 答案：（可以修改stack实现 为 linkedlist 会更快。stack使用向量实现就是数组实现。）
```Java
class CQueue {
    private Stack<Integer> tail = new Stack<>();
    private Stack<Integer> head = new Stack<>();

    public CQueue() {

    }

    public void appendTail(int value) {
        tail.push(value);
    }

    public int deleteHead() {
        if (head.isEmpty()) {
            while(!tail.isEmpty()) {
                head.push(tail.pop());
            }
        }
        if (head.isEmpty()) {
            return -1;
        }
        return head.pop();
    }
}

/**
 * Your CQueue object will be instantiated and called as such:
 * CQueue obj = new CQueue();
 * obj.appendTail(value);
 * int param_2 = obj.deleteHead();
 */
 ```
### 面试题8：旋转数组的最小数字
- 题目：把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如，数组 [3,4,5,1,2] 为 [1,2,3,4,5] 的一个旋转，该数组的最小值为1。
- [https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)
- 思路：直接一个for循环是最简单的，O(N)。再进一步二分查找法。
- 答案：
```Java
class Solution {
    public int minArray(int[] numbers) {
        int p;
        int start = 0;
        int end = numbers.length - 1;
        while (start < end) {
            p = (end - start) / 2 + start;
            if (numbers[end] > numbers[p]) {
                end = p;
            } else if (numbers[end] < numbers[p]){
                start = p + 1;
            } else {
                end -= 1;
            }
        }
        return numbers[start];
    }
}
```
### 面试题9：斐波那契数列
- 题目：写一个函数，输入 n ，求斐波那契（Fibonacci）数列的第 n 项（即 F(N)）。斐波那契数列的定义如下：
- [https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof/](https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof/)
- 思路：从0开始一个一个累加上去一直加到 第n项。（这里Leetcode 为了不超出int的返回，将结果取模，这不是我们需要关注的地方）
- 答案：
```java
class Solution {
    public int fib(int n) {
        int n1 = 1;
        int n2 = 0;
        for (int i = 0; i < n; i++) {
            n1 = n2 + n1;
            n2 = n1 - n2;
            n1 = n1 % 1000000007;
        }
        return n2;
    }
}
```
### 面试题9-2：青蛙跳台阶问题
- 题目：一只青蛙一次可以跳上1级台阶，也可以跳上2级台阶。求该青蛙跳上一个 n 级的台阶总共有多少种跳法。
- [https://leetcode-cn.com/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/](https://leetcode-cn.com/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)
- 思路：斐波那契数列的实际应用
- 答案：
```java
class Solution {
    public int fib(int n) {
        int n1 = 1;
        int n2 = 0;
        for (int i = 0; i < n + 1; i++) {
            n1 = n2 + n1;
            n2 = n1 - n2;
            n1 = n1 % 1000000007;
        }
        return n2;
    }
}
```
### 面试题10：二进制中的1的个数
- 题目：请实现一个函数，输入一个整数（以二进制串形式），输出该数二进制表示中 1 的个数。例如，把 9 表示成二进制是 1001，有 2 位是 1。因此，如果输入 9，则该函数输出 2。
- [https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)
- 思路：当n&(n-1) 会消除一个1.
- 答案：
```java
public class Solution {
    // you need to treat n as an unsigned value
    public int hammingWeight(int n) {
        int count = 0;
        while (n != 0) {
            count++;
            n &= n-1;
        }
        return count;
    }
}
```
### 面试题
- 题目：
- []()
- 思路：
- 答案：
```java

```
