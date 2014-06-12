---
layout: post
title: Least Recently Used(LRU) Cache
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
According to [LeetCode](https://oj.leetcode.com/problems/lru-cache/):

>Design and implement a data structure for Least Recently Used (LRU) cache. It should support the following operations: get and set.

>get(key) - Get the value (will always be positive) of the key if the key exists in the cache, otherwise return -1.

>set(key, value) - Set or insert the value if the key is not already present. When the cache reached its capacity, it should invalidate the least recently used item before inserting a new item.

When dealing with (key, vlaue) pairs, the most straight-forward data structure is hashmap(map or unordered_map in C++). However, at least for C++, it is difficult to control the insertion of new items: (1) the position you can specify is just a hint and does not force the new element to be inserted at that position within the map/unordered_map container, and (2) there is no push_back or push_front methods provided for map/unordered_map. 

So if you implement the LRU cache only using hashmap, you may find that the order of inserted/deleted items is a mess. If you at first insert 10 items(keys 0~9) into the LRU cache of size 10, then enter another 10 items(keys 10~19), you will find that old items are not replaced in order.

```
Initial LRU cache:
Key  Value
9: 54
8: 71
7: 49
6: 12
5: 42
4: 10
3: 37
2: 12
1: 35
0: 83

Erase pair <9,54>
Erase pair <10,2>
Erase pair <8,71>
Erase pair <7,49>
Erase pair <6,12>
Erase pair <5,42>
Erase pair <15,87>
Erase pair <16,35>
Erase pair <17,72>
Erase pair <18,36>

Updated LRU cache:
Key  Value
19: 61
4: 10
14: 20
3: 37
13: 72
2: 12
12: 4
1: 35
11: 53
0: 83

```

To fix the order issue, there are three candidates: `vector`, `list`, `queue`. Since queue doesn't support deleting an item at arbitrary position and deleting an item requires O(n) time cost for vector, the best choice is list. List is flexible enough to support inserting items into any position and erazing an arbitrary item in constant time. 

However, the potential issue for list is that searching for an item requires O(n) time. To find an item in list in constant time, the best way is constructing a hashmap to record the key and its corresponding address in list. In C++, the builtin(actually, from C++ STL) hashmap data structure is `unordered_map`.

In order to update the values in the cache according to keys, `list` also needs to record both keys and values. There are two data structures can be used: `pair<int, int>` or `struct{int key;int val;}`. We were trained to use data strcutures/algorithms provided by standard library, so people tend to use `pair` at first. Unfortunately, when dealing with large data, the STL `pair` struct will be much slower than the self-defined `struct`, because [std::pair<int, int>::pair() constructor initializes the fields with default values and initializing requires writing to each field which requires a whole lot of memory accesses that are relatively time consuming](http://stackoverflow.com/questions/1606894/stdpairint-int-vs-struct-with-two-ints).  

I implemented LRU cache using both `pair<int, int>` and `struct{int key;int val;}`, and I also tested the time cost on my Mac. When setting the LRU capacity to 100, with the same test code, the `pair<int, int>` implementation is faster.

```
macmini:LRU boyang$ time ./lru_pair
real	0m0.010s
user	0m0.007s
sys	0m0.003s

macmini:LRU boyang$ time ./lru_struct
real	0m0.020s
user	0m0.016s
sys	0m0.003s
```

However, when setting the LRU capacity to 10000, the `pair<int, int>` is much slower than `struct{int key;int val;}` implementation.

```
macmini:LRU boyang$ time ./lru_pair
real	0m0.334s
user	0m0.330s
sys	0m0.003s
macmini:LRU boyang$ time ./lru_struct
real	0m1.083s
user	0m1.079s
sys	0m0.004s
```

Following are my implementations of LRU cache. There is one tricky thing of implementing `set(key,value)`: if you check if the cache size surpasses the capacity for every input and move the `while` clause out of the `if` block, then your code won't pass the LeetCode tests, and you will receive an error of exceeding time limit. I think the cache size checking should be trivial, but I still cannot really understand why it matters so much in large data tests.

// struct implementation
```
class LRUCache{
public:
	struct Node {
		int key;
		int val;
		Node(int k, int v):key(k),val(v) {}
	};

	LRUCache(int capacity) {
        cap=capacity;
    }

    int get(int key) {
		unordered_map<int,list<Node>::iterator>::iterator got=hash.find(key);
		if(got!=hash.end()) {
			// update key&value
			Node ptr=*(got->second);
			cache.erase(got->second);
			cache.push_front(ptr);
			hash[key]=cache.begin();

			return ptr.val;
		} else {
			return -1;
		}
    }
    
    void set(int key, int value) {
		Node ptr(key, value);
		unordered_map<int,list<Node>::iterator>::iterator got=hash.find(key);
		if(got!=hash.end()) {
			cache.erase(got->second); // erase so as to update key&value
			hash.erase(key);
		} else {
			// Assume that least recently used items are stored at the end of the cache
	        while(cache.size()>=cap) {
				Node it=cache.back();
				//cout<<"Erase pair <"<<key<<","<<it.val<<">"<<endl; // TEST ONLY
				hash.erase(it.key);
				cache.pop_back();
			}
		}

		cache.push_front(ptr);
		hash[key]=cache.begin();

    }

	void print() {
		cout<<"Key  Value"<<endl;
		for(auto& x: cache)
			cout<<x.key<<": "<<x.val<<endl;
	}

private:
	list<Node> cache; // <value>
	unordered_map<int,list<Node>::iterator> hash; // <key, iterator>
	int cap;
};
```

// pair implementation
```
class LRUCache{
public:
    LRUCache(int capacity) {
        cap=capacity;
    }
    
    int get(int key) {
		unordered_map<int,list<pair<int,int>>::iterator>::iterator got=hash.find(key);
		if(got!=hash.end()) {
			// update key&value
			int val=got->second->second;
			cache.erase(got->second);
			cache.push_front(pair<int,int>(key,val));
			hash[key]=cache.begin();

			return val;
		} else {
			return -1;
		}
    }
    
    void set(int key, int value) {
		// Assume that least recently used items are stored at the end of the cache
		unordered_map<int,list<pair<int,int>>::iterator>::iterator got=hash.find(key);
		if(got!=hash.end()) {
			cache.erase(got->second); // erase so as to update key&value
			hash.erase(key);
		} else {
			while(cache.size()>=cap) { // for big data, must run here
				pair<int,int> it=cache.back();
				//cout<<"Erase pair <"<<it.first<<","<<it.second<<">"<<endl; // TEST ONLY
				hash.erase(it.first);
				cache.pop_back();
			}
		}

		cache.push_front(pair<int,int>(key,value));
		hash[key]=cache.begin();

    }

	void print() {
		cout<<"Key  Value"<<endl;
		for(auto& x: cache)
			cout<<x.first<<": "<<x.second<<endl;
	}

private:
	list<pair<int,int> > cache; // <key, value>
	unordered_map<int,list<pair<int,int>>::iterator> hash; // <key, iterator>
	int cap;
};
```