---
layout: post
title: Longest Palindromic Substring  
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

The [Longest Palindromic Substring problem](https://oj.leetcode.com/problems/longest-palindromic-substring/) is:

> Given a string S, find the longest palindromic substring in S. You may assume that the maximum length of S is 1000, and there exists one unique longest palindromic substring. 

To solve this problem, first investigate following examples:

- `abcdefedchz`: the longest palindrome is `cdefedc`, the left and right parts are symmetric to letter `f`.
- `acbbcae`: the longest palindrome `acbbca`, which is symmetric to the (virtual) postion between `bb`.
- `a`: the longest palindrome is the letter `a`.

Based on above investigations, we could summarize the solution as: 

1. For every letter in the string, find the longest repetitive characters to current letter, record the `start` and `end` position;
2. Minus `start` position by one and see if `s[start]` is still identical to `s[end]`.
3. If yes, iteratively compare `s[start--]` and `s[end--]` until they are not equal.
4. The possible longest palindrome for current letter is `end-start-1`.

Following is my C++ implementation. This algorithm could be enhanced by not checking every letters, however, the time complexity will still be \\( O(N^2) \\). A better method is [Manacher's Algorithm](http://en.wikipedia.org/wiki/Longest_palindromic_substring), which has  \\( O(N) \\) time complexity.

	class Solution {
	public:
	    string longestPalindrome(string s) {
			if(s.empty())
				return "";
	
	        int pd_start=0; // the start position of the palindrome
			int pd_len=0; // the length of the palindrome
			for(int i=0;i<s.length();++i) {
				int start=i;
				int end=i;
				while(end<s.length() && s[start]==s[end])
					end++;
	
				start--;
				while(start>=0 && end<s.length() && s[start]==s[end]) {
					start--;
					end++;
				}
				
				if(pd_len<end-start-1) {
					pd_len=end-start-1;
					pd_start=start+1;	
				}
			}
	
			return s.substr(pd_start,pd_len);		
	    }
	};


<p>_If you find this post helpful, please click the Google Ads on the right(you don't need to buy anything, just a click will help). Many Thanks\!_</p>
