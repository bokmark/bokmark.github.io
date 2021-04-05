---
title: '剑指offer - 解决面试题的思路'
date: 2021-02-28
weight: 2
categories: ["剑指offer","算法"]
---
# 剑指offer中的算法题
> [剑指Offer](/books/Offer.pdf)

## 解决面试题的思路

### 面试题19：二叉树的镜像（举例让抽象问题具现化）
- 题目：请完成一个函数，输入一个二叉树，该函数输出它的镜像。
- [https://leetcode-cn.com/problems/er-cha-shu-de-jing-xiang-lcof/](https://leetcode-cn.com/problems/er-cha-shu-de-jing-xiang-lcof/)
- 思路：递归替换左右子节点即可。
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
    public TreeNode mirrorTree(TreeNode root) {
        if (root == null) {
            return null;
        }
        TreeNode left = root.left;
        TreeNode right = root.right;
        root.left = mirrorTree(right);
        root.right = mirrorTree(left);
        return root;

    }
}
```

### 面试题20：顺时针打印矩阵（举例让抽象问题具现化）
- 题目：输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字。
- [https://leetcode-cn.com/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/](https://leetcode-cn.com/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/)
- 思路：没什么算法问题，就这么写。考察写代码的质量多个判断
- 答案：
```java
class Solution {
    public int[] spiralOrder(int[][] matrix) {
        if (matrix == null || matrix.length == 0) {
            return new int[0];
        }
        int n = matrix.length, m = matrix[0].length;
        int[] res = new int[n * m];
        int x = 0, y = 0;// 读取指针
        int startX = 0, endX = n - 1;
        int startY = 0, endY = m - 1;
        int way = 0; // 0 xiangzuo 1 xiang xia 2 xiang you 3 xiang shang
        for (int i = 0;  i < n * m; i ++) {
            res[i] = matrix[x][y];
            switch(way) {
                case 0: {
                    if (y < endY) {
                        y++;
                    } else if (y == endY) {
                        way++;
                        x++;
                    }
                }
                break;
                case 1: {
                    if (x < endX) {
                        x++;
                    } else if (x == endX) {
                        way++;
                        y--;
                    }
                }
                break;
                case 2: {
                    if (y > startY) {
                        y--;
                    } else if (y == startY) {
                        way++;
                        x--;
                    }
                }
                break;
                case 3: {
                    if (x > startX + 1) {
                        x--;
                    } else if (x == startX + 1) {
                        y ++;
                        way=0;
                        startX++;
                        startY++;
                        endX--;
                        endY--;
                    }
                }
                break;
            }
        }
        return res;
    }
}
```

### 面试题21：包含min函数的栈（举例让抽象问题具现化）
- 题目：定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的 min 函数在该栈中，调用 min、push 及 pop 的时间复杂度都是 O(1)
- [https://leetcode-cn.com/problems/bao-han-minhan-shu-de-zhan-lcof/](https://leetcode-cn.com/problems/bao-han-minhan-shu-de-zhan-lcof/)
- 思路：栈的push 和pop 都是o(1)的，关键在于min方法的耗时缩减到o(1)。用额外的一个stack 来保存 min的思路解决问题。
- 答案：
```java
class MinStack {
    private Stack<Integer> s = new Stack<>(); // 2 0 3 0
    private Stack<Integer> min = new Stack<>(); // 2 0 0

    /** initialize your data structure here. */
    public MinStack() {

    }

    public void push(int x) {
        s.push(x);
        if (min.isEmpty() || min.peek() >= x) {
            min.push(x);
        }
    }

    public void pop() {
        int sp = s.pop();
        if (sp <= min.peek()) {
            min.pop();
        }
    }

    public int top() {
        if (s.isEmpty()) {
            return 0;
        }
        return s.peek();
    }

    public int min() {
        if (min.isEmpty()) {
            return 0;
        }
        return min.peek();
    }
}

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack obj = new MinStack();
 * obj.push(x);
 * obj.pop();
 * int param_3 = obj.top();
 * int param_4 = obj.min();
 */
