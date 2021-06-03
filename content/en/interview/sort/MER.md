---
date: 2020-10-16T12:00:08+09:00
description: "战略上藐视技术，战术上重视技术——闪客"
image: "images/recommend_site/xingyouji.jpg"
title: "归并排序"
author: 诸葛青
authorEmoji: 🤖
pinned: false
tags:
- 排序算法
series:
-   
---

``约翰·冯·诺伊曼``在 1945 年提出了``归并排序``。在讲解归并排序之前，我们先一起思考一个问题：``如何将两个有序的列表合并成一个有序的列表？``

## 将两个有序的列表合并成一个有序的列表

这太简单了，首先想到的思路就是，将两个列表拼接成一个列表，然后之前学的``冒泡``、``选择``、``插入``、``希尔``、``堆``、``快排``都可以派上用场了。

觉得太暴力了一点？那我们换个思路。

既然列表已经有序了，通过前几章的学习，我们已经知道，``插入排序的过程中``，``被插入的数组也是有序的``。这就好办了，我们将``其中一个列表中的元素``逐个``插入另一个列表``中即可。

但是按照这个思路，我们只需要一个列表有序就行了，另一个列表不管是不是有序的，都会被逐个取出来，插入第一个列表中。那么，在两个列表都已经有序的情况下，还可以有更优的合并方案吗？

深入思考之后，我们发现，在第二个列表向第一个列表逐个插入的过程中，由于第二个列表已经有序，所以后续插入的元素一定不会在前面插入的元素之前。在逐个插入的过程中，``每次插入时``，只需要``从上次插入的位置开始``，继续``向后寻找插入位置``即可。这样一来，我们最多只需要将两个有序数组``遍历一次``就可以完成合并。

思路很接近了，如何实现它呢？我们发现，在向数组中不断插入新数字时，``原数组需要不断腾出位置``，这是一个比较复杂的过程，而且这个过程必然导致增加一轮遍历。

但好在我们有一个替代方案：只要``开辟一个长度等同于两个数组长度之和的新数组``，并``使用两个指针``来``遍历``原有的``两个数组``，``不断``将较小的数字``添加到新数组``中，并``移动对应的指针``即可。

根据这个思路，我们可以写出合并两个有序列表的代码：
{{< codes golang java>}}

  {{< code >}}

  ```golang
// 将两个有序数组合并为一个有序数组
func merge(arr1,arr2 []int)[]int{
    //返回的有序的结果数组
    res := make([]int,len(arr1)+len(arr2))
	//遍历两个数组的指针
    index1,index2 := 0,0
    //遍历	
	for index1 < len(arr1) && index2 < len(arr2){
		if arr1[index1]<=arr2[index2]{
			res[index1+index2] = arr1[index1]
			index1++
		}else{
			res[index1+index2] = arr2[index2]
			index2++
		}
	}
    // 将剩余数字补到结果数组之后
	for index1 < len(arr1){
		res[index1+index2] = arr1[index1]
		index1++
	}

	for index2 < len(arr2){
		res[index1+index2] = arr2[index2]
		index2++
	}
	return res
}
  ```
  {{< /code >}}

  {{< code >}}

  ```java
// 将两个有序数组合并为一个有序数组
private static int[] merge(int[] arr1, int[] arr2) {
    int[] result = new int[arr1.length + arr2.length];
    int index1 = 0, index2 = 0;
    while (index1 < arr1.length && index2 < arr2.length) {
        if (arr1[index1] <= arr2[index2]) {
            result[index1 + index2] = arr1[index1];
            index1++;
        } else {
            result[index1 + index2] = arr2[index2];
            index2++;
        }
    }
    // 将剩余数字补到结果数组之后
    while (index1 < arr1.length) {
        result[index1 + index2] = arr1[index1];
        index1++;
    }
    while (index2 < arr2.length) {
        result[index1 + index2] = arr2[index2];
        index2++;
    }
    return result;
}
  ```
    {{< /code >}}
{{< /codes >}}

合并有序数组的问题解决了，但我们排序时用的都是``无序数组``，那么上哪里去``找这两个有序的数组呢？``

答案是 —— ``自己拆分``，我们可以把数组不断地拆成两份，直到``只剩下一个数字``时，``这一个数字组成的数组我们就可以认为它是有序的``。

