---
layout: post
title: Word Search Problem - Non-recursive Solution
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

The [Word Search](https://oj.leetcode.com/problems/word-search/) states:

> Given a 2D board and a word, find if the word exists in the grid.
> 
> The word can be constructed from letters of sequentially adjacent cell, where "adjacent" cells are those horizontally or vertically neighboring. The same letter cell may not be used more than once.
> 
> For example,
>
> Given board =
>
> [
>
>   ["ABCE"],
>
>   ["SFCS"],
>
>   ["ADEE"]
>
> ]

> * word = "ABCCED", -> returns `true`,
> * word = "SEE", -> returns `true`,
> * word = "ABCB", -> returns `false`.

The most popular solution to this problem is recursive, such as [http://yucoding.blogspot.com/2013/02/leetcode-question-124-word-search.html](http://yucoding.blogspot.com/2013/02/leetcode-question-124-word-search.html).

However, I tried to solve this problem without recursion. In my implementation, three stacks are used to record the possible coordinates in `board`, the character position in `word`(with respect to each candidate cell) and the cells have been accessed to each possible coordinates, repectively. 

The first step is finding the possible cells for the first char in `word`, and push coordinates, position of the char in `word`(here is 0) and accessed cells(here are matched cells) into stacks. 

The second step is exploring all the neighbors of cells in stack, and match each char in `word` respectively. For a match, push contexts into stack and try to match the next char. For a mismatch, pop the top of stacks and try another candidate. If the search could reach the end of the word, then we successfully find the word. If the stacks turns empty and there are still chars left, that means the seach fails.

Following is my C++ implementation of the non-recursive solution.

	class Solution {
	public:
	    bool exist(vector<vector<char> > &board, string word) {
			if(board.empty())
				return false;
	
			std::stack<std::pair<int,int>> st_coord; // coordinates of the board
			std::stack<int> st_char; // store characters of the words
			vector<std::pair<int,int> > vec;
			std::stack<vector<std::pair<int,int> > > st_access;
			bool found=false;
	        for(int l=0;l<word.length();) {
				if(l==0) { // Find the first character of word in board
					for(int i=0;i<board.size();++i) {
						for(int j=0;j<board[0].size();++j)
							if(board[i][j]==word[l]){
								st_coord.push(std::pair<int,int>(i,j));
								st_char.push(l);
								vec.push_back(std::pair<int,int>(i,j));
								st_access.push(vec);
								vec.pop_back(); // do not affect other candidates
								if(word.size()==1) 
									return true;
							}
					}
					l++;
				} else {
					if(st_coord.empty()) // cannot find the first character
						break;
	
					// Explore all possible cells to match each character of the word
					while(!st_coord.empty()) {
						int x=st_coord.top().first;
						int y=st_coord.top().second;
						vec=st_access.top();
						l=st_char.top()+1;
						if(l==word.length()) {
							found=true;
							break;
						}
						
						st_coord.pop();
						st_char.pop();
						st_access.pop();
	
						if(x-1>=0 && board[x-1][y]==word[l]) {
							vector<std::pair<int,int> >::iterator got=
								std::find(vec.begin(),vec.end(),std::pair<int,int>(x-1,y));
							if(got==vec.end()) {
								st_coord.push(std::pair<int,int>(x-1,y));
								st_char.push(l);
								vec.push_back(std::pair<int,int>(x-1,y));
								st_access.push(vec);
								vec.pop_back(); // do not affect other candidates
							}
						}
						if(x+1<board.size() && board[x+1][y]==word[l]) {
							vector<std::pair<int,int> >::iterator got=
								std::find(vec.begin(),vec.end(),std::pair<int,int>(x+1,y));
							if(got==vec.end()) {
								st_coord.push(std::pair<int,int>(x+1,y));
								st_char.push(l);
								vec.push_back(std::pair<int,int>(x+1,y));
								st_access.push(vec);
								vec.pop_back(); // do not affect other candidates
							}
						}
						if(y-1>=0 && board[x][y-1]==word[l]) {
							vector<std::pair<int,int> >::iterator got=
								std::find(vec.begin(),vec.end(),std::pair<int,int>(x,y-1));
							if(got==vec.end()) {
								st_coord.push(std::pair<int,int>(x,y-1));
								st_char.push(l);
								vec.push_back(std::pair<int,int>(x,y-1));
								st_access.push(vec);
								vec.pop_back(); // do not affect other candidates
							}
						}
						if(y+1<board[0].size() && board[x][y+1]==word[l]) {
							vector<std::pair<int,int> >::iterator got=
								std::find(vec.begin(),vec.end(),std::pair<int,int>(x,y+1));
							if(got==vec.end()) {
								st_coord.push(std::pair<int,int>(x,y+1));
								st_char.push(l);
								vec.push_back(std::pair<int,int>(x,y+1));
								st_access.push(vec);
								vec.pop_back(); // do not affect other candidates
							}
						}
					}
				} // end else
			} // end for l
	
			return found;
	    }
	};
<p><i>If you find this post helpful, please click the Google Ads on the right(you don't need to buy anything, just a click will help). Many Thanks.</i></p>
