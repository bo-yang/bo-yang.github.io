---
layout: post
title: Top N Numbers
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

The problem is:

> Given an arbitrarily large file and a number, \\(N\\), containing individual numbers on each line (e.g. 200Gb file), will output the largest \\(N\\) numbers, highest first. Analyse the run time/space complexity of your approach.

The intuitive method for the top-\\(N\\) is to sort all numbers first, and then return the first/last \\(N\\) numbers. However, the time complexity of sorting is usually \\(O(n \log n)\\), and when the data are very big, it is impossible to do the sorting in memory directly.

Finding the top \\(N\\) items can be done in \\(O(n \log N) \\) time, which is approximate to \\( O(n) \\). The key is to use a heap(in C++, it is *priority_queue*) to store the top \\(N\\) items during reading the huge data. [Heap](http://en.wikipedia.org/wiki/Heap_(data_structure)) is a specialized (binary) tree-based data structure that satisfies the *heap property*: either the keys of parent nodes are always greater than or equal to those of the children and the highest key is in the root node (this kind of heap is called max heap) or the keys of parent nodes are less than or equal to those of the children and the lowest key is in the root node (min heap).

The strategy is:

- maintain an \\(N\\)-item long heap, and read all items iteratively.
- after reading a new item from the huge data, compare it with the smallest number in heap.
- if the number is greater than the smallest number in heap, then pop the top number in heap(also is the smallest one) and push this data into heap.
- otherwise, continue and read a new data.
- finally, dump the \\(N\\) items in the heap.

Since the heap could mantain the *heap property* by iteself, inserting a new item won't disorder the heap. As inserting a new item to heap requires \\( O(\log N) \\) time, the worst case of processing \\(n\\) data costs \\(O(n \log N) \\) time.

Following is my C++ solution. The *push* function of *priority_queue* will invoke [two functions](http://www.cplusplus.com/reference/queue/priority_queue/push/): one call to push_back on the underlying container and one call to push_heap on the range that includes all the elements of the underlying container.

    class Numbers {
    	public:
    		Numbers(){}
    		Numbers(long len){ genNums(len); }
    		Numbers(string file){ readNums(file); }
    
    		vector<int>& topN(vector<int>& nums, int N);
    		vector<int>& topNbySort(vector<int>& nums, int N);
    		vector<int>& genNums(long len);
    		vector<int>& readNums(string file);
    		void printNums();
    		void printTopN(vector<int>& top_num);
    
    	private:
    		vector<int> nums;
    		vector<int> top_nums;
    };
    
    // Find the top N numbers by heap.
    vector<int>& Numbers::topN(vector<int>& nums, int N) {
    	priority_queue<int,vector<int>,greater<int> > pq; // top element is the minimum
    	pq.push(nums.front());
    	for(vector<int>::iterator it=nums.begin()+1;it!=nums.end();++it) {
    		if(pq.top()<*it) {
    			if(pq.size()<N) {
    				pq.push(*it);
    			} else {
    				pq.pop();
    				pq.push(*it);
    			}
    		}
    	}
    	while(!pq.empty()) {
    		top_nums.push_back(pq.top());
    		pq.pop();
    	}
    
    	return top_nums;
    }
    
    // Find the top N numbers using sort.
    vector<int>& Numbers::topNbySort(vector<int>& nums, int N) {
    	sort(nums.begin(),nums.end()-1);
    	auto rit=nums.crbegin();
    	for(int i=0;i<N;++i) {
    		top_nums.push_back(*rit);
    		rit++;
    	}
    	return top_nums;
    }
    
    void Numbers::printTopN(vector<int>& top_num) {
    	cout<<"The top "<<top_num.size()<<" numbers are:"<<endl;
    	for(auto& x:top_num)
    		cout<<x<<" ";
    	cout<<endl;
    }
    
    vector<int>& Numbers::genNums(long len) {
    	for(int i=0;i<len;++i)
    		nums.push_back(rand()%len+1);
    
    	return nums;
    }
    
    vector<int>& Numbers::readNums(string file) {
    	fstream fs(file,std::fstream::in);
    	if (fs.is_open()) {
    		while(!fs.eof()) {
    			int tmp;
    			fs>>tmp;
    			nums.push_back(tmp);
    		}
    	} else {
    		cerr<<"Failed to open file "<<file<<endl;
    	}
    	fs.close();
    	nums.pop_back(); // delete the last number, because it was read two times.
    	return nums;
    }
    
    void Numbers::printNums() {
    	for(auto& x:nums)
    		cout<<x<<" ";
    	cout<<endl;
    }



I tested the time costs of sorting and heap when handling 10,000,000 integers respectively. Obviously the heap method is 5 times faster than sort method.

<pre>
macmini:TopNumbers boyang$ time ./top_nums_sort 
The top 10 numbers are:
8507985 9999999 9999997 9999995 9999994 9999992 9999991 9999991 9999990 9999986 

real	0m6.646s
user	0m6.589s
sys	0m0.052s

macmini:TopNumbers boyang$ time ./top_nums_heap 
The top 10 numbers are:
9999986 9999986 9999990 9999991 9999991 9999992 9999994 9999995 9999997 9999999 

real	0m1.182s
user	0m1.130s
sys	0m0.043s
</pre>
