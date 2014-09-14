---
layout: post
title: Merge K Sorted Lists
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

#### Contents

1. [Naive Method](#naive_method)
2. [Divide-and-Conquer Algorithm](#divide_conquer)
3. [Non-Recursive Method](#heap)

The [Merge `k` Sorted Lists](https://oj.leetcode.com/problems/merge-k-sorted-lists/) is:

> Merge k sorted linked lists and return it as one sorted list. Analyze and describe its complexity.

In this problem, the sorted lists are stored in a vector, i.e. `vector<ListNode*>`. Assume the longest list contains `n` elements.

### <a name="naive_method">Naive Method</a>

The naive method of merging `k` sorted lists is: iteratively compare the head elements of the `k` lists, find the smallest node, append this node to the merged list and forward that list by one step. Then repeat above step until all lists reach the end. Obviously, since in every iteration `k` comparisons are needed, and the longest list has `n` elements, therefore, the time complexity is \\( O(k^{nk}) \\) - the worst case is all lists have `n` elements, so for the total `nk` elements, totally \\( k^{nk} \\) comparisons are needed to merge them into one list.

### <a name="divide_conquer">Divide-and-Conquer Algorithm</a>

Other than exponential time complexity, a much better method is to use divide-and-conquer. Since merging two sorted lists is easy, we can recursively divide the `k` lists into two parts until there are no more than two lists. Then we can use a merge method similiar to the merge sort to merge the two lists together.

Assume the longest list contains `n` elements, the time required for dividing and merging lists is \\( T(k)=2T(k/2)+O(nk) \\). According to the Master Theorem, the time complexity is \\( O(nk \log k) \\). The space complexity of this algorithm is \\( O(1) \\).

Following is my C++ implementation of the divide-conquer method.

	class Solution {
	public:
	    ListNode *mergeKLists(vector<ListNode *> &lists) {
			if(lists.size()==0)
	            return NULL;
	        if(lists.size()==1)
	            return lists[0];
	
			return divideConquerLists(lists,0,lists.size()-1);
	    }
	
		ListNode* divideConquerLists(vector<ListNode *> &lists, int start, int end) {
			if(start+1<end) {
				int mid=(start+end)/2;
				ListNode* head1=divideConquerLists(lists,start,mid);
				ListNode* head2=divideConquerLists(lists,mid+1,end);
				return mergeTwoLists(head1,head2);
			} else if(start+1==end) {
				return mergeTwoLists(lists[start],lists[end]);
			} else if(start==end) {
				return lists[start];
			}
		}
	
		ListNode* mergeTwoLists(ListNode* head1, ListNode* head2) {
			if(head1==NULL)
				return head2;
			if(head2==NULL)
				return head1;
		
			ListNode* pre_head=new ListNode(-1);
			ListNode* pre=pre_head;
			ListNode* node1=head1;
			ListNode* node2=head2;
			while(node1!=NULL && node2!=NULL) {
				if(node1->val < node2->val) {
					pre->next=node1;
					node1=node1->next;
				} else {
					pre->next=node2;
					node2=node2->next;
				}
				pre=pre->next;
			}
			while(node1!=NULL) {
				pre->next=node1;
				node1=node1->next;
				pre=pre->next;
			}
			while(node2!=NULL) {
				pre->next=node2;
				node2=node2->next;
				pre=pre->next;
			}
			pre->next=NULL;
	
			ListNode* head=pre_head->next;
			delete pre_head;
	
			return head;
		}
	};

### <a name="heap">Non-Recursive Method</a>

In addition to the recursive divide-conquer method, there is also a non-recursive method. We can use a maximum `k`-element heap(in C++, `priority_queue`) to merge the `k` lists. The strategy is:

1. Create a heap to store `ListNode*`, which should sort list nodes in the ascending order or node values.
2. Insert the first node(head) of each list into the heap, so that we store all the `k` entries in the heap.
3. Get the top element in heap and add in to the merged list.
4. If the top element of heap is the last node in a list, pop it and go to step 3.
5. If the top element of heap has followers, pop it and push its following node into heap. 
6. Go to step 3 until the heap is empty.

To create a heap meet step'1 requirement, we need to create a structure/class that overloads the operator `()`, so that we can compare two list nodes based on their values. Such structure/class is required by the `priority_queue` template. Instead of a struct, I would rather use a lambda function to do so. Unfortunately, lambda function still cannot be used as the comparison object in `priority_queue`.

For a `k`-element `priority_queue`, an insertion requires \\( O(\log k) \\) time. Since we need to do such insertion for at most \\(nk\\) times, therefore the time complexity for this method is also \\( O(nk \log k) \\). Obviously, the space complexity of this algorithm is \\( O(k) \\).

	class Solution {
	public:
		// comparison structure, required by priority_queue template
		struct greaterListNode{ 
			bool operator() (ListNode* x, ListNode* y) {return x->val > y->val;}
		};
	
	    ListNode *mergeKLists(vector<ListNode *> &lists) {
			if(lists.size()==0)
				return NULL;
			if(lists.size()==1)
				return lists[0];
	
			ListNode* pre_head=new ListNode(-1);
			ListNode* pre=pre_head;
			std::priority_queue<ListNode*, vector<ListNode*>, greaterListNode> pq;
	
			// Insert entries of every list into heap
			for(auto& lst:lists)
				if(lst!=NULL)
					pq.push(lst);
	
			// Recursively parse every element in lists
			while(!pq.empty()) {
				pre->next=pq.top();
				pre=pre->next;
				pq.pop();
	
				if(pre!=NULL && pre->next!=NULL)
					pq.push(pre->next);
			}
	
			ListNode* head=pre_head->next;
			delete pre_head;
			return head;
	    }
	};

### References:

1. [Merge k Sorted Lists -- LeetCode](http://blog.csdn.net/linhuanmars/article/details/19899259)
2. [std::priority_queue](http://www.cplusplus.com/reference/queue/priority_queue/)
3. [std::greater](http://www.cplusplus.com/reference/functional/greater/)
<p><i>If you find this post helpful, please click the Google Ads on the right(you don't need to buy anything, just a click will help). Many Thanks.</i></p>
