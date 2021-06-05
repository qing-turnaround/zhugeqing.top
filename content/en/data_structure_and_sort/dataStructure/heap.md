---
date: 2021-03-16
description: "战略上藐视技术，战术上重视技术——闪客"
image: "images/recommend_site/xingyouji.jpg"
title: "堆"
author: 诸葛青
authorEmoji: 🤖
pinned: true
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

``最大堆（大顶堆）``：堆中每一个节点的值 ``都大于等于`` 其孩子节点的值。``最大堆``的特性是 ``堆顶元素（根节点）是堆中的最大值``。

``最小堆（小顶堆）``：堆中每一个节点的值 ``都小于等于`` 其孩子节点的值。``最小堆``的特性是 ``堆顶元素（根节点）是堆中的最小值``。

![](/images/dataStructure/heap/heap1.png)

## 堆的存储
``堆``的逻辑结构是一棵二叉树，所以可以考虑使用二叉树的表示方法来表示堆。但是因为``堆中``元素按照一定的优先顺序排列，因此可以使用 ``数组``来表示，这样可以节省子节点指针空间，并且可以``快速访问``每个节点。``堆``的数组表示其实就是堆层级遍历的结果，如下图所示：

![](/images/dataStructure/heap/heap2.png)

## 堆的操作
在``堆``的数据结构中，我们常用堆的`初始化`、``插入``、``删除``、``获取堆顶元素``的操作。
接下来我们一一进行学习：

## 堆初始化操作
* 1.``确定堆包含的属性``：``堆的最大空间``，``堆的当前大小``，``堆的存放数组``
* 2.``初始化堆，为其分配内存空间``

代码实现如下（大顶堆）：
{{< codes golang c>}}

  {{< code >}}

  ```golang
type MaxHeap struct {
	// 使用数组创建完全二叉树的结构，然后使用二叉树构建一个「堆」
	maxHeap []int
	// heapSize用于数组的大小，因为数组在创建的时候至少需要指明数组的元素个数
	heapSize int
	// realSize用于记录「堆」的元素个数
	realSize int
}

//大顶堆初始化函数
func MaxHeapInit(size int)MaxHeap{
	return MaxHeap{
		maxHeap: make([]int, size+1),
		heapSize: size,
		realSize: 0,
	}
}
  ```
  {{< /code >}}

  {{< code >}}

  ```c
typedef struct
{
    //堆的数组
    int *maxHeap;
    //堆的元素个数
    int realSize;
    //堆的大小
    int MaxSize;
} MaxHeap;

//初始化堆函数
void initMaxHeap(MaxHeap *mh, int size)
{
    mh->MaxSize = size;
    mh->maxHeap = (int *)malloc((mh->MaxSize + 1) * sizeof(int)); //从1开始存储
    mh->realSize = 0;
}
  ```
  {{< /code >}}
{{< /codes >}}

## 堆插入操作

``堆``可以看成一个完全二叉树，每次总是先填满``上一层``，再从``下一层从左往右``依次插入。堆插入操作的步骤（大顶堆）：

* 1.``将新元素增加到堆的末尾``
* 2.``按照优先顺序，将新元素与其父节点比较，如果新元素小于父节点则将两者交换位置``
* 3.``不断进行第2步操作，直到不需要交换新元素和父节点，或者达到堆顶``
* 4.``最后通过得到一个大顶堆``

代码实现如下（大顶堆）：

{{< codes golang c>}}

  {{< code >}}

  ```golang
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
  ```
  {{< /code >}}

  {{< code >}}

  ```c
//添加元素函数
void insert(MaxHeap *mh, int element)
{
    //如果当前元素个数已经满了，那就输出错误信息
    if (mh->realSize + 1 > mh->MaxSize)
    {
        printf("Add too many elements!");
        return;
    }

    mh->realSize++;
    //插入元素到堆的末尾
    mh->maxHeap[mh->realSize] = element;
    //调整堆
    int index = mh->realSize;
    //父节点的下标
    int parent = index / 2;
    while (mh->maxHeap[index] > mh->maxHeap[parent] && index > 1) //堆下标从1开始，1没有父节点
    {
        //交换元素
        int temp = mh->maxHeap[index];
        mh->maxHeap[index] = mh->maxHeap[parent];
        mh->maxHeap[parent] = temp;
        //为了不断向上调整
        index = parent;
        parent = index / 2;
    }
}
  ```
  {{< /code >}}
{{< /codes >}}

