---
layout: post
title: Binary Tree Operations(IV) - Determine if a Binary Tree is a Binary Search Tree
categories: 
- Algorithm
- Python
tags:
- Algorithm
- Python
status: publish
type: post
published: true
author: Bo Yang
---

The problem is: _given a binary tree, how to determine if it is a Binary Search Tree(BST) or not?_ A binary search tree is a binary tree data structure which has the following properties.

- The left subtree of a node contains only nodes with keys less than the node’s key.
- The right subtree of a node contains only nodes with keys greater than the node’s key.
- Both the left and right subtrees must also be binary search trees. 

The strategy is to recursively check (1) if the maximum value in the left subtree is smaller than the root node, and (2) if the minimum value in the right subtree is greater than the root node. 

In order to implement the recursion, a helper function `IsBSTHelper(TreeNode* root, int min, int max)` is needed, where `min` and `max` define the range of possible values of subtree nodes. When checking a binary tree, the initial value of `min` is `INT_MIN` and the initial value of `max` is `INT_MAX`. However, when exploring the left subtree, the `max` will be changed to the value of root, and the `min` value remains; when exploring the right subtree, the `min` value is the value of root and the `max` value remains. 

For complete code of checking BST, please refer to functions `IsBST()` and `IsBSTHelper(TreeNode* root, int min, int max)` in [my binary tree library](https://github.com/bo-yang/BinaryTree). Please note that, we assume a `NULL` node is a valid binary search tree.


	bool IsBST(TreeNode* root) {
		return IsBSTHelper(root, INT_MIN, INT_MAX);
	}
	
	// Helper function for checking binary search tree
	bool IsBSTHelper(TreeNode* root, int min, int max) {
		if(root==NULL)
			return true;

		assert(root->val > min && root->val < max);

		bool bst_left=true;
		bool bst_right=true;
		if(root->left!=NULL && (root->left->val >= root->val || root->left->val <= min))
			return false;
		else
			bst_left=IsBSTHelper(root->left, min, root->val);
		
		if(root->right!=NULL && (root->right->val <= root->val || root->right->val >= max))
			return false;
		else
			bst_right=IsBSTHelper(root->right, root->val, max);

		return (bst_left && bst_right);
	}


