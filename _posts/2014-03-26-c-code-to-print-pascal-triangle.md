---
layout: post
title: C++ Code to Print Pascal Triangle
categories: 
- Alogrithm
- C/C++
tags:
- Algorithm
- C/C++
status: publish
type: post
published: true
author: Bo Yang
---

Printing Pascal Triangle seems like an easy problem, however, it is not that easy to print a good-looking Pascal Triangle.

To print the Pascal Triangle, for each line, first print spaces to the left of numbers, and then print digit numbers.

To calculate Pascal numbers, two assays can be used: one array to store numbers of above line, the other array to store numbers of this line. The first line has 1 number(1), the second lien has 2 numbers(1 1), the third line has three numbers(1 2 1) and so on.

To make the Pascal Triangle more readable, print spaces between two neighbor &nbsp;numbers in the same line.

~~~cpp

// Pascal triangle
#include <iostream>
#include <iomanip>
#include <cstring>
using namespace std;

const int WIDTH=7;
int main()
{
	int* oldarr=new int[WIDTH]; // store numbers of above row
	int* newarr=new int[WIDTH]; // store numbers of current row

	for(int i=0;i<WIDTH;++i) // cntrol row counting
	{
		for(int j=0;j<WIDTH-i-1;++j) // left space
			cout << setw(2) << " ";

		// set default boundary numbers
		newarr[0]=1;
		if(i<=1)
			newarr[i]=1;

		if(i<=2)
		{
			for(int k=1;k<i;++k)
				newarr[k]=oldarr[k-1]+oldarr[k];
		}
		memcpy(oldarr,newarr,sizeof(int)*WIDTH);

		// Print the middle part
		int idx=0;
		int flag=(WIDTH-i-1)%2;
		for(int j=WIDTH-i-1;j<WIDTH+i;++j)
		{
			if(j%2==flag)
			{
				cout << setw(2) << newarr[idx++];
			} else {
				cout << setw(2) << " ";
			}
		}
		
		//for(int j=WIDTH+i;j&lt;WIDTH;++j) // Print right space, if you like
		//	cout&lt;&lt;setw(4)&lt;&lt;&quot; &quot;;

		cout << endl;
	}

	delete [] oldarr;
	delete [] newarr;

	return 0;
}
~~~

The output looks like:

~~~
             1
           1   1
         1   2   1
       1   3   3   1
     1   4   6   4   1
   1   5  10  10   5   1
 1   6  15  20  15   6   1

~~~