## 堆删除操作
``堆``的``删除操作``与``插入操作``相反，``插入操作``是``从下往上``调整``堆``，而``删除操作``却是``从上往下``调整``堆``，堆删除操作的步骤（大顶堆）：

* 1.``删除堆顶元素（通常是将堆顶元素放置在数组的末尾）``
* 2.``比较左右子节点，将大的元素上调``
* 3.``不断进行步骤2，直到不需要调整或者调整到堆底``

代码实现如下：（大顶堆）
{{< codes golang c>}}

  {{< code >}}

  ```golang
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
  ```
  {{< /code >}}

  {{< code >}}

  ```c
//堆元素删除函数
int delete (MaxHeap *mh)
{
    // 如果当前「堆」的元素个数为0， 则返回「Don't have any element」
    if (mh->realSize < 1)
    {
        printf("Don't have any element!");
        return intMIN;
    }

    //保留删除的元素，以便返回
    int removeElement = mh->maxHeap[1];
    //删除堆顶元素，把最后一个元素移至堆顶
    mh->maxHeap[1] = mh->maxHeap[mh->realSize];
    //把最后一个元素置为最小(已经宏定义IntMin为4字节int类型最小值)
    mh->maxHeap[mh->realSize] = intMIN;
    //元素长度大小减1
    mh->realSize--;
    //向下调整
    int index = 1;

    while (index < mh->realSize && index <= mh->realSize / 2)
    {
        //左节点
        int left = index * 2;
        //右节点
        int right = index * 2 + 1;

        if (mh->maxHeap[index] < mh->maxHeap[left] || mh->maxHeap[index] < mh->maxHeap[right])
        {
            if (mh->maxHeap[left] > mh->maxHeap[right])
            {
                int temp = mh->maxHeap[index];
                mh->maxHeap[index] = mh->maxHeap[left];
                mh->maxHeap[left] = temp;
                index = left;
            }
            else
            {
                int temp = mh->maxHeap[index];
                mh->maxHeap[index] = mh->maxHeap[right];
                mh->maxHeap[right] = temp;
                index = right;
            }
        }
        else
        {
            break;
        }
    }
    return removeElement;
}
  ```
  {{< /code >}}
{{< /codes >}}

## 获取堆顶元素操作

直接返回堆顶元素就行

代码如下：（大顶堆）
{{< codes golang c>}}

  {{< code >}}

  ```golang
//获取堆顶元素函数
func (Maxheap MaxHeap)peek()int{
	return Maxheap.maxHeap[1]
}
  ```
  {{< /code >}}

  {{< code >}}

  ```c
//获取堆顶元素函数
int peek(MaxHeap mh)
{
    return mh.maxHeap[1];
}

  ```
  {{< /code >}}
{{< /codes >}}

## 总结
对于``堆``这种数据结构，我们只需要掌握如何来``初始化``，``删除元素``，``添加元素``，堆的应用十分广泛，如下：

* 1.``堆排序``
* 2.``快速找出一个集合中的最小值（或者最大值）``
* 3.``构建优先队列``
* 4.``在朋友面前装逼（开个玩笑）``

总体代码如下（包含大顶堆和小顶堆）：

