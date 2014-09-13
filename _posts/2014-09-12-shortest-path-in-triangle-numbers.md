---
layout: post
title: Find The Shortest Path In Triangle Numbers
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

## Problem

An example of triangle of numbers is:

	      1
	     / \
	    2   5
	   / \ / \ 
	  9   4  33
	 / \ / \/  \ 
	11  6  99   0

In each row (except for the last), the numbers are adjacent to two numbers on the row below it. For example, 9 is adjacent to 11 and 6.

Imagine drawing a path connecting adjacent numbers from top to bottom. Take the sum of the numbers in this path. The lowest possible sum you could arrive at for this triangle is 13 (1 + 2 + 4 + 6).

Write a program which will read a file describing a triangle such as this one, and determine the lowest possible sum of numbers which connect the top and the bottom. The input will be a tab-separated list of numbers.

## Solution

The lowest sum can be found recursively by choosing the child node with the smallest sum. This greedy strategy should work for both positive and negative numbers, because every possible combination of nearby numbers are considered in the recursion(considering the connections of numbers in two nearby layers). 

The recursion function is:

	OPT(i,j)=tnum(i,j)+min(OPT(i+1,j),OPT(i+1,j+1))

where `OPT(i,j)` is the lowest sum of number `tnum[i][j]`. Above function can be implemented by dynamic programming.

The data structure for triangle numbers can be:

	// Data structure of Triangle numbers.
	class TriangNum {
	public:
		TriangNum();	// default constructor
		TriangNum(string file);	// constructor
		~TriangNum(){}
	
		const vector<vector<int> >& LoadData(string file); // Build a binary tree from Triangle Numbers
		int ShortestPath();	// Find the path of lowest possible sum of numbers
		void PrintNum() const;
		void PrintPath() const;
	
	private:
		vector<vector<int> > tnum;	// triangle numbers
		vector<vector<int> > trace; // all possible sums in the shortest path
		vector<int> spath; // shortest path
	
		vector<int> ReadNumbers(string line); // Read numbers in a line
		int Min(int x, int y) { return x<y?x:y; }
	};

And the code for finding the shortest path is:

	int TriangNum::ShortestPath() {
		// Find the lowest possible sum of numbers.
		trace=tnum;
		for(int i=tnum.size()-2;i>=0;--i) {
			for(int j=tnum[i].size()-1;j>=0;--j) {
				trace[i][j]=tnum[i][j]+Min(trace[i+1][j],trace[i+1][j+1]);
			}
		}
	
		// Find the shortest path
		spath.push_back(tnum[0][0]);
		int idx=0;
		for(int i=1;i<tnum.size();++i) {
			if(trace[i][idx]<trace[i][idx+1]) {
				spath.push_back(tnum[i][idx]);
			} else {
				idx+=1;
				spath.push_back(tnum[i][idx]);
			}
		}
	
		return trace[0][0];
	}

## Complexity

Given a triangle number with n lines, then:

 - Time complexity for finding the lowest sum of numbers: \\( 1+2+ \cdots +n = ( O(n^2) \\).
 - Time complexity for finding the shortest path: \\( O(n) \\).
 - Space complexity for above code is \\( O(n^2) \\), because the sums from bottom to the root of each node are recorded in an matrix.

 ## Code

 For the complete code of this problem, please refer to [https://github.com/bo-yang/ProgrammingChallenges/tree/master/triangle_numbers](https://github.com/bo-yang/ProgrammingChallenges/tree/master/triangle_numbers).
