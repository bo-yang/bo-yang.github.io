---
layout: post
title: Binary Tree Traversal
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

There are two kinds of traversal for (binary) trees: Depth First Search(DFS) and Breadth First Search(BFS). Depth First Search visits the tree by proceeding deeper and deeper until it reaches the leaf nodes, including pre-order, in-order, and post-order tree traversal. DFS uses a data structure called Stack.

Breadth First Search is the most natural solution for level-order traversal, since it visits the nodes level by level. BFS requires the use of a data structure called Queue, which is a First In First Out (FIFO) structure.

For the complete C++ source code used in this article, please refer to: [https://github.com/bo-yang/misc/blob/master/BTtraverse.cc](https://github.com/bo-yang/misc/blob/master/BTtraverse.cc).

### Data Structure

The data structure used for binary tree is very simple, which is adopted from [LeetCode OJ](https://oj.leetcode.com/problems/binary-tree-inorder-traversal/):

	struct TreeNode {
	    int val;
	    TreeNode *left;
	    TreeNode *right;
		TreeNode() : val(0), left(NULL), right(NULL) {}
	    TreeNode(int x) : val(x), left(NULL), right(NULL) {}
	};

The constructing function TreeNode() is added to purely because it is easier to dynamically allocate TreeNode array by `new TreeNode[size]`. C++98 doesn't support specifying default arguments for objects allocated by `new`. However, the TreeNode array can simplify the process of building binary trees. 

### Build Binary Tree

In my implementation, binary trees are built from a vector of strings. For example, given sequence {1,2,3,#,#,4,#,#,5}, the following binary tree can be built:

<pre>
	  1
	 / \
	2   3
	   /
	  4
	   \
	    5
</pre>

The tree is built layer by layer, where the first element in sequence is viewed as the root node. Here the "#" is used as terminator. 

For each node other than the last layer, there are two children in the next layer, either a number(or string) or a "#"(invalid node). When parsing nodes in current layer, count the number of elements in next layer. The exception is for "#"s - they have no child nodes. In this way, we can parse the dependency of nodes layer by layer. 

Following is the C++ code for building binary trees:

	TreeNode* BuildTree(vector<string>& t) {
		tree=new TreeNode[t.size()];
	
		// Build binary tree from vector of strings, where # denotes an invalid node.
		//
		// The main idea is to parse vector of strings(nodes) layer by layer. 
		// The first item in the vector should be the root node, and the number of
		// first layer is one. When parsing the first layer, count the nodes of the
		// second layer, and so on.
		int idx=0;
		int nodes_cur_layer=1;
		vector<string>::iterator it=t.begin(); 
		while(idx<t.size()) {
			int nodes_next_layer=0; // all nodes, including #s
			int vi=0;	// index of valid nodes in current layer
			for(int i=0;i<nodes_cur_layer;++i) {
				if(*(it+i)=="#") { // Skip #s
					continue;
				}
	
				int left=nodes_cur_layer+2*vi;
				int right=nodes_cur_layer+2*vi+1;
				if(idx+left<t.size() && *(it+left)!="#" ) {
					tree[idx+i].left=&tree[idx+left];
				} else {
					tree[idx+i].left=NULL;
				}
				if(idx+right<t.size() && *(it+right)!="#") {
					tree[idx+i].right=&tree[idx+right];
				} else {
					tree[idx+i].right=NULL;
				}
	
				nodes_next_layer+=2;
				tree[idx+i].val=atoi((it+i)->c_str());
				vi++;
			}
	
			idx+=nodes_cur_layer;
			it+=nodes_cur_layer;
			nodes_cur_layer=nodes_next_layer;
			layers++;
		}
	
		return &tree[0];	// root of the tree
	}

### Preorder Traversal

To traverse a binary tree in Preorder, following operations are carried-out 
1. Visit the root, 
2. Traverse the left subtree, and 
3. Traverse the right subtree. 

For the example binary tree {1,2,3,#,#,4,#,#,5}, the preorder traversal is:

```
1,2,3,4,5
```

Following is my C++ implementation of preorder traversal([Pseudo Code](http://en.wikipedia.org/wiki/Tree_traversal#Implementations)):

	vector<int> PreorderTraversal(TreeNode *root) {
		vector<int> trace;
		stack<TreeNode*> st;
		while(root!=NULL) {
			trace.push_back(root->val); // Visit the root
			if(root->left!=NULL) {	// Traverse the left subtree
				TreeNode* tmp=root;
				root=root->left;
				if(tmp->right!=NULL)
					st.push(tmp->right);	// store the root of the right subtree
			} else if(root->right!=NULL) {
				root=root->right;
			} else {
				if(st.empty()) {
					root=NULL;
				} else {
					root=st.top();
					st.pop();
				}
			}
		} // end of while
		return trace;
	}

### Inorder Traversal

To traverse a binary tree in Inorder, following operations are carried-out:
1. Traverse the leftmost subtree starting at the left external node,
2. Visit the root, and 
3. Traverse the right subtree starting at the left external node.

For the example binary tree {1,2,3,#,#,4,#,#,5}, the inorder traversal is:

```
2,1,4,5,3
```

Following is my C++ implementation of inorder traversal([Pseudo Code](http://en.wikipedia.org/wiki/Tree_traversal#Implementations)):

	vector<int> InorderTraversal(TreeNode *root) {
		vector<int> trace;
		stack<TreeNode> st;
	    while(root!=NULL) {
			// Find the left-most node
			if(root->left!=NULL) {
				TreeNode tmp=*root;
				root=root->left;
				tmp.left=NULL;
				st.push(tmp);	// store the root of the right subtree
			} else if(root->right!=NULL) {
				// Handle the right subtree
				trace.push_back(root->val); // Visit leftmost node
				root=root->right;
			} else {
				trace.push_back(root->val); // Visit leaf/root node
				if(st.empty()) {
					root=NULL;
				} else {
					root=&(st.top());
					st.pop();
				}
			}
		}
		return trace;
	}

### Postorder Traversal

To traverse a binary tree in Postorder, following operations are carried-out: 
1. Traverse all the left external nodes starting with the leftmost subtree which is then followed by bubble-up all the internal nodes, 
2. Traverse the right subtree starting at the left external node which is then followed by bubble-up all the internal nodes, and 
3. Visit the root.

For the example binary tree {1,2,3,#,#,4,#,#,5}, the postorder traversal is:

```
2,5,4,3,1
```

Following is my C++ implementation of postorder traversal([Pseudo Code](http://en.wikipedia.org/wiki/Tree_traversal#Implementations)):

	vector<int> PostorderTraversal(TreeNode *root) {
		vector<int> trace;
		stack<TreeNode> st;
	    while(root!=NULL) {
			// Find the left-most node
			if(root->left!=NULL) {
				TreeNode tmp=*root;
				root=root->left;
				tmp.left=NULL;
				st.push(tmp);	// store the root of the right subtree
			} else if(root->right!=NULL) {
				TreeNode tmp=*root;
				root=root->right;
				tmp.left=NULL;
				tmp.right=NULL;
				st.push(tmp);	// store the root
			} else {
				trace.push_back(root->val);	// Print root at last
				if(st.empty()) {
					root=NULL;
				} else {
					root=&(st.top());
					st.pop();
				}
			}
		}
		return trace;
	}

### Breadth First Traversal(BFT)

Breadth first traversal is used for printing the binary tree in my code. The key of for BFT is to use a queue(s) to store nodes in current and/or next layers. The simplest way to use two queues: one to store nodes of current layer and the other for next layer. 

However, a single queue also can work well. For the first layer, there is only one node - the root. When accessing this node, count the number of nodes in the second layer and push them into queue. When accessing the second layer, pop items in FIFO sequence and remove them from queue, and push the nodes in third layer into queue, and so on. Two inteher variables can be used to count the numbers of nodes in current layer and the next layer, respectively.

Following is my C++ impementation of traversing binary tree in level order:

	void PrintTree(TreeNode *root) {
		queue<TreeNode*> q;
		TreeNode* tmp=root;
		q.push(tmp);
		int nodes_cur_layer=1; // # of nodes in current layer
		int nodes_next_layer=0;	// # of nodes in next layer
		while(nodes_cur_layer>0) {
			for(int i=0;i<nodes_cur_layer;++i) {
				tmp=q.front();
				q.pop();
	
				cout<<tmp->val<<" ";
	
				if(tmp->left!=NULL) {
					q.push(tmp->left);
					nodes_next_layer++;
				}
				if(tmp->right!=NULL) {
					q.push(tmp->right);
					nodes_next_layer++;
				}
			}
			cout<<endl;
			nodes_cur_layer=nodes_next_layer;
			nodes_next_layer=0;
		}
	}

### References
1. https://oj.leetcode.com/problems/binary-tree-inorder-traversal/
2. [Binary tree traversal: Preorder, Inorder, and Postorder](http://datastructuresnotes.blogspot.com/2009/02/binary-tree-traversal-preorder-inorder.html)
3. [Tree Traversal](http://en.wikipedia.org/wiki/Tree_traversal)