{{< codes go go c c>}}

  {{< code >}}

  ```golang
package main

import (
	"fmt"
	"math"
)

//构建大顶堆
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

//大顶堆初始化函数
func MaxHeapInit(size int)MaxHeap{

	return MaxHeap{
		maxHeap: make([]int, size+1),
		heapSize: size,
		realSize: 0,
	}

}


func main(){
	//指明需要构建堆的大小
	size := 5
	var MaxHeap MaxHeap = MaxHeapInit(size)
	//此堆的下标从1开始，有些是从0开始
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

//构建小顶堆
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

//小顶堆初始化函数
func MinHeapInit(size int)MinHeap{
	return MinHeap{
		minHeap: make([]int, size+1),
		heapSize: size,
		realSize: 0,
	}
}



func main(){

	//指明需要构建堆的大小
	size := 5
	var MinHeap MinHeap = MinHeapInit(size)
	//此堆的下标从1开始，有些是从0开始
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

    {{< code >}}

  ```c
#include <stdio.h>
#include <stdlib.h>
// 宏定义最小值
#define intMIN 0x80000000;

typedef struct
{
    //堆的数组
    int *maxHeap;
    //堆的元素个数
    int realSize;
    //堆的大小
    int MaxSize;
} MaxHeap;

void insert(MaxHeap *mh, int element)
{
    //如果当前元素个数已经满了，那就输出错误信息
    if (mh->realSize + 1 > mh->MaxSize)
    {
        printf("Add too many elements!");
        return;
    }

    mh->realSize++;
    //插入元素到堆的末尾
    mh->maxHeap[mh->realSize] = element;
    //调整堆
    int index = mh->realSize;
    //父节点的下标
    int parent = index / 2;
    while (mh->maxHeap[index] > mh->maxHeap[parent] && index > 1) //堆下标从1开始，1没有父节点
    {
        //交换元素
        int temp = mh->maxHeap[index];
        mh->maxHeap[index] = mh->maxHeap[parent];
        mh->maxHeap[parent] = temp;
        //为了不断向上调整
        index = parent;
        parent = index / 2;
    }
}

//堆元素删除函数
int delete (MaxHeap *mh)
{
    // 如果当前「堆」的元素个数为0， 则返回「Don't have any element」
    if (mh->realSize < 1)
    {
        printf("Don't have any element!");
        return intMIN;
    }

    //保留删除的元素，以便返回
    int removeElement = mh->maxHeap[1];
    //删除堆顶元素，把最后一个元素移至堆顶
    mh->maxHeap[1] = mh->maxHeap[mh->realSize];
    //把最后一个元素置为最小(已经宏定义IntMin为4字节int类型最小值)
    mh->maxHeap[mh->realSize] = intMIN;
    //元素长度大小减1
    mh->realSize--;
    //向下调整
    int index = 1;

    while (index < mh->realSize && index <= mh->realSize / 2)
    {
        //左节点
        int left = index * 2;
        //右节点
        int right = index * 2 + 1;

        if (mh->maxHeap[index] < mh->maxHeap[left] || mh->maxHeap[index] < mh->maxHeap[right])
        {
            if (mh->maxHeap[left] > mh->maxHeap[right])
            {
                int temp = mh->maxHeap[index];
                mh->maxHeap[index] = mh->maxHeap[left];
                mh->maxHeap[left] = temp;
                index = left;
            }
            else
            {
                int temp = mh->maxHeap[index];
                mh->maxHeap[index] = mh->maxHeap[right];
                mh->maxHeap[right] = temp;
                index = right;
            }
        }
        else
        {
            break;
        }
    }
    return removeElement;
}

//初始化堆函数
void initMaxHeap(MaxHeap *mh, int size)
{
    mh->MaxSize = size;
    mh->maxHeap = (int *)malloc((mh->MaxSize + 1) * sizeof(int)); //从1开始存储
    mh->realSize = 0;
}

//获取堆的大小函数
int size(MaxHeap *mh)
{
    return mh->MaxSize;
}

//遍历堆函数
void traversal(MaxHeap mh)
{
    for (int i = 1; i <= mh.realSize; i++)
    {
        printf("%d\n", mh.maxHeap[i]);
    }
}

//获取堆顶元素函数
int peek(MaxHeap mh)
{
    return mh.maxHeap[1];
}

