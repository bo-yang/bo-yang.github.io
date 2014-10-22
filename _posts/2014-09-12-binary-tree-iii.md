---
layout: post
title: Binary Tree Operations(III) - Convert a Binary Tree to Down-Right Representation 
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
_This is the third article on binary tree operations. For other topics on binary tree, please refer to:_

1. [Binary Tree Operations(I)](http://bo-yang.github.io/2014/05/26/binary-tree-traversal/)
2. [Binary Tree Operations(II)](http://bo-yang.github.io/2014/08/16/binary-tree-ii/)
3. [Binary Tree Operations(III) - Convert a Binary Tree to Down-Right Representation](http://bo-yang.github.io/2014/09/12/binary-tree-iii/)
4. [Binary Tree Operations(IV) - Determine if a Binary Tree is a Binary Search Tree](http://bo-yang.github.io/2014/10/09/is-valid-bst/)

Left-Right representation of a binary tree is standard representation where every node has a pointer to left child and another pointer to right child.

Down-Right representation is an alternate representation where every node has a pointer to left (or first) child and another pointer to next sibling. So siblings at every level are connected from left to right.

Given a binary tree in left-right representation as below

		  A
		 /  \
		B    C
		    /  \
		   D    E
		  /    /  \
		 F    G    H

Convert the structure of the tree to down-right representation like the below tree.

	   	A
	   	|
	   	B – C
	   	|
	   	D — E
	   	|
	   	F — G – H

The conversion should happen in-place, i.e., left child pointer should be used as down pointer and right child pointer should be used as right sibling pointer.

The solution to this problem is (1) traversing this binary tree in layer order, (2)converting all nodes in the same layer as a single list and (3) connecting all the head nodes in each layer together. 

In order to connecting layers together via heads, at the begining of each layer, we need to record the leftmost node as the head of this layer. And after traversing current layer, we also need to link the head of current layer to the head of next layer. 

Following is the code to converting a left-right binary tree to down-right representation. For more details, please refer to [https://github.com/bo-yang/BinaryTree](https://github.com/bo-yang/BinaryTree). 

	TreeNode* Convert2DL(TreeNode* root) {
		if(root==NULL)
			return NULL;

		queue<TreeNode*> q;
		q.push(root);
		int nCur=1; // number of nodes in current layer
		TreeNode* head=NULL; // the head node in each layer
		while(!q.empty()) {
			head=q.front();
			TreeNode* cur=head;
			int nNext=0; // number of nodes in next layer
			for(int i=0;i<nCur;++i) {
				TreeNode* n=q.front();
				q.pop();
				if(n->left!=NULL) {
					q.push(n->left);
					nNext++;
				}
				if(n->right!=NULL) {
					q.push(n->right);
					nNext++;
				}
				// Link current node to the right of this layer
				if(n!=head) { 
					cur->right=n;
					cur=n;
					cur->left=NULL;
				}
			}
			nCur=nNext;
			cur->right=NULL;
			if(q.empty())
				head->left=NULL;
			else 
				head->left=q.front(); // point to the head of the next layer
		}

		return root;
	}


### References:

1. [Convert left-right representation of a bianry tree to down-right](http://geeksquiz.com/convert-left-right-representation-bianry-tree-right/)
