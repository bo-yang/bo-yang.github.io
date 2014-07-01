---
layout: post
title: Longest Consecutive Sequence  
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

[The problem](https://oj.leetcode.com/problems/longest-consecutive-sequence/) is:

> Given an unsorted array of integers, find the length of the longest consecutive elements sequence.
>
> For example,
>
> Given [100, 4, 200, 1, 3, 2],the longest consecutive elements sequence is [1, 2, 3, 4]. Return its length: 4.
>
>Your algorithm should run in \\( O(n) \\) complexity.

The most intuitive solution to this problem is sorting the array first, and then go through the sorted array and record the length of the longest consecutive sequence. However, the time complexity of this method is \\(O(n \log n)\\). 

In order to get the \\( O(n) \\) complexity, we can pick up the first element in array(say `num[0]`), and then check if `num[0]-1` and `num[0]+1` are in the array. If yes, iteratively check the upper/lower boundary of this sequence. Until the upper/lower boundaries never change, find a new element in the array that has not been accessed and start a new search. 

In this solution, a hash table(`unordered_map` in C++) is needed to record the accessibility of each element in the array. The keys to the hash table are the integeres in the array, and the values are bool - if the key has been accessed, then set to `false`; otherwise set to `true`. In average, checking the existence(in C++, the `find()` of `unordered_map`) of a key in the hash table costs constant time. Since we only access every element in the array(or hash table) for only one time, and the operations for each element only cost \\(O(1)\\) time, the time complexity of this method is \\( O(n) \\). And the space complexity is also \\( O(n) \\) due to the hash table.

Following is my C++ implementation.

<pre>
class Solution {
public:
    int longestConsecutive(vector<int> &num) {
        unordered_map<int,bool> map;
		for(auto& x:num)
			map[x]=true;
		
		int start=num[0];
		int end=num[0];
		int max_len=1;
		int cnt=1; // items have been accessed
		int idx=1;
		while(cnt<=num.size()) {
			//cout<<"start="<<start<<", end="<<end<<endl; // TEST ONLY
			map[start]=false;
			map[end]=false;
			// Find the lower boundary
			unordered_map<int,bool>::iterator got=map.find(start-1);
			if(got!=map.end() && got->second) { 
				start--;
				cnt++;
			}
			
			// Find the upper boundary
			got=map.find(end+1);
			if(got!=map.end() && got->second) { 
				end++;
				cnt++;
			}

			if(max_len<end-start+1)
				max_len=end-start+1;

			if(!(map[start]||map[end])) { 
				// If both start and end rich the boundaries, 
				// find a new entry for them.
				while(idx<num.size() && map[num[idx]]==false)
					idx++;
				start=num[idx];
				end=num[idx];
				map[start]=false;
				cnt++;
			}
		}

		return max_len;
    }
};
</pre>
