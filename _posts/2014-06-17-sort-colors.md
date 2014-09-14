---
layout: post
title: Sort Colors
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

[Problem](https://oj.leetcode.com/problems/sort-colors/):

> Given an array with n objects colored red, white or blue, sort them so that objects of the same color are adjacent, with the colors in the order red, white and blue.
> 
> Here, we will use the integers 0, 1, and 2 to represent the color red, white, and blue respectively.

This problem is trivial when using two-pass counting sort: first pass count the numbers of each color, and then assign each element in array into corresponding regions in the array in the second pass.

The follow up of this problem requires to design a one-pass algorithm using constant space. The strategy is go through all the elements in the array, and move red colors(0s) to the left and move blue colors to the right(2s). The green colors(1s) will finally remain in the middle part of the array.

In order to implement above strategy, three indices can be used: one index for current element, one idex for the right-most red color, and one index for the left-most blue color. When meeting a red color, move it to the end of the red colors. When meeting a blue color, move it to the front(left-most) of the blue colors. And when meeting a green color, just increase current index.

Following is my C++ implementation:

	class Solution {
	public:
	    void sortColors(int A[], int n) {
			int idx=0,ridx=0,bidx=n-1; // ridx - index for red color, bidx - index for blue color
	        while(idx<=bidx) {
				switch(A[idx]) {
					case 0: { // Red color, move it to left
						while(A[ridx]==0 && ridx<n)
							++ridx;
						if(idx>ridx){
							swap(A[idx],A[ridx]);
							++ridx;
						} else {
							idx=ridx;
						}
						break;
					}
					case 1: { // Green color, skip it
						++idx;					
						break;
					}
					case 2: { // Blue color, move it to right
						while(A[bidx]==2 && bidx>idx)
							--bidx;
						swap(A[idx],A[bidx]);
						--bidx;
						break;
					}
					default:
						cout<<"Error: invalid color detected"<<endl;
				}
			} // end while
	    }
	};

And the following code can be used for test:

	#include <iostream>
	#include <utility>
	#include <cstdlib>
	using namespace std;
	
	int main() {
		/* initialize random seed: */
		srand (time(NULL));
		
		const int N=10;
		Solution sol;
		int* A=new int[N];
		cout<<"Before sorting:"<<endl;
		for(int i=0;i<N;++i) {
			A[i]=rand()%3;
			cout<<A[i]<<" ";
		}
		cout<<endl;
	
		sol.sortColors(A,N);
		
		cout<<"After sorting:"<<endl;
		for(int i=0;i<N;++i)
			cout<<A[i]<<" ";
		cout<<endl;
	
	
		delete [] A;
	}
<p>_If you find this post helpful, please click the Google Ads on the right(you don't need to buy anything, just a click will help). Many Thanks\!_</p>
