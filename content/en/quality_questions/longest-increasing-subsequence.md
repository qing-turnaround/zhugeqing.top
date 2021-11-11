---
date: 2021-11-10
description: "生命没有极限，除非你自己定义"
image: "images/recommend_site/xingyouji.jpg"
title: "最长上升子序列"
author: 诸葛青
authorEmoji: 😃
pinned: true
tags:
- 
series:
-
---

## 题目
<font color=SkyBlue>

给定一个长度为 N 的数列，求数值严格单调递增的子序列的长度最长是多少。

输入格式
第一行包含整数 N。

第二行包含 N 个整数，表示完整序列。

输出格式
输出一个整数，表示最大长度。
</font>
``
数据范围
1≤N≤1000，
−109≤数列中的数≤109
``

```ha
输入样例：
7
3 1 2 1 8 5 6
输出样例：
4
```

## 解法 


### 动态规划

<font color=SkyBlue>
思路：定义dp[i]为考虑数组前i个元素，以nums[i]结尾的最长上升子序列，从小到大计算dp[i]的值。
状态转移方程为：dp[i] = max(dp[j]) + 1，（其中 0 <= j < i且nums[i] > nums[j]）
当计算dp[i]的值时（初始dp[i]=1），j需要从0遍历到i-1，其中当nums[j] < nums[i]时，dp[i] = max(dp[i]，dp[j]+1)
然后求出dp数组中的最大值。
</font>

``时间复杂度 O(n^2)：遍历nums数组需要O(n)，计算每一个dp[i]需要O(n) ``
``空间复杂度 O(n)：dp数组需要O(n)的空间``

```Go
package main

import (
	"fmt"
)

func main() {
	n := 0
	fmt.Scanln(&n)
	nums := make([]int, n)
	for i := range nums {
		fmt.Scan(&nums[i])
	}
	fmt.Println(lengthOfLIS(nums))
}

func lengthOfLIS(nums []int) int {
	n := len(nums)
	dp := make([]int, n)
	dp[0] = 1
	res := 1
	for i := 1; i < n; i++ {
		//初始
		dp[i] = 1
		for j := 0; j < i; j++ {
			if nums[i] > nums[j] {
				dp[i] = max(dp[i], dp[j]+1)
			}
		}

		res = max(res, dp[i])
	}

	return res
}

func max(a, b int) int {
	if a > b {
		return a
	}

	return b
}
```

### 二分 + 动态规划

<font color=SkyBlue>单纯的动态规划做法，计算每一个i的dp值需要从0遍历到i-1，这样太耗费时间，所以可以考虑维护一个数组dp，dp中的元素是严格递增的，每当遍历一个数组中元素v时，如果v比dp末尾元素还大，那就直接插入尾部，否则将dp数组中第一个大于v的元素改成v（除非有和v相同大小的元素，那么就不用改变），在遍历的同时需要维护一个res，记录dp数组的长度（不是真实长度，而是维护的虚拟长度），最终返回res</font>

``时间复杂度 O(NlogN) ： 遍历 numsnums 列表需 O(N)O(N)，在每个 nums[i]nums[i] 二分法需 O(logN)。``
``空间复杂度 O(N)：dp数组需要O(n)的空间``

<img src="/images/algorithm_interview/LIS.png" width="70%" height="70%">

```Go
package main

import "fmt"

func main() {
	var n int
	fmt.Scan(&n)
	nums := make([]int, n)

	for i := range nums {
		fmt.Scan(&nums[i])
	}

	fmt.Println(lengthOfLIS(nums))

}

func lengthOfLIS(nums []int) int {
	// 二分查找 + 动态规划
	n := len(nums)
	dp := make([]int, n)
	res := 0
	for _, v := range nums {
		l, r := 0, res
		for l < r {
			mid := l + (r-l)>>1
			if dp[mid] < v {
				l = mid + 1
			} else {
				r = mid
			}
		}

		dp[l] = v
		//如果r依然等于res，说明v要大于dp数组内的所有元素，res++
		if res == r {
			res++
		}
	}

	return res
}
```