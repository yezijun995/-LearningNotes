# 144. 二叉树的前序遍历

[题目链接](https://leetcode-cn.com/problems/binary-tree-preorder-traversal//)

## 题目要求

- 给定一个二叉树，返回它的 前序 遍历。

示例:

````txt
输入: [1,null,2,3]  
   1
    \
     2
    /
   3 

输出: [1,2,3]
````

进阶: 递归算法很简单，你可以通过迭代算法完成吗？

## 解题思路

### 方法1：递归

#### 代码

````java
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

    List<Integer> list = new ArrayList<>();
    public List<Integer> preorderTraversal(TreeNode root) {
        recursion(root);
        return list;
    }

    public void recursion(TreeNode root){
        if(root == null) return;
        
        list.add(root.val);
        recursion(root.left);
        recursion(root.right);

    }

}
````

### 方法2：迭代

因为递归的思路与栈的工作原理相同，所以使用栈来存储，先序遍历是**先存入右子树在存入左子树**。

#### 代码

````txt
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

    public List<Integer> preorderTraversal(TreeNode root) {

        List<Integer> list = new ArrayList<>();
        
        if(root == null) return list;

        Stack<TreeNode> stack = new Stack<>();
        
        stack.push(root);
        while(!stack.isEmpty()){
            TreeNode node = stack.pop();

            list.add(node.val);

            if(node.right != null) stack.push(node.right);

            if(node.left != null) stack.push(node.left);
        }

        return list;
        
    }
}
````
