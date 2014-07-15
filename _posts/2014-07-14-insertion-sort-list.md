---
layout: post
title: Insertion Sort List
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

The [Insertion Sort List problem](https://oj.leetcode.com/problems/insertion-sort-list/) is:

> Sort a linked list using insertion sort.

There is a very simple insertion sort method for arrays in Wikipedia.

	for i ← 1 to length(A)
	    j ← i
	    while j > 0 and A[j-1] > A[j]
	        swap A[j] and A[j-1]
	        j ← j - 1

The core of this algorithm is to find the position where to insert an smaller element. Compared to array, the list nodes cannot access nodes before them - they only know the nodes behind them. Therefore, instead of looking for insert positions decreasingly from current position down to the beginning, we can search the position from the head instead.

Following is my C++ implementation. Swapping two nodes is a little tricky, because you need to take care of four links.

	struct ListNode {
	    int val;
	    ListNode *next;
	    ListNode(int x) : val(x), next(NULL) {}
	};
	 
	class Solution {
	public:
	    ListNode *insertionSortList(ListNode *head) {
	        if(head==NULL||head->next==NULL)
				return head;
	
			ListNode* pre_head=new ListNode(-1); // to manipulate the head
			pre_head->next=head;
			ListNode* cur=head;
			while(cur->next!=NULL) {
				if(cur->val > cur->next->val) {
					ListNode* node=pre_head;
	
					// find position to insert
					while(node->next->val < cur->next->val)
						node=node->next;
	
					// swap nodes
					ListNode* tmp1=node->next;
					ListNode* tmp2=cur->next->next;
					node->next=cur->next;
					cur->next->next=tmp1;
					cur->next=tmp2;
				} else {
					cur=cur->next;
				}
			}
			
			head=pre_head->next;
			delete pre_head;
	
			return head;
	    }
	};

In addition to above insertion sort, which moves smaller elements forward, we also can move the larger elments backward. The following code is more like bubble sort.

    ListNode *insertionSortList(ListNode *head) {
        if(head==NULL||head->next==NULL)
			return head;

		// find the length of the list
		int len=0;
		ListNode* node=head;
		while(node!=NULL) {
			node=node->next;
			len++;
		}

		// iteratively insert the largest number to the end
		for(int i=len-1;i>=0;i--) {
			int j=0;
			node=head;
			ListNode* prev=NULL; // previous of node
			while(j<i && node!=NULL && node->next!=NULL) {
				if(node->val > node->next->val) {
					// swap nodes
					ListNode* tmp=node->next;
					node->next=node->next->next;
					tmp->next=node;
					if(j==0||prev==NULL) {
						head=tmp;
					} else {
						prev->next=tmp;
					}
					prev=tmp;
				} else {
					prev=node;
					node=node->next;
				}
				j++;
			}
		} // end for

		return head;
    }

