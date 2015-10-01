---
layout: post
category : NOIP-minsum
tagline: "简单贪心"
tags : [C++, 算法]
---
{% include JB/setup %}

# NOIP minsum   

## desercption    

> ![uva_1629_3](https://raw.githubusercontent.com/Trickness/trickness.github.io/master/_image/noip_minsum.png)   

## solution

	#include <iostream>
	
	using namespace std;
	
	int map[200001];
	
	const int INF = 0x0FFFFFFF
	
	int main(){
		int n;
		cin >> n; 	// number
		for(int i=0;i<n;i++){
			cin >> map[i];
			map[i+n] = map[i];	//	使数值循环
		}
		int c = 0;		//	用于累加
		int max = -INF;	// 	记录结果
		for(int i=0;i<2*n;i++){
			c += map[i];
			if(c > max){
				max = c;
			}
			if(c < 0){
				c = 0;
				continue;
			}
		}
		return 0;
	}