```

### 面试题22：栈的压入、弹出序列（举例让抽象问题具现化）
- 题目：输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如，序列 {1,2,3,4,5} 是某栈的压栈序列，序列 {4,5,3,2,1} 是该压栈序列对应的一个弹出序列，但 {4,3,5,1,2} 就不可能是该压栈序列的弹出序列
- [https://leetcode-cn.com/problems/zhan-de-ya-ru-dan-chu-xu-lie-lcof/](https://leetcode-cn.com/problems/zhan-de-ya-ru-dan-chu-xu-lie-lcof/)
- 思路：循环判断比较栈顶元素。（举例让抽象问题具现化）
- 答案：
```java
class Solution {
    public boolean validateStackSequences(int[] pushed, int[] popped) {
       Stack<Integer> stack = new Stack<>();
        int i = 0;
        for(int num : pushed) {
            stack.push(num); // num 入栈
            while(!stack.isEmpty() && stack.peek() == popped[i]) { // 循环判断与出栈
                stack.pop();
                i++;
            }
        }
        return stack.isEmpty();
    }
}
```

### 面试题23：从上往下打印二叉树（举例让抽象问题具现化）
- 题目：从上到下打印出二叉树的每个节点，同一层的节点按照从左到右的顺序打印
- [https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/)
- 思路：不断的将未打印的节点放入队列，然后打印左右子节点。
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
    public int[] levelOrder(TreeNode root) {
        if(root == null) return new int[0];
        Queue<TreeNode> queue = new LinkedList<>(){{ add(root); }};
        ArrayList<Integer> ans = new ArrayList<>();
        while(!queue.isEmpty()) {
            TreeNode node = queue.poll();
            ans.add(node.val);
            if(node.left != null) queue.add(node.left);
            if(node.right != null) queue.add(node.right);
        }
        int[] res = new int[ans.size()];
        for(int i = 0; i < ans.size(); i++)
            res[i] = ans.get(i);
        return res;
    }
}
```

### 面试题23-1：从上往下打印二叉树（举例让抽象问题具现化）
- 题目：从上到下打印出二叉树的每个节点，同一层的节点按照从左到右的顺序打印
- [https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/)
- 思路：不断的将未打印的节点放入队列，然后打印左右子节点。
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
    public List<List<Integer>> levelOrder(TreeNode root) {
        if(root == null) return new ArrayList<>();
        Queue<TreeNode> queue = new LinkedList<>(){{ add(root); }};
        List<List<Integer>> ans = new ArrayList<List<Integer>>();
        while(!queue.isEmpty()) {
            List<Integer> temp = new ArrayList<>();
            for (int i = queue.size(); i > 0; i--) {
                TreeNode node = queue.poll();
                temp.add(node.val);
                if(node.left != null) queue.add(node.left);
                if(node.right != null) queue.add(node.right);
            }
            ans.add(temp);
        }
        return ans;
    }
}
```

### 面试题23-2：从上往下打印二叉树（举例让抽象问题具现化）
- 题目：请实现一个函数按照之字形顺序打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右到左的顺序打印，第三行再按照从左到右的顺序打印，其他行以此类推。
- [https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/)
- 思路：不断的将未打印的节点放入队列，然后打印左右子节点。
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
    public List<List<Integer>> levelOrder(TreeNode root) {
        if(root == null) return new ArrayList<>();
        Queue<TreeNode> queue = new LinkedList<>(){{ add(root); }};
        List<List<Integer>> ans = new ArrayList<List<Integer>>();

        while(!queue.isEmpty()) {
            LinkedList<Integer> temp = new LinkedList<>();
            for (int i = queue.size(); i > 0; i--) {
                TreeNode node = queue.poll();
                if (ans.size() % 2 == 0) {
                    temp.addLast(node.val);
                } else {
                    temp.addFirst(node.val);
                }

                if(node.left != null) queue.add(node.left);
                if(node.right != null) queue.add(node.right);
            }
            ans.add(temp);
        }
        return ans;
    }
}
```

### 面试题24：二叉搜索树的后序遍历序列
- 题目：输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历结果。如果是则返回 true，否则返回 false。假设输入的数组的任意两个数字都互不相同。
- [https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/)
- 思路：后序遍历的最后一个节点是中间的点位，之前的所有数字都是先小于 中间数字，又大于中间数字，只要不符合这个规则就false
- 答案：
```java
class Solution {
    public boolean verifyPostorder(int[] postorder) {

        if (postorder == null || postorder.length == 0) {
            return true;
        }
        return verify(postorder, 0, postorder.length - 1);

    }
    private boolean verify(int[] postorder, int i, int j) {

        if(i >= j) return true;
        int p = i;
        while(postorder[p] < postorder[j]) p++;
        int m = p;
        while(postorder[p] > postorder[j]) p++;
        return p == j && verify(postorder, i, m - 1) && verify(postorder, m, j - 1);

    }
}
```

