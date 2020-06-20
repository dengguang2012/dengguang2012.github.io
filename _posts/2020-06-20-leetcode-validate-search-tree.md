# leetcode 98.Validate Binary Search Tree

二叉搜索树。 如果该树的左子树不为空， 则左子树的虽有节点的值均小于它的根结点的值；若它的右子树不为空，则它的右子树上所有节点的值均大雨它的根结点的值；它的左右子树也为二叉搜索树。

易错点： 认为左边儿子 < 当前节点 < 右边儿子 其实是左子树所有节点

使用两种方法求解
1. 使用递归方法， 把子树的大小范围作为参数传递
```
class Solution:
    def isValidBST(self, root):
        def helper(node, lower = float('-inf'), upper = float('inf')):
            if not node:
                return True
            val = node.val
            if val <= lower or val >= upper:
                return False
            if not helper(node.right, val, upper):
                return False
            if not helper(node.left, lower, val):
                return False
            return True
        return helper(root)
```

2. 使用基于栈的中序遍历方法

```
class Solution2:
    def isValidBST(self, root):
        stack, inorder = [], float('-inf')
        while stack or root:
            while root:
                stack.append(root)
                root = root.left
            root = stack.pop()
            if root.val <= inorder:
                return False
            inorder = root.val
            root = root.right
        return True
```