---
date: 2020-10-16T12:00:13+09:00
description: "战略上藐视技术，战术上重视技术——闪客"
image: "images/recommend_site/xingyouji.jpg"
title: "堆"
author: 诸葛青
authorEmoji: 🤖
pinned: false
tags:
- 数据结构
series:
-   
---
## 堆的定义

根据 [维基百科](https://zh.wikipedia.org/wiki/%E5%A0%86%E7%A9%8D) 的定义，``堆`` 是一种特别的二叉树，满足以下条件的二叉树，可以称之为 ``堆``：
* 1.完全二叉树；
* 2.每一个节点的值都必须 大于等于或者小于等于 其孩子节点的值。

``堆`` 具有以下的特点：

* 1.可以在 ``O(logN)`` 的时间复杂度内向 ``堆`` 中插入元素；
* 2.可以在 ``O(logN)`` 的时间复杂度内向 ``堆`` 中删除元素；
* 3.可以在 ``O(1)`` 的时间复杂度内获取 ``堆`` 中的最大值或最小值。

## 堆的分类
``堆`` 分为两种类型：``最大堆`` 和 ``最小堆``。

``最大堆``：堆中每一个节点的值 ``都大于等于`` 其孩子节点的值。``最大堆``的特性是 ``堆顶元素（根节点）是堆中的最大值``。

``最小堆``：堆中每一个节点的值 ``都小于等于`` 其孩子节点的值。``最小堆``的特性是 ``堆顶元素（根节点）是堆中的最小值``。

![](/images/dataStructure/heap/heap1.png)

## 堆的存储
``堆``的逻辑结构是一棵二叉树，所以可以考虑使用二叉树的表示方法来表示堆。但是因为``堆中``元素按照一定的优先顺序排列，因此可以使用 ``数组``来表示，这样可以节省子节点指针空间，并且可以``快速访问``每个节点。``堆``的数组表示其实就是堆层级遍历的结果，如下图所示：

![](/images/dataStructure/heap/heap2.png)

## 堆的操作
在``堆``的数据结构中，我们常用堆的``插入``、``删除``、``获取堆顶元素``的操作。
接下来我们一一进行学习：

## 堆初始化操作





{{< codes go go>}}

  {{< code >}}

  ```golang
package main

import (
	"fmt"
	"math"
)

type MaxHeap struct {
	// 使用数组创建完全二叉树的结构，然后使用二叉树构建一个「堆」
	maxHeap []int
	// heapSize用于数组的大小，因为数组在创建的时候至少需要指明数组的元素个数
	heapSize int
	// realSize用于记录「堆」的元素个数
	realSize int
}

//添加元素函数
func (Maxheap *MaxHeap)insert(element int){
	//如果当前元素个数已经和堆想等，那就输出错误信息
	if Maxheap.realSize+1 > Maxheap.heapSize{
		fmt.Println("Add too many elements!")
		return
	}
	Maxheap.realSize++
	//新增加元素的下标
	index := Maxheap.realSize
	//父节点下标
	parent := index / 2
	Maxheap.maxHeap[index] = element

	//如果大于父节点就不断向上交换
	for Maxheap.maxHeap[index] > Maxheap.maxHeap[parent] && index > 1{ //因为下标1没有父节点，所有index不能等于或者小于1
		//进行交换
		Maxheap.maxHeap[index],Maxheap.maxHeap[parent] = Maxheap.maxHeap[parent],Maxheap.maxHeap[index]
		index = parent
		parent = index / 2
		//继续比较
	}
}

//获取堆顶元素函数
func (Maxheap MaxHeap)peek()int{
	return Maxheap.maxHeap[1]
}

//删除堆顶元素函数
func (Maxheap *MaxHeap)delete()int{
	// 如果当前「堆」的元素个数为0， 则返回「Don't have any element」
	removeElement := Maxheap.maxHeap[1]
	if Maxheap.realSize < 1{
		fmt.Println("Don't have any element!")
		return math.MaxInt64
	}else {
		//当前堆中有元素，Maxheap.realSize >= 1
		// 将「堆」中的最后一个元素赋值给堆顶元素
		Maxheap.maxHeap[1] = Maxheap.maxHeap[Maxheap.realSize]
		//去除最后一个元素
		Maxheap.maxHeap[Maxheap.realSize] = math.MinInt64
		Maxheap.realSize--
		index := 1
		// 不断向下调整堆
		// 当删除的元素不是孩子节点时
		for index < Maxheap.realSize && index <= Maxheap.realSize/2 {
			// 被删除节点的左孩子节点
			left := index * 2;
			// 被删除节点的右孩子节点
			right := (index * 2) + 1;
			// 当删除节点的元素小于 左孩子节点或者右孩子节点，代表该元素的值小，此时需要将该元素与左、右孩子节点中最大的值进行交换
			if Maxheap.maxHeap[index] < Maxheap.maxHeap[left] ||  Maxheap.maxHeap[index] < Maxheap.maxHeap[right] {
				if Maxheap.maxHeap[left] > Maxheap.maxHeap[right] {
					temp := Maxheap.maxHeap[left]
					Maxheap.maxHeap[left] = Maxheap.maxHeap[index]
					Maxheap.maxHeap[index] = temp
					index = left
				} else {
					// maxHeap[left] <= maxHeap[right]
					temp := Maxheap.maxHeap[right]
					Maxheap.maxHeap[right] = Maxheap.maxHeap[index]
					Maxheap.maxHeap[index] = temp
					index = right
				}
			} else {
				break
			}
		}

	}

	return removeElement
}

// 返回「堆」的元素个数
func (Maxheap MaxHeap)size()int{
	return Maxheap.realSize
}


func main(){
	var MaxHeap MaxHeap
	//指明需要构建堆的长度
	MaxHeap.heapSize = 5
	//初始化堆数组
	//此堆的下标从1开始，有些是从0开始
	MaxHeap.maxHeap = make([]int,MaxHeap.heapSize+1)
	for i:=0;i<MaxHeap.heapSize+1;i++{ //大顶堆，先把所有元素初始化为最小值
		MaxHeap.maxHeap[i] = math.MinInt64
	}
	//当前堆中元素个数为0
	MaxHeap.realSize = 0
	//添加1
	MaxHeap.insert(1)
	fmt.Println(MaxHeap.maxHeap)
	fmt.Println(MaxHeap.realSize)
	//添加2
	MaxHeap.insert(2)
	//添加3
	MaxHeap.insert(3)
	fmt.Println(MaxHeap.maxHeap)

	//添加5
	MaxHeap.insert(5)
	//删除堆顶
	fmt.Println(MaxHeap.delete())
	fmt.Println(MaxHeap.maxHeap)
	//取堆顶
	fmt.Println(MaxHeap.peek())

	//取堆元素个数
	fmt.Println(MaxHeap.size())
}
  ```
  {{< /code >}}

  {{< code >}}

  ```golang
package main

import (
	"fmt"
	"math"
)

type MinHeap struct {
	// 使用数组创建完全二叉树的结构，然后使用二叉树构建一个「堆」
	minHeap []int
	// heapSize用于数组的大小，因为数组在创建的时候至少需要指明数组的元素个数
	heapSize int
	// realSize用于记录「堆」的元素个数
	realSize int
}

//添加元素函数
func (Minheap *MinHeap)insert(element int){
	//如果当前元素个数已经和堆想等，那就输出错误信息
	if Minheap.realSize+1 > Minheap.heapSize{
		fmt.Println("Add too many elements!")
		return
	}
	Minheap.realSize++
	//新增加元素的下标
	index := Minheap.realSize
	//父节点下标
	parent := index / 2
	Minheap.minHeap[index] = element

	//如果大于父节点就不断向上交换
	for Minheap.minHeap[index] < Minheap.minHeap[parent] && index > 1{ //因为下标1没有父节点，所有index不能等于或者小于1
		//进行交换
		Minheap.minHeap[index],Minheap.minHeap[parent] = Minheap.minHeap[parent],Minheap.minHeap[index]
		index = parent
		parent = index / 2
		//继续比较
	}
}

//获取堆顶元素函数
func (Minheap MinHeap)peek()int{
	return Minheap.minHeap[1]
}

//删除堆顶元素函数
func (Minheap *MinHeap)delete()int{
	// 如果当前「堆」的元素个数为0， 则返回「Don't have any element」
	removeElement := Minheap.minHeap[1]
	if Minheap.realSize < 1{
		fmt.Println("Don't have any element!")
		return math.MaxInt64
	}else {
		//当前堆中有元素，Minheap.realSize >= 1
		// 将「堆」中的最后一个元素赋值给堆顶元素
		Minheap.minHeap[1] = Minheap.minHeap[Minheap.realSize]
		//去除最后一个元素
		Minheap.minHeap[Minheap.realSize] = math.MinInt64
		Minheap.realSize--
		index := 1
		// 不断向下调整堆
		// 当删除的元素不是孩子节点时
		for index < Minheap.realSize && index <= Minheap.realSize/2 {
			// 被删除节点的左孩子节点
			left := index * 2;
			// 被删除节点的右孩子节点
			right := (index * 2) + 1;
			// 当删除节点的元素小于 左孩子节点或者右孩子节点，代表该元素的值小，此时需要将该元素与左、右孩子节点中最大的值进行交换
			if Minheap.minHeap[index] > Minheap.minHeap[left] ||  Minheap.minHeap[index] > Minheap.minHeap[right] {
				if Minheap.minHeap[left] < Minheap.minHeap[right] {
					temp := Minheap.minHeap[left]
					Minheap.minHeap[left] = Minheap.minHeap[index]
					Minheap.minHeap[index] = temp
					index = left
				} else {
					// MinHeap[left] >= MinHeap[right]
					temp := Minheap.minHeap[right]
					Minheap.minHeap[right] = Minheap.minHeap[index]
					Minheap.minHeap[index] = temp
					index = right
				}
			} else {
				break
			}
		}

	}

	return removeElement
}

// 返回「堆」的元素个数
func (Minheap MinHeap)size()int{
	return Minheap.realSize
}


func main(){
	var MinHeap MinHeap
	//指明需要构建堆的长度
	MinHeap.heapSize = 5
	//初始化堆数组
	//此堆的下标从1开始，有些是从0开始
	MinHeap.minHeap = make([]int,MinHeap.heapSize+1)
	for i:=0;i<MinHeap.heapSize+1;i++{ //大顶堆，先把所有元素初始化为最小值
		MinHeap.minHeap[i] = math.MinInt64
	}
	//当前堆中元素个数为0
	MinHeap.realSize = 0
	//添加1
	MinHeap.insert(1)
	fmt.Println(MinHeap.minHeap)
	fmt.Println(MinHeap.realSize)
	//添加2
	MinHeap.insert(2)
	//添加3
	MinHeap.insert(3)
	fmt.Println(MinHeap.minHeap)

	//添加5
	MinHeap.insert(5)
	//删除堆顶
	fmt.Println(MinHeap.delete())
	fmt.Println(MinHeap.minHeap)
	//取堆顶
	fmt.Println(MinHeap.peek())

	//取堆元素个数
	fmt.Println(MinHeap.size())
}
  ```
  {{< /code >}}
{{< /codes >}}

