---
layout: post
title: String Permutation
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

The _string permutation_ problem aims to find all the _permutations_ of a string(re-arrangement of characters in this string). A string of length `n` has `n!` permutations.

The essence of string permutation is recursively swapping each letter with other letters in this string. Following image shows the permutation of string "ABC". For each step, fix the left part of the string, and swap two letters in the right part, until there is no letters left in the right part. 

<img src="http://d2o58evtke57tz.cloudfront.net/wp-content/uploads/NewPermutation.gif" width="678" height="277" class="aligncenter size-full wp-image-118925"></img>

Based on above strategy, the following two recusive algorithm could be implemented.

Algorithm 1(Reference: [Write a C program to print all permutations of a given string](http://www.geeksforgeeks.org/write-a-c-program-to-print-all-permutations-of-a-given-string/)): 

	void StrPermute(string& str, vector<string>& vec, int i) {
		if(i==str.length())
			vec.push_back(str);
		else {
			for(int j=i;j<str.size();++j) {
				std::swap(str[i],str[j]);
				StrPermute(str,vec,i+1);
				std::swap(str[i],str[j]);
			}
		} // end if
	}

Algorithm 2(Reference: [Find all the permutations of a string in Java](http://learnprogramming.machinesentience.com/java_permutations_recursion/)):

	void StrPermute(vector<string>& vec, string prefix, string str) {
		//cout<<"prefix="<<prefix<<", str="<<str<<endl; // TEST ONLY
		int n=str.length();
		if(n==0)
			vec.push_back(prefix);
		else {
			for(int j=0;j<n;++j)
				StrPermute(vec,prefix+str[j],str.substr(0,j)+str.substr(j+1));
		} // end if
	}

The permuted strings are stored as a vector of string. For both algorithms, the time complexities are `O(n*n!)`.
<p>_If you find this post helpful, please click the Google Ads on the right(you don't need to buy anything, just a click will help). Many Thanks\!_</p>