int main()
{
    MaxHeap mh;
    initMaxHeap(&mh, 5);
    insert(&mh, 3);
    insert(&mh, 1);
    insert(&mh, 5);
    insert(&mh, 8);
    traversal(mh);
    printf("\n删除堆顶的元素为%d\n\n", delete (&mh));
    traversal(mh);
    system("pause");
}

  ```
  {{< /code >}}

    {{< code >}}

  ```c
#include <stdio.h>
#include <stdlib.h>
// 宏定义最大值
#define intMax 0x1fffffff;

//构建小顶堆
typedef struct
{
    //堆的数组
    int *minHeap;
    //堆的元素个数
    int realSize;
    //堆的大小
    int MinSize;
} MinHeap;

void insert(MinHeap *mh, int element)
{
    //如果当前元素个数已经满了，那就输出错误信息
    if (mh->realSize + 1 > mh->MinSize)
    {
        printf("Add too many elements!");
        return;
    }

    mh->realSize++;
    //插入元素到堆的末尾
    mh->minHeap[mh->realSize] = element;
    //调整堆
    int index = mh->realSize;
    //父节点的下标
    int parent = index / 2;
    while (mh->minHeap[index] < mh->minHeap[parent] && index > 1) //堆下标从1开始，1没有父节点
    {
        //交换元素
        int temp = mh->minHeap[index];
        mh->minHeap[index] = mh->minHeap[parent];
        mh->minHeap[parent] = temp;
        //为了不断向上调整
        index = parent;
        parent = index / 2;
    }
}

//堆元素删除函数
int delete (MinHeap *mh)
{
    // 如果当前「堆」的元素个数为0， 则返回「Don't have any element」
    if (mh->realSize < 1)
    {
        printf("Don't have any element!");
        return intMax;
    }

    //保留删除的元素，以便返回
    int removeElement = mh->minHeap[1];
    //删除堆顶元素，把最后一个元素移至堆顶
    mh->minHeap[1] = mh->minHeap[mh->realSize];
    //把最后一个元素置为最小(已经宏定义IntMin为4字节int类型最小值)
    mh->minHeap[mh->realSize] = intMax;
    //元素长度大小减1
    mh->realSize--;
    //向下调整
    int index = 1;

    while (index < mh->realSize && index <= mh->realSize / 2)
    {
        //左节点
        int left = index * 2;
        //右节点
        int right = index * 2 + 1;

        if (mh->minHeap[index] > mh->minHeap[left] || mh->minHeap[index] > mh->minHeap[right])
        {
            if (mh->minHeap[left] < mh->minHeap[right])
            {
                int temp = mh->minHeap[index];
                mh->minHeap[index] = mh->minHeap[left];
                mh->minHeap[left] = temp;
                index = left;
            }
            else
            {
                int temp = mh->minHeap[index];
                mh->minHeap[index] = mh->minHeap[right];
                mh->minHeap[right] = temp;
                index = right;
            }
        }
        else
        {
            break;
        }
    }
    return removeElement;
}

//初始化堆函数
void initMinHeap(MinHeap *mh, int size)
{
    mh->MinSize = size;
    mh->minHeap = (int *)malloc((mh->MinSize + 1) * sizeof(int)); //从1开始存储
    mh->realSize = 0;
}

//获取堆的大小函数
int size(MinHeap *mh)
{
    return mh->MinSize;
}

//遍历堆函数
void traversal(MinHeap mh)
{
    for (int i = 1; i <= mh.realSize; i++)
    {
        printf("%d\n", mh.minHeap[i]);
    }
}

//获取堆顶元素函数
int peek(MinHeap mh)
{
    return mh.minHeap[1];
}

int main()
{
    MinHeap mh;
    initMinHeap(&mh, 5);
    insert(&mh, 3);
    insert(&mh, 1);
    insert(&mh, 5);
    insert(&mh, 8);
    traversal(mh);
    printf("\n删除堆顶的元素为%d\n\n", delete (&mh));
    traversal(mh);
    system("pause");
}

  ```
  {{< /code >}}
{{< /codes >}}

