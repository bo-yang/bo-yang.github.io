---
layout: post
title: Binary Tree Operations(I)
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

There are two kinds of traversal for (binary) trees: Depth First Search(DFS) and Breadth First Search(BFS). Depth First Search visits the tree by proceeding deeper and deeper until it reaches the leaf nodes, including pre-order, in-order, post-order tree traversal. DFS uses a data structure called Stack.

Breadth First Search is the most natural solution for level-order traversal, since it visits the nodes level by level. BFS requires the use of a data structure called Queue, which is a First In First Out (FIFO) structure.

For the complete C++ source code used in this article, please refer to: [https://github.com/bo-yang/BinaryTree](https://github.com/bo-yang/BinaryTree).

1. [Data Structure](#data_structure)
2. [Build Binary Tree](#build_tree)
3. [Preorder Traversal](#preorder)
4. [Inorder Traversal](#inorder)
5. [Postorder Traversal](#postorder)
6. [Breadth First Traversal](#bft)
7. [Zigzag Traversal](#zigzag)
8. [Binary Tree Comparison](#comparison)


### <a name="data_structure">Data Structure</a>

The data structure used for the tree node is very simple, which is adopted from [LeetCode OJ](https://oj.leetcode.com/problems/binary-tree-inorder-traversal/):

	struct TreeNode {
	    int val;
	    TreeNode *left;
	    TreeNode *right;
	    TreeNode(int x) : val(x), left(NULL), right(NULL) {}
	};

<strike>The constructing function TreeNode() is added to purely because it is easier to dynamically allocate TreeNode array by `new TreeNode[size]`. C++98 doesn't support specifying default arguments for objects allocated by `new`. However, the TreeNode array can simplify the process of building binary trees.</strike> 

The data structure for the binary tree is:

	class BinaryTree {
	public:
		BinaryTree() { root=NULL;layers=0; }
		BinaryTree(vector<string>& t) { BuildTree(t); }
		~BinaryTree();
		TreeNode* GetRoot() { return root; }
		vector<int> PreorderTraversal(TreeNode *root);
		vector<int> InorderTraversal(TreeNode *root);
		vector<int> PostorderTraversal(TreeNode *root);
		vector<vector<int> > ZigzagLevelOrder(TreeNode *root);
		void PrintTraversal(vector<int>& vec, string type);
		void PrintTraversal(vector<vector<int> >& vec, string type);
		TreeNode* BuildTree(vector<string>& t);
		void PrintTree(TreeNode *root);
		bool IsSameTree(TreeNode *p, TreeNode *q);

	private:
		TreeNode* root;
		int layers;	// number of layers
	};


### <a name="build_tree">Build Binary Tree</a>

In my implementation, binary trees are built from a vector of strings. For example, given sequence {1,2,3,#,#,4,#,#,5}, the following binary tree can be built:

<pre>
	  1
	 / \
	2   3
	   /
	  4
	 / \
	5   6
</pre>

The tree is built layer by layer, where the first element in sequence is viewed as the root node. Here the "#" is used as terminator. 

For each node other than the last layer, there are two children in the next layer, either a number(or string) or a "#"(invalid node). When parsing nodes in current layer, count the number of elements in next layer. The exception is for "#"s - they have no child nodes. In this way, we can parse the dependency of nodes layer by layer. 

Following is the C++ code for building binary trees:

	TreeNode* BuildTree(vector<string>& t) {
		if(t.empty())
			return NULL;

		root=new TreeNode(-1);
		TreeNode* tree=root;
		queue<TreeNode*> q; // store nodes of next layer
		q.push(tree);

		// Build binary tree from vector of strings, where # denotes an invalid node.
		// The main idea is to parse vector of strings(nodes) layer by layer. 
		// The first item in the vector should be the root node, and the number of
		// first layer is one. When parsing the first layer, count the nodes of the
		// second layer, and so on.
		int idx=0;
		int nodes_cur_layer=1;
		vector<string>::iterator it=t.begin(); 
		while(idx<t.size()){
			int nodes_next_layer=0; // all nodes, including #s
			int vi=0;	// index of valid nodes in next layer
			for(int i=0;i<nodes_cur_layer;++i) {
				if(idx+i<t.size() && *(it+i)=="#") { // Skip #s
					continue;
				}

				tree=q.front();
				q.pop();

				int left=nodes_cur_layer+nodes_next_layer;
				int right=nodes_cur_layer+nodes_next_layer+1;
				if(idx+left<t.size()) { // in case of out-of-boundary access
					if(*(it+left)!="#" ) {
						tree->left=new TreeNode(-1);
						q.push(tree->left);
						vi++;
					} else {
						tree->left=NULL;
					}
					nodes_next_layer++;
				}
				if(idx+right<t.size()) { // in case of out-of-boundary access
					if(*(it+right)!="#") {
						tree->right=new TreeNode(-1);
						q.push(tree->right);
						vi++;
					} else {
						tree->right=NULL;
					}
					nodes_next_layer++;
				}

				tree->val=atoi((it+i)->c_str());
			}

			idx+=nodes_cur_layer;
			it+=nodes_cur_layer;
			nodes_cur_layer=nodes_next_layer;
			layers++;
		}

		return root;	// root of the tree
	}

In order to delete the memroy dynamically allocated in function BuildTree(), in the desctructor, every node in the tree should be traversed and deleted. For simplicity, layer-by-layer traversal is used.

	~BinaryTree() {
		if(root!=NULL) {
			// delete allocated nodes iteratively
			queue<TreeNode*> q;
			TreeNode* tmp=root;
			q.push(tmp);
			int nodes_cur_layer=1; // # of nodes in current layer
			int nodes_next_layer=0;	// # of nodes in next layer
			while(nodes_cur_layer>0) {
				for(int i=0;i<nodes_cur_layer;++i) {
					tmp=q.front();
					q.pop();

					if(tmp->left!=NULL) {
						q.push(tmp->left);
						nodes_next_layer++;
					}
					if(tmp->right!=NULL) {
						q.push(tmp->right);
						nodes_next_layer++;
					}

					delete tmp;
				}
				nodes_cur_layer=nodes_next_layer;
				nodes_next_layer=0;
			}
		}
	}


### <a name="preorder">Preorder Traversal</a>

To traverse a binary tree in Preorder, following operations are carried-out 
1. Visit the root, 
2. Traverse the left subtree, and 
3. Traverse the right subtree. 

For the example binary tree {1,2,3,#,#,4,#,5,6}, the preorder traversal is:

```
1,2,3,4,5,6
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

### <a name="inorder">Inorder Traversal</a>

To traverse a binary tree in Inorder, following operations are carried-out:
1. Traverse the leftmost subtree starting at the left external node,
2. Visit the root, and 
3. Traverse the right subtree starting at the left external node.

For the example binary tree {1,2,3,#,#,4,#,5,6}, the inorder traversal is:

```
2,1,5,4,6,3
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

### <a name="postorder">Postorder Traversal</a>

To traverse a binary tree in Postorder, following operations are carried-out: 
1. Traverse all the left external nodes starting with the leftmost subtree which is then followed by bubble-up all the internal nodes, 
2. Traverse the right subtree starting at the left external node which is then followed by bubble-up all the internal nodes, and 
3. Visit the root.

For the example binary tree {1,2,3,#,#,4,#,5,6}, the postorder traversal is:

```
2,5,6,4,3,1
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

### <a name="bft">Breadth First Traversal(BFT)</a>

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

### <a name="zigzag">Zigzag Traversal</a>

Zigzag level order traversal is a Breadth First Traversal(BFT). The procedure  of this traversal is (i) from left to right, (ii) then right to left for the next level, (iii) and alternate between). 

For the example binary tree {1,2,3,#,#,4,#,5,6}, the postorder traversal is:

```
[[1],[3,2],[4],[6,5]]
```

Although zigzag traversal is kind of BFT, it is not easy to use one or two queues to implement it, because you need to traverse every layer from left to right, and print nodes from right to left for some layers. Since I don't want to use three queues(one to store the nodes of current layer, the other two store the left-to-right traversal of next layer, but one of them should be used for left-to-right traveral in next iteration and the other used for right-to-left printing), I implemented the zigzag traversal with two lists: one list to store the left-to-right traversal of current layer, and the other list for storing left-to-right traversal of next layer. During printing, the same list can be controlled to output nodes either from left to right or right to left without loosing any information(unlike queue, quque is not iterable).

Following is my code for zigzag traversal. Note that the output is a vector of vectors.

	vector<vector<int> > ZigzagLevelOrder(TreeNode *root) {
        vector<vector<int> > trace;
		list<TreeNode*> lst_cur;
		int next_layer_nodes=0;
		int cur_layer_nodes;
		bool l2r=true;
		if(root!=NULL) {
			cur_layer_nodes=1;
			lst_cur.push_back(root);
		}else{
			return trace;
		}

		// Use two lists for left to right traversal and vice versa: 
		// one list used for storing nodes of current layer, and the 
		// other used for storing nodes layer below.
		//
		// The binary tree will always be traversed from left to right 
		// for each layer, however, for every other layer, the nodes will
		// be printed from right to left.
		while(cur_layer_nodes>0) {
			vector<int> subtr;
			list<TreeNode*> lst_next;
			list<TreeNode*>::iterator lr=lst_cur.begin();
			list<TreeNode*>::iterator rl=lst_cur.end();
			for(int i=0;i<cur_layer_nodes;++i) {
				TreeNode* tn = *(lr);
				if(l2r) {
					subtr.push_back((*(lr))->val);
				} else {
					rl--; // lst_cur.end() points to undefined memory
					subtr.push_back((*(rl))->val);
				}
				
				if(tn->left!=NULL) {
					next_layer_nodes++;
					lst_next.push_back(tn->left);
				}

				if(tn->right!=NULL) {
					next_layer_nodes++;
					lst_next.push_back(tn->right);
				}

				lr++;
			} // end for
			trace.push_back(subtr);
			cur_layer_nodes=next_layer_nodes;
			next_layer_nodes=0;
			lst_cur=lst_next;
			l2r=!l2r;
		} // end while

		return trace;
    }


### <a name="comparison">Binary Tree Comparison</a>

As for the comparison of two binary trees, only equality is considered. Two binary trees are considered equal if they are structurally identical and the nodes have the same value. To implement this, two trees should be traversed at the same time using same traverse method. If for every pair of nodes, the values are equal, and both of them have left child and right child, then the two nodes are considered equal.

For simplicity, Breadth First Traversal is used in my implementation:

	bool IsSameTree(TreeNode *p, TreeNode *q) {
		bool same_tree=true;
        queue<TreeNode*> qp;
		queue<TreeNode*> qq;
		int cur_layer_nodes;
		int next_layer_nodes=0;
		if(p!=NULL && q!=NULL) {
			cur_layer_nodes=1;
			qp.push(p);
			qq.push(q);
		} else if(p==NULL && q==NULL) {
			return true;
		} else {
			return false;
		}

		// Traverse trees layer by layer
		while(cur_layer_nodes>0) {
			next_layer_nodes=0;
			for(int i=0;i<cur_layer_nodes;++i) {
				TreeNode* tp=qp.front();
				qp.pop();
				TreeNode* tq=qq.front();
				qq.pop();

				if((tp->val!=tq->val) || (tp->left==NULL) && (tq->left!=NULL) || (tp->left!=NULL) && (tq->left==NULL) || (tp->right==NULL) && (tq->right!=NULL) || (tp->right!=NULL) && (tq->right==NULL)) {
					same_tree=false;
					break;
				}

				if(tp->left!=NULL) {
					qp.push(tp->left);
					qq.push(tq->left);
					next_layer_nodes++;
				}

				if(tp->right!=NULL) {
					qp.push(tp->right);
					qq.push(tq->right);
					next_layer_nodes++;
				}
			} // end for
			cur_layer_nodes=next_layer_nodes;
			if(!same_tree)
				break;
		} // end while

		return same_tree;
    }


_[Updated 05/30/2014]_

### References
1. https://oj.leetcode.com/problems/binary-tree-inorder-traversal/
2. [Binary tree traversal: Preorder, Inorder, and Postorder](http://datastructuresnotes.blogspot.com/2009/02/binary-tree-traversal-preorder-inorder.html)
3. [Tree Traversal](http://en.wikipedia.org/wiki/Tree_traversal)
<p><i>If you find this post helpful, please click the Google Ads on the right(you don't need to buy anything, just a click will help). Many Thanks.</i></p>
