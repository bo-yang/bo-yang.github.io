---
layout: post
title: Linked List Cycle Problems
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

The [Linked List Cycle problem](https://oj.leetcode.com/problems/linked-list-cycle/) is:

> Given a linked list, determine if it has a cycle in it. Can you solve it without using extra space?

A [follow up problem](https://oj.leetcode.com/problems/linked-list-cycle-ii/) is:

> Given a linked list, return the node where the cycle begins. If there is no cycle, return `null`.Can you solve it without using extra space? 

The first problem can be solved by the **[Floyd’s Cycle-Finding Algorithm](http://www.geeksforgeeks.org/write-a-c-function-to-detect-loop-in-a-linked-list/)**:

> Traverse linked list using two pointers: move one pointer by *one* and other pointer by *two*.  If these pointers meet at some node then there is a loop.  If pointers do not meet then linked list doesn’t have loop.

As for the second problem, it is also based on the **Floyd’s Cycle-Finding Algorithm**. However, after detecting loop, we need to find the starting point of the cycle. **The most convinient way is to use two pointers, one points to the head node, and the other points to the node that pointers meet when detecting loop. Then move both the two pointers by one respectively in each iteration, until they meet. The node they meet is the starting point of the cycle.**

Here is the detailed explanation of above trick(reference: http://www.cnblogs.com/TenosDoIt/p/3416702.html). 

1. Assume the length(number of nodes, the same below) of the list is `l`, the distance(number of nodes, the same below) between `head` and the starting point of cycle is `a`, the distance between the cycle-starting point and the point where the slow/fast pointers meet is `b`, and the length of the cycle is `r`.
2. Assume when the fast/slow pointers meet, the slow pointer has accessed `s` nodes. Then the fast pointer has moved `2s`, and `s=a+b`.
3. Since the nodes accessed by the fast pointer also calculated as `kr+s`, where `k` is the number of cycles and \\( k \ge 1 \\). We have `2s=kr+s`, i.e. `s=kr`.
4. Since `s=a+b` and `s=kr`, we have `a+b=kr=(k-1)r+r`. Since `r=l-a`, we have `a+b=(k-1)+l-a`, i.e. `a=(k-1)+l-a-b`. Notice that `l-a-b` is the distance from the point where slow/fast pointers meet to the starting point of cycle(note: we only can go through the cycle of the linked list in one direction).
5. Since `a` is the distance between `head` node and the starting point of cycle, `a=(k-1)+l-a-b` means that the distance between `head` node and the starting point of cycle(`a`) *equals* to the distance from the point where slow/fast pointers meet to the starting point of cycle(`l-a-b`) plus several loops of the cycle.
6. Therefore, when we use two pointers to find the starting point of cycle, one pointer moves to the start point from the head of the list, while the other pointer winds aroung the cycle at the same pace. When the two pointers meet, one pointer must have moved `a` and the other moved `(k-1)+l-a-b`.

Following is my C++ implmentation.

	class Solution {
	public:
		ListNode *detectCycle(ListNode *head) {
			if(head==NULL || head->next==NULL)
				return NULL;
	
			ListNode* first=head;
			ListNode* second=head;
	        while(first!=NULL && second!=NULL && second->next!=NULL) {
				first=first->next;
				second=second->next->next;
	
				if(first==second)
					break;
			}
	
			if(first==second) {
				// Continue walking X steps
				first = head;
				while(first!=second)
				{
		            first = first->next;
		            second = second->next;
		        }
		
				return second;
			} else {
				return NULL;
			}
	    }
	
		// Create list with cycle using given vector of integers.
		// 	lb - the beginning index(in vec) of the cycle.
		ListNode* createList(vector<int>& vec, int lb) {
			head=new ListNode(vec[0]);
			ListNode* node=head;
			ListNode* begin=NULL;
			for(int i=1;i<vec.size();++i) {
				node->next=new ListNode(vec[i]);
				node=node->next;
				if(i==lb)
					begin=node;
			}
			if(lb>0 && lb<vec.size())
				node->next=begin;
			else if(lb>=vec.size()) // index out of upper bound, set to NULL
				node->next=NULL;
			else
				node->next=head; // index out of lower bound, linked to the head
	
			return head;
		}
	
		void printList(ListNode *head) {
			ListNode* node=head;
			while(node!=NULL) {
				cout<<node->val<<" ";
				node=node->next;
			}
			cout<<endl;
		}
	
		Solution() {head=NULL;}
	
		~Solution() {
			ListNode* node=head;
			while(head!=NULL){
				head=head->next;
				delete node;
				node=head;
			}
		}
	
	private:
		ListNode* head;
	
	};