然后通过上述合并有序列表的思路，``将 1 个数字组成的有序数组````合并成一个包含 2 个数字的有序数组``，``再将 2 个数字组成的有序数组合并成包含 4 个数字的有序数组...直到整个数组排序完成``，``这就是归并排序（Merge Sort）的思想``。

##  将数组拆分成有序数组
拆分过程使用了二分的思想，这是一个递归的过程，归并排序使用的递归框架如下：

{{< codes golang java>}}

  {{< code >}}

  ```golang
//归并排序
func mergeSort(arr []int){
	if len(arr) <= 1{
		return
	}
	//把结果存入res中
	res := mergeRescurison(arr,0,len(arr)-1)
	for i:=0;i<len(arr);i++{
		arr[i] = res[i]
	}

}
//拆分数组
func mergeRescurison(arr []int,start,end int)[]int{
	if start >= end{//递归终止条件
		return []int{arr[start]}
	}
	
	//左区间递归
	left := mergeRescurison(arr,start,(start+end)/2)
	//右区间递归
	right := mergeRescurison(arr,(start+end)/2+1,end)
	//将两个有序数组合并
	return merge(left,right)
}

  ```
  {{< /code >}}

  {{< code >}}

  ```java
public static void mergeSort(int[] arr) {
    if (arr.length == 0) return;
    int[] result = mergeSort(arr, 0, arr.length - 1);
    // 将结果拷贝到 arr 数组中
    for (int i = 0; i < result.length; i++) {
        arr[i] = result[i];
    }
}

// 对 arr 的 [start, end] 区间归并排序
private static int[] mergeSort(int[] arr, int start, int end) {
    // 只剩下一个数字，停止拆分，返回单个数字组成的数组
    if (start == end) return new int[]{arr[start]};
    int middle = (start + end) / 2;
    // 拆分左边区域
    int[] left = mergeSort(arr, start, middle);
    // 拆分右边区域
    int[] right = mergeSort(arr, middle + 1, end);
    // 合并左右区域
    return merge(left, right);
}
  ```
  {{< /code >}}
{{< /codes >}}

可以看到，我们在这个函数中，将原有数组``不断地二分``，直到``只剩下最后一个数字``。此时嵌套的递归开始返回，一层层地调用merge函数，也就是我们刚才写的将两个有序数组合并为一个有序数组的函数。

这就是``最经典的归并排序``，只需要``一个二分拆数组的递归函数``和``一个合并两个有序列表的函数``即可。

归并排序分成两步:
* ``一是拆分数组``
* ``二是合并数组``

但这份代码还有一个缺点，那就是在递归过程中，开辟了很多临时空间，接下来我们就来看下它的``优化过程``。

## 归并排序的优化：减少临时空间的开辟

为了减少在递归过程中不断开辟空间的问题，我们可以在归并排序之前，先开辟出一个临时空间，在递归过程中统一使用此空间进行归并即可。
代码如下：

