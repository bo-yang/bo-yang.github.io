---
layout: post
title: Binary Tree Operations(II) - Path Sum and Cycle Detection
categories: 
- Algorithm
- C/C++
tags:
- Algorithm
- C/C++
status: publish
type: post
published: true
author: Bo Yang
---
This is the second article on binary tree operations. For topics on binary tree build, traversal and comparison, please refer to [Binary Tree Operations(I)](http://bo-yang.github.io/2014/05/26/binary-tree-traversal/)

1. [Path Sum](#path_sum)
2. [Build Binary Tree With Cycle](#build_cycle)
3. [Detect Cycle in Binary Tree](#detect_cycle)

### 1. <a name="path_sum">Path Sum</a>

The [Path Sum problem](http://www.programcreek.com/2013/01/leetcode-path-sum/) of binary tree is:

>Given a binary tree and a sum, determine if the tree has a root-to-leaf path such that adding up all the values along the path equals the given sum.

This problem can be easily solved by level-order traversal. During traversal, record the sum and path to each node: 

- if the sum to current node is larger than the given sum, then remove the sum and path to current node and stop tracing its child nodes.
- if the sum to current node equals to the given sum, the print or record the path.
- if to current node is less than the given sum, continue searching its children.

For the source code of this section, please refer to function `PathSum()` in [my Binary Tree Library](https://github.com/bo-yang/BinaryTree).

### 2. <a name="build_cycle">Build Binary Tree With Cycle</a>

To build a binary tree with cycles, we can first build a binary tree using the method described in [Binary Tree Operations(I)](http://bo-yang.github.io/2014/05/26/binary-tree-traversal/), and then add links between two nodes. However, we need to keep in mind that we are dealing with binary tree, so for each node, there are at most two children(`left` and `right`).

	struct TreeNode {
		int val;
		TreeNode *left;
		TreeNode *right;
		TreeNode(int x) : val(x), left(NULL), right(NULL) {}
	};

For example, given a binary tree in following image,

![Binary Tree with Cycle]({{ site.url }}/assets/images/binary_tree_with_cycle.png)

We can build a tree using string sequence {1,2,3,4,#,5,#} at first, and then add links {3->2, 2->5}. Or build tree {1,2,3,4,5,#,#} first and then add links {3->2,3->5}. 

For the source code of this section, please refer to functions `BuildTree()` and `BuildCycleTree()` in [my Binary Tree Library](https://github.com/bo-yang/BinaryTree).


### 3. <a name="detect_cycle">Detect Cycle in Binary Tree</a>

Cycles in a binary tree can be detected by DFS(in preorder) - if there's a cycle, there must be a node has a child node that is already been accessed before(i.e. a right hand node linked to the left hand node). A `unordered_set` can be used to record the nodes that have been accessed.

Considering above example again, when we traverse that binary tree in pre-order, we would first access nodes {1,2,4,5}. And when we accessing node `3`, we will find that node `3` has a child `2`(or`5`) that has already been accessed. So this tree has a cycle.

For the source code of this section, please refer to function `HasLoop()` in [my Binary Tree Library](https://github.com/bo-yang/BinaryTree).

### References:

1. [Detect Cycle in a Directed Graph](http://www.geeksforgeeks.org/detect-cycle-in-a-graph/)
