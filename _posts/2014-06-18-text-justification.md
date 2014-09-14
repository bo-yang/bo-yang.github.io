---
layout: post
title: Text Justification
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

[The Problem](https://oj.leetcode.com/problems/text-justification/):

>Given an array of words and a length L, format the text such that each line has exactly L characters and is fully (left and right) justified.
>

>You should pack your words in a greedy approach; that is, pack as many words as you can in each line. Pad extra spaces ' ' when necessary so that each line has exactly L characters.
>

>Extra spaces between words should be distributed as evenly as possible. If the number of spaces on a line do not divide evenly between words, the empty slots on the left will be assigned more spaces than the slots on the right.
>

>For the last line of text, it should be left justified and no extra space is inserted between words.
>

>For example, words: ["This", "is", "an", "example", "of", "text", "justification."] 
>L: 16.

>Return the formatted lines as:
<pre>
[
   "This    is    an",
   "example  of text",
   "justification.  "
]
</pre>

>Note: Each word is guaranteed not to exceed L in length.

This problem can be solved in two steps in general: (1) count the number of words that can be placed in current line, and then (2) allocate spaces. 

For the first step, since the words should be packed in a greedy approach, what we need count is the sum of characters in the \\( N \\) words plus the default \\( N-1 \\) spaces between the N words. Be careful to the last word in this line.

For the second step, we should treat two types of lines differently. Assume there are totally \\( M \\) spaces should be inserted in a line. For the lines other than the last line, insert \\( M/(N-1) \\) spaces between two nearby words if \\( M \\) can be divided by \\( N-1 \\)(where \\( N \\) is the number of words in this line). If \\( M \\) cannot be divided by \\( N-1 \\), add 1 space for the first \\(M \mod (N-1) \\) words. For the last line, put 1 space between two words, and put other spaces to the end of the last word.  

Following is my C++ implementation of text justification:

<pre>
class Solution {
public:
    vector<string> fullJustify(vector<string> &words, int L) {
        vector<string> ret;
		int word_cnt=0; // count words that have been processed
		
		while(word_cnt < static_cast<int>(words.size())) {
			// count words that can be placed in current line
			int char_cnt=0; // characters justified in a line
			vector<string> vec;
			int idx=word_cnt;
			while(char_cnt+static_cast<int>(vec.size())-1<=L && idx<static_cast<int>(words.size())) {
				char_cnt+=static_cast<int>(words[idx].size());
				vec.push_back(words[idx]);
				idx++;
			}
			if(char_cnt+static_cast<int>(vec.size())-1>L) {
				char_cnt-=vec.back().size();
				vec.pop_back(); // remove the last word because it surpasses the line limit
			}
			word_cnt+=vec.size();

			// assign spaces for this line
			string line;
			int total_spaces=L-char_cnt;
			int avg_spaces; // average spaces for 
			int left_spaces; //spaces left
			if(word_cnt!=static_cast<int>(words.size())) {
				if(static_cast<int>(vec.size())>1) { // This line contains more than 1 word
					avg_spaces=total_spaces/(static_cast<int>(vec.size())-1);
					left_spaces=total_spaces%(static_cast<int>(vec.size())-1);
					// Formulate strings per line
					for(int i=0;i<static_cast<int>(vec.size())-1;++i) {
						line+=vec[i];
						for(int j=0;j<avg_spaces;++j)
							line+=" ";
						if(left_spaces>0) {
							line+=" ";
							left_spaces--;
						}
					}
					line+=vec.back(); // put the last word to the rightmost position
				} else { // Only one word
					avg_spaces=0;
					left_spaces=total_spaces;
					line+=vec.back();
					while(left_spaces>0) {
						line+=" "; // append spaces to the last word
						--left_spaces;
					}
				}
			} else { 
				// The last line
				avg_spaces=1;
				left_spaces=total_spaces-static_cast<int>(vec.size())+1;
				for(int i=0;i<static_cast<int>(vec.size())-1;++i) {
					line+=vec[i];
					line+=" ";
				}
				line+=vec.back();
				while(left_spaces>0) {
					line+=" "; // append spaces to the last word
					--left_spaces;
				}
			}
			ret.push_back(line);
		}

		return ret;
    }
};
</pre>
<p><i>If you find this post helpful, please click the Google Ads on the right(you don't need to buy anything, just a click will help). Many Thanks.</i></p>