{{< codes golang java>}}

  {{< code >}}

  ```golang
func mergeSort(arr []int){
	if len(arr) <= 1{
		return
	}
	//辅助数组
	res := make([]int,len(arr))
	mergeRescurison(arr,0,len(arr)-1,res)
}

func mergeRescurison(arr []int,start,end int,res []int){
	if start >= end{
		return 
	}
	middle := (start+end)/2
    // 拆分左边区域，并将归并排序的结果保存到 result 的 [start, middle] 区间
	mergeRescurison(arr,start,middle,res)
	// 拆分右边区域，并将归并排序的结果保存到 result 的 [middle + 1, end] 区间
    mergeRescurison(arr,middle+1,end,res)
    // 合并左右区域到 result 的 [start, end] 区间
    merge(arr,start,end,res)
}

// 将 result 的 [start, middle] 和 [middle + 1, end] 区间合并
func merge(arr []int,start,end int,res []int){
	middle := (start+end)/2
	// 数组 1 的首尾位置
	start1,end1 := start,middle
	// 数组 2 的首尾位置
	start2,end2 := middle+1,end
    // 用来遍历数组的指针
	index1 := start1
	index2 := start2
    // 结果数组的指针
	resIndex := start
	for index1 <= end1 && index2 <= end2{
		if arr[index1] <= arr[index2]{
			res[resIndex] = arr[index1] 
			index1++
			resIndex++
		}else{
			res[resIndex] = arr[index2] 
			index2++
			resIndex++
		}
	} 
	//把剩余的数字移到数组里面去
	for index1 <= end1{
		res[resIndex] = arr[index1] 
		index1++
		resIndex++
	}
	
	for index2 <= end2{
		res[index1+index2-start2] = arr[index2] 
		index2++
		resIndex++
	}
	// 将 result 操作区间的数字拷贝到 arr 数组中，以便下次比较
    for i := start; i <= end; i++{
        arr[i] = res[i]
	}

}
  ```
  {{< /code >}}

  {{< code >}}

  ```java
public static void mergeSort(int[] arr) {
    if (arr.length == 0) return;
    int[] result = new int[arr.length];
    mergeSort(arr, 0, arr.length - 1, result);
}

// 对 arr 的 [start, end] 区间归并排序
private static void mergeSort(int[] arr, int start, int end, int[] result) {
    // 只剩下一个数字，停止拆分
    if (start == end) return;
    int middle = (start + end) / 2;
    // 拆分左边区域，并将归并排序的结果保存到 result 的 [start, middle] 区间
    mergeSort(arr, start, middle, result);
    // 拆分右边区域，并将归并排序的结果保存到 result 的 [middle + 1, end] 区间
    mergeSort(arr, middle + 1, end, result);
    // 合并左右区域到 result 的 [start, end] 区间
    merge(arr, start, end, result);
}

// 将 result 的 [start, middle] 和 [middle + 1, end] 区间合并
private static void merge(int[] arr, int start,  int end, int[] result) {
    int middle = (start + end) / 2;
    // 数组 1 的首尾位置
    int start1 = start;
    int end1 = middle;
    // 数组 2 的首尾位置
    int start2 = middle + 1;
    int end2 = end;
    // 用来遍历数组的指针
    int index1 = start1;
    int index2 = start2;
    // 结果数组的指针
    int resultIndex = start1;
    while (index1 <= end1 && index2 <= end2) {
        if (arr[index1] <= arr[index2]) {
            result[resultIndex++] = arr[index1++];
        } else {
            result[resultIndex++] = arr[index2++];
        }
    }
    // 将剩余数字补到结果数组之后
    while (index1 <= end1) {
        result[resultIndex++] = arr[index1++];
    }
    while (index2 <= end2) {
        result[resultIndex++] = arr[index2++];
    }
    // 将 result 操作区间的数字拷贝到 arr 数组中，以便下次比较
    for (int i = start; i <= end; i++) {
        arr[i] = result[i];
    }
}
  ```
  {{< /code >}}
{{< /codes >}}

在这份代码中，我们统一使用``result 数组作为递归过程中的临时数组``，所以merge 函数接收的参数``不再是两个数组``，而是 result 数组中需要合并的``两个数组的首尾下标``。根据首尾下标可以分别计算出两个有序数组的首尾下标 start1、end1、start2、end2，之后的过程就和之前合并两个有序数组的代码类似了。

这份代码还可以``再精简一下``，我们可以``去掉一些不会改变的临时变量``。比如 start1 始终等于 start，end2 始终等于 end，end1 始终等于 middle。并且分析可知，``resultIndex 的值始终等于 start 加上 index1 和 index2 移动的距离``。即：

{{< codes golang java>}}

  {{< code >}}

  ```golang
resultIndex = start + (index1 - start1) + (index2 - start2)
  ```
  {{< /code >}}

  {{< code >}}

  ```java
resultIndex = start + (index1 - start1) + (index2 - start2)
  ```
  {{< /code >}}
{{< /codes >}}

将 start1 == start 代入，化简得：

{{< codes golang java>}}

  {{< code >}}

  ```golang
resultIndex = index1 + index2 - start2
  ```
  {{< /code >}}

  {{< code >}}

  ```java
resultIndex = index1 + index2 - start2
  ```
  {{< /code >}}
{{< /codes >}}

