---
layout: post
title: "Binary Search Tree"
date: 2015-09-24 01:28:33 +0800
comments: true
categories: algorithm
---

I try to collect problems about `Binary Search Tree (BST)` which are asked in interview frequently.

Basicly, here is the definition of Binary Search Tree. And I show the way how to implement the basic operation like `insert`, `delete` and so on.

``` python
class TreeNode() :
    def __init__(self, num = -1) :
      self.val = num
      self.right  = None
      self.left   = None
```

You could find all my practices with BST in github: [BST](https://github.com/jasonleaster/Algorithm/blob/master/Binary_Search_Tree/Python_version/bst.py)

![images](/images/img_for_2015_09_25/bst.png)

<!-- more -->

There are three different ways to travel a BST. N(node), L(left subtree), R(right subtree)

 * NLR: Firstly the traveller access the data of the Node(N) and then it enter into the left sub-tree, travel the right sub-tree

 * LNR

 * LRN

It's very easy and obvious to implement the recursive definition of the three different traveller.

You will find that it's a very efficient way to understand what means Pre-traveller, In-traveller and Post-traveller.

``` python
    """
    Recursive definition
    """
    def pre_traveller(self, node):
        if node == None :
            return None

        print node.val
        self.pre_traveller(node.left)
        self.pre_traveller(node.right)

    def in_traveller(self, node):
        if node == None:
            return None

        self.in_traveller(node.left)
        print node.val
        self.in_traveller(node.right)

    def post_traveller(self, node):
        if node == None:
            return None

        self.post_traveller(node.left)
        self.post_traveller(node.right)
        print node.val

```

But the interview officer may not be satisfy with your recursive implementation. Take some time to understand the iteratly implementation below there.

``` python

    """
    Iterately implementation

    Given a binary tree, return the preorder traversal of 
    its nodes' values.
    """
    def preorderTraversal(self, root):
        """
        :type root: TreeNode
        :rtype: List[int]
        """
        if root is None:
            return []

        stack = [root]
        ret = []
        
        while len(stack) != 0:
            node = stack.pop()
            ret.append(node.val)
            if node.right is not None:
                stack.append(node.right)
            if node.left is not None:
                stack.append(node.left)
                
        return ret

    """
    Given a binary tree, return the inorder traversal 
    of its nodes' values.
    """
    def inorderTraversal(self, root):
        res, stack = [], []
        while True:
            while root:
                stack.append(root)
                root = root.left
            if not stack:
                return res
            node = stack.pop()
            res.append(node.val)
            root = node.right

    def postorderTraversal(self, root):
        """
        :type root: TreeNode
        :rtype: List[int]
        """
        res   = []
        stack = [root]
        
        while len(stack):
            node = stack.pop()
            if node:
                res.append(node.val)
                stack.append(node.left)
                stack.append(node.right)
                
        return res[::-1]
```

There is another interesting problem that how to travel a BST in level order. Something like this:
For example:

Given binary tree {3,9,20,#,#,15,7},

![images](/images/img_for_2015_09_25/1.png)

return its level order traversal as:

![images](/images/img_for_2015_09_25/2.png)

At this moment, you should try to use some basic data structure to solve this problem. Don't forget STACK :)

``` python
    """
    Given a binary tree, return the level order traversal of 
    its nodes' values. (ie, from left to right, level by level).
    """
    def levelOrder(self, node):
        if node is None:
            return []

        stack = [node]
        length = len(stack)
        ret = []
        j = 0

        while length != 0:
            ret.append([])
            for i in range(0, length):
                
                ret[j].append(stack[i].val)
                if stack[i].left != None:

                    stack.append(stack[i].left)
                if stack[i].right != None:
                    stack.append(stack[i].right)

            for i in range(0, length):
                stack.remove(stack[0])

            length = len(stack)
            j += 1

        return ret
```

After you solve this problem, you have knew that stack is a efficient and useful ADT. Once you meet a hard problem, try to use another ADT to solve your problem.


Function `isValidBST` help us to check whether the tree is a BST. Now, if you have no idea about what means a BST, go to wikipedia and help yourself :)

You know that if we travel the tree with `In-Order traveller`, the output of the traveller is sorted from small element to big one.
``` python
    def isValidBST(self, root):
        A = []
        A = self.in_traveller(self.root)

        for i in range(0, len(A)):
            if A[i-1] >= A[i]:
                return False

        return True
```

There is a joke about how to invert a BST. 
> Google: 90% of our engineers use the software you wrote (Homebrew), but you can’t invert a binary tree on a whiteboard so fuck off.

It's easy to solve this problem by recursion.

``` python
            
    def invertTree(self, root):
        if root is None:
            return
        
        root.left, root.right = root.right, root.left
        self.invertTree(root.left)
        self.invertTree(root.right)
```

You may also be asked to translate a sorted array into BST. So, how to make it?

``` python
    def sortedArrayToBST(self, nums):
        """
        :type nums: List[int]
        :rtype: TreeNode
        """
        return self.helperSortedArrayToBST(nums, 0, len(nums)-1)
    
    def helperSortedArrayToBST(self, nums, start, end):
        if start > end:
            return None
            
        middle = int((start + end)/2)
        
        root = TreeNode(nums[middle])
        root.left  = self.helper(nums, start, middle-1) 
        root.right = self.helper(nums, middle + 1, end)
        
        return root
```

How to use the output of `InOrder-Traveller` and `PostOrder-Traveller` to rebuild a BST ?

``` python
    """
    Given inorder and postorder traversal of a tree, 
    construct the binary tree.

    Note:
    You may assume that duplicates do not exist in the tree.
    """
    def buildTree(self, inorder, postorder):
        """
        :type inorder: List[int]
        :type postorder: List[int]
        :rtype: TreeNode
        """
        if len(inorder) == 0 or len(postorder) == 0:
            return None
        
        index = inorder.index(postorder.pop())
        root = TreeNode(inorder[index])
        
        # You HAVE TO build right sub-tree first, 
        # otherwise you will get wrong answer
        # because you poped the last element of @postorder before here.
        root.right = self.buildTree(inorder[index+1:], postorder)
        root.left  = self.buildTree(inorder[0:index] , postorder)
        
        return root
```


-----------
Photo By Jason Leaster in ChangDe, China

![images](/images/img_for_2015_09_25/highschool.jpg)