### 面试题25：二叉树中和为某一值的路径
- 题目：输入一棵二叉树和一个整数，打印出二叉树中节点值的和为输入整数的所有路径。从树的根节点开始往下一直到叶节点所经过的节点形成一条路径。
- [https://leetcode-cn.com/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/](https://leetcode-cn.com/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/)
- 思路：回溯法考察的是 如何回溯。
- 答案：
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    private List<List<Integer>> res = new ArrayList<>();
    public List<List<Integer>> pathSum(TreeNode root, int target) {
        List<Integer> list = new LinkedList<>();
        cur(list, root, target);
        return res;
    }
    private void cur(List<Integer> list, TreeNode root, int target) {
        if (root == null) {
            return;
        }
        target -= root.val;
        list.add(root.val);
        if (root.left == null && root.right == null) {
            if (target == 0) {
                res.add(new LinkedList<Integer>(list));
            }
            list.remove(list.size() - 1);
            return;
        } else {
            cur(list, root.left, target);
            cur(list, root.right, target);
            list.remove(list.size() - 1);
        }
    }
}
```

### 面试题26：复杂链表复制
- 题目：请实现 copyRandomList 函数，复制一个复杂链表。在复杂链表中，每个节点除了有一个 next 指针指向下一个节点，还有一个 random 指针指向链表中的任意节点或者 null。
- [https://leetcode-cn.com/problems/fu-za-lian-biao-de-fu-zhi-lcof/](https://leetcode-cn.com/problems/fu-za-lian-biao-de-fu-zhi-lcof/)
- 思路：a->b->c =>  a->a1->b->b1->c->c1 => a->b->c  a1->b1->c1
- 答案：
```java
/*
// Definition for a Node.
class Node {
    int val;
    Node next;
    Node random;

    public Node(int val) {
        this.val = val;
        this.next = null;
        this.random = null;
    }
}
*/
class Solution {
    public Node copyRandomList(Node head) {
        if (head == null) {
            return null;
        }
        Node p = head;
        while (p != null) {
            Node t = new Node(p.val);
            t.next = p.next;
            p.next = t;
            p = p.next.next;
        }

        //ok
        p = head;
        while (p != null) {
            if (p.random != null) {
                Node t = p.next;
                t.random = p.random.next;
            }
            // next jiacha ?
            p = p.next.next;
        }
        p = head;
        Node copy = p.next;
        Node t = copy;
        while(p != null && t != null) {
            p.next = p.next.next;
            p = p.next;
            if (t.next == null) {
                break;
            }
            t.next = t.next.next;
            t = t.next;
        }
        return copy;
    }
}
```

### 面试题27：二叉搜索树与双向链表
- 题目：输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的循环双向链表。要求不能创建任何新的节点，只能调整树中节点指针的指向。
- [https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/)
- 思路：
- 答案：
```java
/*
// Definition for a Node.
class Node {
    public int val;
    public Node left;
    public Node right;

    public Node() {}

    public Node(int _val) {
        val = _val;
    }

    public Node(int _val,Node _left,Node _right) {
        val = _val;
        left = _left;
        right = _right;
    }
};
*/
class Solution {
    Node pre, head;
    public Node treeToDoublyList(Node root) {
        if(root == null) return null;
        dfs(root);
        head.left = pre;
        pre.right = head;
        return head;
    }
    void dfs(Node cur) {
        if(cur == null) return;
        dfs(cur.left);
        if(pre != null) pre.right = cur;
        else head = cur;
        cur.left = pre;
        pre = cur;
        dfs(cur.right);
    }
}
```

### 面试题28：字符串的排列
- 题目：输入一个字符串，打印出该字符串中字符的所有排列。
- [https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/](https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/)
- 思路：字符串取出字符，固定位数，然后排序。
- 答案：
```java
class Solution {
    List<String> res = new LinkedList<>();
    char[] c;
    public String[] permutation(String s) {
        c = s.toCharArray();
        dfs(0);
        return res.toArray(new String[res.size()]);
    }
    void dfs(int x) {
        if(x == c.length - 1) {
            res.add(String.valueOf(c));      // 添加排列方案
            return;
        }
        HashSet<Character> set = new HashSet<>();
        for(int i = x; i < c.length; i++) {
            if(set.contains(c[i])) continue; // 重复，因此剪枝
            set.add(c[i]);
            swap(i, x);                      // 交换，将 c[i] 固定在第 x 位
            dfs(x + 1);                      // 开启固定第 x + 1 位字符
            swap(i, x);                      // 恢复交换
        }
    }
    void swap(int a, int b) {
        char tmp = c[a];
        c[a] = c[b];
        c[b] = tmp;
    }
}
```
