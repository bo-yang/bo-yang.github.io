---
layout: post
title: Subset Sum Problem  
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

[The 3Sum problem](https://oj.leetcode.com/problems/3sum/) is:

> Given an array `S` of n integers, are there elements `a, b, c` in `S` such that `a + b + c = 0`? Find all unique triplets in the array which gives the sum of zero.
>
> Note:
>
> * Elements in a triplet `(a,b,c)` must be in non-descending order. (ie, `a ≤ b ≤ c`)
> * The solution set must not contain duplicate triplets.

Inspired by the [Wikipedia solution](http://en.wikipedia.org/wiki/3SUM), I implemented following solution. The first step is sorting the array `S` in ascending order, so that we can guarantee the front elements in the array are not less than the elemts at the end of the array. Then for every number in array, we can use the two variables to go through the whole array to find the triplets summed to 0. 

In order to guarantee there are no duplicate triplets, a hash table is used to record the triplets that have been found. Before inserting the newly found triplet into the solution matrix(`vector<vector<int> >`), try to look for the triplet in the hash table first. Since C++ doesn't support `vector<int>` keys, I formulate string keys using the integer triplets in the following code.

	class Solution {
	public:
	    vector<vector<int> > threeSum(vector<int> &num) {
			// Sort the vector
			std::sort(num.begin(), num.end());
			
			// Find triplets
			unordered_map<string,bool> map; // store found triplets
			char* buf=new char[100];
			for(int i=0;i<num.size();++i) {
				vector<int> triplet;
				int hi=num.size()-1;
				int lo=0;
				while(hi>i && lo<i) {
					if(num[i]+num[hi]+num[lo]==0) { // found a triplet
						sprintf(buf,"%d%d%d",num[i],num[hi],num[lo]);
						string str(buf);
						unordered_map<string,bool>::const_iterator got=map.find(str);
						if(got==map.end()) { // a brand new triplet
							triplet.push_back(num[lo]);
							triplet.push_back(num[i]);
							triplet.push_back(num[hi]);
							map[str]=true;
							sol.push_back(triplet);
						}
						triplet.clear();
						hi--;lo++;
					} else if(num[i]+num[hi]+num[lo]>0) {
						hi--;
					} else { // <0
						lo++;
					}
				}
			}
			delete [] buf;
	
			return sol;
	    }
		
	private:
		vector<vector<int> > sol;
	};

