---
layout: post
title: Coin Change Problem
categories: 
- Algorithm
- Java
- Dynamic Programming
tags:
- Algorithm
- Java
- Dynamic Programming
status: publish
type: post
published: true
author: Bo Yang
---

Given \(N\) denominations,how can a given amount of money \(V\) be made with the least number of coins? The most intuitive solution is greedy algorithm: sort the denominations, try the largest denominations lower than the value every time, and decrease the value with the denominations picked in every iteration. Repeat this process until the value reduces to 0. This greedy strategy works for the US (and most other) coin systems, However, it is not a general solution to all denomincations. For example, if the coin denominations were 1, 3 and 4, then to make 6, the greedy algorithm would choose three coins (4,1,1) whereas the optimal solution is two coins (3,3).

Actually the [change-making problem](http://en.wikipedia.org/wiki/Change-making_problem) is a knapsack type problem, which can be solved efficiently by Dynamic Programming. 

Given denominations \(\left{ x_1, x_2,\cdots,x_n \right}\), in order to get value \(V\), there are two possibilities for a denomination \(x_i\): (1) if \(x_i\) is not picked, then value \(V\) could be made out of \(\left{ x_1, x_2,\cdots,x_{i-1} \right}\). (2) if \(x_i\) is picked, then value \(V-x_i\) should be made out of \(\left{ x_1, x_2,\cdots,x_i \right}\). Since our goal is to find the least number of coins, we can formulate the following recursive function:
\[
	OPT(i,v)=\left{ \begin{aligned}
	0 & \text{if} i=0 \\
	OPT(i-1,v) & \text{if} x_i>v \\
	\min \left{ OPT(i-1,v), 1+OPT(i,v-v_i) \right} & \text{otherwise}
	\end{aligned} 
\]

Following is my implementation in Java. During the initialization, the first column(value=0) are set to 0, and the first row(denomination=1) are set to the number of corresponding value. Another observation is that, if a value(the grid) in the following table can be divided by the corresponding denomination(the denomination \(x_i\) of the row), then the smallest coin changes should be \(v/x_i\). 

Denom|Values
-----|-----
1|0 1 2 3 4 5 6 7 8 9 10 11 
3|0 1 2 1 2 3 2 3 4 3 4 5 
4|0 1 2 1 1 2 2 2 2 3 3 3 



	public class Coin {
		public static int CoinChange(int value, int[] denom) {
			int M[][]=new int[denom.length][value+1];
	
			// Initialization
			final int INF=9999999;
			for(int i=0;i<denom.length;++i) {
				for(int j=0;j<=value;++j) {
					if(j%denom[i]==0)
						M[i][j]=j/denom[i];
					else
						M[i][j]=INF;	
				}
			}
	
			// Fill the table with smallest coin changes
			for(int i=1;i<denom.length;++i) {
				for(int j=1;j<=value;++j) {
					if(denom[i]>j)
						M[i][j]=M[i-1][j];
					else
						M[i][j]=M[i-1][j]<1+M[i][j-denom[i]] ? M[i-1][j] : 1+M[i][j-denom[i]];
				}
			}
	
			// Print the table
			for(int i=0;i<denom.length;++i) {
				for(int j=0;j<=value;++j) {
					System.out.print(M[i][j]+" ");
				}
				System.out.print("\n");
			}
	
			return M[denom.length-1][value];
		}
	
		public static void main(String[] args) {
			int[] denom={1,3,4};
			int value=10;
			int num=CoinChange(value,denom);
			System.out.println(num);
		}
	}
	
