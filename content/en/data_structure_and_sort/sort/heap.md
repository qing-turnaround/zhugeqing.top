---
date: 2020-10-16T12:00:09+09:00
description: "战略上藐视技术，战术上重视技术——闪客"
image: "images/recommend_site/xingyouji.jpg"
title: "堆排序"
author: 诸葛青
authorEmoji: 🤖
pinned: true
tags:
- 排序算法
series:
-   
---

``数组、链表都是一维的数据结构``，相对来说比较容易理解，``而堆是二维的数据结构``，对抽象思维的要求更高，所以许多程序员「谈堆色变」。但堆又是数据结构进阶必经的一步，我们不妨静下心来，将其梳理清楚。

{{< alert theme="info" >}}
  堆：符合以下两个条件之一的完全二叉树：

* ``根节点的值 ≥ 子节点的值，这样的堆被称之为最大堆，或大顶堆（用于升序排序）``；
* ``根节点的值 ≤ 子节点的值，这样的堆被称之为最小堆，或小顶堆（用于降序排序）``。
{{< /alert >}}

{{<figure src="/images/sort_gif/heap.png">}}

堆排序是由罗伯特·弗洛伊德(Robert W．Floyd）和威廉姆斯(J．Williams） 在1964年发明的。

``堆排序过程``如下：

* ``用数列构建出一个大顶堆，取出堆顶的数字``；
* ``调整剩余的数字，构建出新的大顶堆，再次取出堆顶的数字``；
* ``循环往复，完成整个排序``。

整体的思路就是这么简单，我们需要解决的问题有两个：

* <font color=CadetBlue size=3 >如何用数列构建出一个大顶堆；</font>
* <font color=CadetBlue size=3 >取出堆顶的数字后，如何将剩余的数字调整成新的大顶堆。</font>

## 构建大顶堆 & 调整堆

构建``大顶堆``有两种方式：

* 方案一：从 0 开始，将每个数字依次插入堆中，一边插入，一边调整堆的结构，使其满足大顶堆的要求；
* 方案二：将整个数列的初始状态视作一棵完全二叉树，自底向上调整树的结构，使其满足大顶堆的要求。

``方案二更为常用``，动图演示如下：
{{<figure src="/images/sort_gif/heapbuild.gif">}}
{{<figure src="/images/sort_gif/heapify.gif">}}

<font color=CadetBlue size=3 >    </font>

在介绍堆排序具体实现之前，我们先要了解完全二叉树的几个性质。将根节点的下标视为 0，则完全二叉树有如下性质：

* ``对于完全二叉树中的第 i 个数，它的左子节点下标：left = 2i + 1``
* ``对于完全二叉树中的第 i 个数，它的右子节点下标：right = left + 1``
* ``对于有 n 个元素的完全二叉树(n≥2)(n≥2)，它的最后一个非叶子结点的下标：n/2 - 1``

<font color=CadetBlue size=3 >   </font>

堆排序代码如下：

{{< codes golang java>}}

  {{< code >}}

  ```golang
func swap(arr []int, i, j int) {
	//交换元素（异或）
	arr[i] = arr[i] ^ arr[j]
	arr[j] = arr[j] ^ arr[i]
	arr[i] = arr[i] ^ arr[j]
}


func heapSort(arr []int){
	//如何用数列构建出一个大顶堆；
	//取出堆顶的数字后，如何将剩余的数字调整成新的大顶堆。
	// 构建初始大顶堆
	buildMaxHeap(arr)

	for i:=len(arr)-1;i>0;i--{
		swap(arr,0,i) //交换元素，使得当前堆最大的元素交换到堆的末尾
		maxHeapify(arr,0,i) //从0第一个非叶子节点开始调整堆,用i是因为初建堆已经把堆的大小-1了
	}

}

func buildMaxHeap(arr []int){
	 // 从最后一个非叶子结点开始调整大顶堆，最后一个非叶子结点的下标就是 arr.length / 2-1
	 for i := len(arr) / 2 - 1; i >= 0; i--{ //i--是为了不断从前面的非叶子节点调整堆
        maxHeapify(arr,i,len(arr))
	}

}

func maxHeapify(arr []int,index ,heapSize int){
	// 左子结点下标
	l := index*2+1 //当前调整堆节点的左子节点 
	// 右子结点下标
	r := l+1 	   //当前调整堆节点的右子节点
	// 记录根结点、左子树结点、右子树结点三者中的最大值下标
	largest := index // 记录最大数的下标
	// 与左子树结点比较
	if l<heapSize && arr[l]>arr[largest]{
		largest = l
	}
	// 与右子树结点比较
	if r<heapSize && arr[r]>arr[largest]{
		largest = r
	}
	//递归向下调整
	if largest != index {//若不相等就说明
		swap(arr,largest,index)//交换元素，使其符合大顶堆的性质
		maxHeapify(arr,largest,heapSize) //递归，把最大元素和他的子树节点相比较，找出最大元素，
	}

}

  ```
  {{< /code >}}

  {{< code >}}

  ```java
public static void heapSort(int[] arr) {
    // 构建初始大顶堆
    buildMaxHeap(arr);
    for (int i = arr.length - 1; i > 0; i--) {
        // 将最大值交换到数组最后
        swap(arr, 0, i);
        // 调整剩余数组，使其满足大顶堆
        maxHeapify(arr, 0, i);
    }
}
// 构建初始大顶堆
private static void buildMaxHeap(int[] arr) {
    // 从最后一个非叶子结点开始调整大顶堆，最后一个非叶子结点的下标就是 arr.length / 2-1
    for (int i = arr.length / 2 - 1; i >= 0; i--) {
        maxHeapify(arr, i, arr.length);
    }
}
// 调整大顶堆，第三个参数表示剩余未排序的数字的数量，也就是剩余堆的大小
private static void maxHeapify(int[] arr, int i, int heapSize) {
    // 左子结点下标
    int l = 2 * i + 1;
    // 右子结点下标
    int r = l + 1;
    // 记录根结点、左子树结点、右子树结点三者中的最大值下标
    int largest = i;
    // 与左子树结点比较
    if (l < heapSize && arr[l] > arr[largest]) {
        largest = l;
    }
    // 与右子树结点比较
    if (r < heapSize && arr[r] > arr[largest]) {
        largest = r;
    }
    if (largest != i) {
        // 将最大值交换为根结点
        swap(arr, i, largest);
        // 再次调整交换数字后的大顶堆
        maxHeapify(arr, largest, heapSize);
    }
}
private static void swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}
  ```
  {{< /code >}}
{{< /codes >}}

堆排序的第一步就是``构建大顶堆``，对应代码中的``buildMaxHeap 函数``。我们将数组视作一颗完全二叉树，从它的最后一个非叶子结点开始，``调整此结点和其左右子树``，使这三个数字构成一个``大顶堆``。

需要注意的是，如果根结点和左右子树结点任何一个数字发生了交换，则还需要``保证调整后的子树仍然是大顶堆``，所以子树会执行一个``递归``的调整过程。

{{< alert theme="info" >}}
  注：在有的文章中，作者将堆的根节点下标视为 1，这样做的好处是使得第 i 个结点的左子结点下标为 2i，右子结点下标为 2i + 1，与 2i + 1 和 2i + 2 相比，计算量会少一点，本文未采取这种实现，但两种实现思路的核心思想都是一致的。
{{< /alert >}}

## 时间复杂度 & 空间复杂度

堆排序分为两个阶段：初始化建堆（buildMaxHeap）和重建堆（maxHeapify，直译为大顶堆化）。所以时间复杂度要从这两个方面分析。

根据数学运算可以推导出初始化建堆的时间复杂度为O(n)，重建堆的时间复杂度为O(nlogn)，所以堆排序总的时间复杂度为O(nlogn)。

堆排序的空间复杂度为 O(1)，只需要常数级的临时变量。

## 稳定性

堆排序在进行层层比较的时候可能会破坏原有数组中相同关键字的相对次序，``所以堆排序是不稳定的排序算法``。