所以最终的归并排序代码如下：
{{< codes golang java>}}

  {{< code >}}

  ```golang
func mergeSort(arr []int){
	if len(arr) <= 1{
		return
	}
	//辅助数组
	res := make([]int,len(arr))
	mergeRescurison(arr,0,len(arr)-1,res)
}

func mergeRescurison(arr []int,start,end int,res []int){
	if start >= end{
		return 
	}
	middle := (start+end)/2
    // 拆分左边区域，并将归并排序的结果保存到 result 的 [start, middle] 区间
	mergeRescurison(arr,start,middle,res)
	// 拆分右边区域，并将归并排序的结果保存到 result 的 [middle + 1, end] 区间
    mergeRescurison(arr,middle+1,end,res)
    // 合并左右区域到 result 的 [start, end] 区间
    merge(arr,start,end,res)
}

func merge(arr []int,start,end int,res []int){
	end1 := (start+end)/2
	start2 := end1 + 1
	index1,index2:= start,start2
	//数组指针为start + (index1-start1) + (index2 -(middlex+1))
	for index1 <= end1 && index2 <= end{
		if arr[index1] <= arr[index2]{
			res[index1+index2-start2] = arr[index1] 
			index1++
		}else{
			res[index1+index2-start2] = arr[index2] 
			index2++
		}
	} 
	//把剩余的数字移到数组里面去
	for index1 <= end1{
		res[index1+index2-start2] = arr[index1] 
		index1++
	}
	
	for index2 <= end{
		res[index1+index2-start2] = arr[index2] 
		index2++
	}
	//将合并排序后的数组给arr
	for start <= end{
		arr[start] = res[start]
		start++
	}
}
  ```
  {{< /code >}}

  {{< code >}}

  ```java
public static void mergeSort(int[] arr) {
    if (arr.length == 0) return;
    int[] result = new int[arr.length];
    mergeSort(arr, 0, arr.length - 1, result);
}

// 对 arr 的 [start, end] 区间归并排序
private static void mergeSort(int[] arr, int start, int end, int[] result) {
    // 只剩下一个数字，停止拆分
    if (start == end) return;
    int middle = (start + end) / 2;
    // 拆分左边区域，并将归并排序的结果保存到 result 的 [start, middle] 区间
    mergeSort(arr, start, middle, result);
    // 拆分右边区域，并将归并排序的结果保存到 result 的 [middle + 1, end] 区间
    mergeSort(arr, middle + 1, end, result);
    // 合并左右区域到 result 的 [start, end] 区间
    merge(arr, start, end, result);
}

// 将 result 的 [start, middle] 和 [middle + 1, end] 区间合并
private static void merge(int[] arr, int start, int end, int[] result) {
    int end1 = (start + end) / 2;
    int start2 = end1 + 1;
    // 用来遍历数组的指针
    int index1 = start;
    int index2 = start2;
    while (index1 <= end1 && index2 <= end) {
        if (arr[index1] <= arr[index2]) {
            result[index1 + index2 - start2] = arr[index1++];
        } else {
            result[index1 + index2 - start2] = arr[index2++];
        }
    }
    // 将剩余数字补到结果数组之后
    while (index1 <= end1) {
        result[index1 + index2 - start2] = arr[index1++];
    }
    while (index2 <= end) {
        result[index1 + index2 - start2] = arr[index2++];
    }
    // 将 result 操作区间的数字拷贝到 arr 数组中，以便下次比较
    while (start <= end) {
        arr[start] = result[start++];
    }
}
  ```
  {{< /code >}}
{{< /codes >}}

牺牲了一些可读性，代码变得精简了不少。动图演示如下：

{{< figure src="/images/sort_gif/MER.gif" >}}

## 时间复杂度 & 空间复杂度

归并排序的复杂度比较容易分析，拆分数组的过程中，会``将数组拆分logn次``，每层执行的比较次数都``约等于n次``，所以``时间复杂度是 O(nlogn)``。

``空间复杂度是 O(n)``，主要占用空间的就是我们在排序前创建的``长度为 n 的 result 数组``。

## 稳定性

分析归并的过程可知，归并排序是一种稳定的排序算法。其中，对算法稳定性非常重要的一行代码是：

{{< codes golang java>}}

  {{< code >}}

  ```golang
if arr[index1] <= arr[index2]{
    res[index1 + index2 - start2] = arr[index1]
    index1++
}
  ```
  {{< /code >}}

  {{< code >}}

  ```java
if (arr[index1] <= arr[index2]) {
    result[index1 + index2 - start2] = arr[index1++];
}
  ```
  {{< /code >}}
{{< /codes >}}
在这里我们通过``arr[index1] <= arr[index2]``来合并两个有序数组，保证了原数组中，``相同的元素相对顺序不会变化``，如果这里的比较条件``写成了arr[index1] < arr[index2]``，则归并排序将变得不稳定。

