---
date: 2020-10-16T12:00:13+09:00
description: "战略上藐视技术，战术上重视技术——闪客"
image: "images/recommend_site/xingyouji.jpg"
title: "冒泡排序"
author: 诸葛青
authorEmoji: 🤖
pinned: false
tags:
- 排序算法
series:
-   
---
冒泡排序是入门级的算法，但也有一些有趣的玩法。通常来说，冒泡排序有三种写法：
* <font color=Aqua size=3 >一边比较一边向后两两交换，将最大值 / 最小值冒泡到最后一位；</font>
* <font color=Aqua size=3 >经过优化的写法：使用一个变量记录当前轮次的比较是否发生过交换，如果没有发生交换表示已经有序，不再继续排序；</font>
* <font color=Aqua size=3 >进一步优化的写法：除了使用变量记录当前轮次是否发生交换外，再使用一个变量记录上次发生交换的位置，下一轮排序时到达上次交换的位置就停止比较。</font>

<font color=GreenYellow size=3 >先来看看冒泡排序的动图吧！</font>
{{<figure src="/images/sort_gif/BUB.gif">}}

### 冒泡排序的第一种写法

{{< codes golang java>}}
  {{< code >}}
```golang
//升序排序
func swap(arr []int, i, j int) {
	//交换元素（异或）
	arr[i] = arr[i] ^ arr[j]
	arr[j] = arr[j] ^ arr[i]
	arr[i] = arr[i] ^ arr[j]
}

func bubbleSort(arr []int) {
	for i := 0; i < len(arr)-1; i++ {
		for j := 0; j < len(arr)-i-1; j++ {
			if arr[j] > arr[j+1] {
				swap(arr, j, j+1)
			}
		}
	}
}
```
  {{< /code >}}

   {{< code >}}

  ```java
public static void bubbleSort(int[] arr) {
    for (int i = 0; i < arr.length - 1; i++) {
        for (int j = 0; j < arr.length - 1 - i; j++) {
            if (arr[j] > arr[j + 1]) {
                // 如果左边的数大于右边的数，则交换，保证右边的数字最大
                swap(arr, j, j + 1);
            }
        }
    }
}
// 交换元素
private static void swap(int[] arr, int i, int j) {
    arr[i] = arr[i] ^ arr[j];
    arr[j] = arr[j] ^ arr[i];
    arr[i] = arr[i] ^ arr[j];
}
  ```
  {{< /code >}}
{{< /codes >}}
### 冒泡排序的第二种写法

{{< codes golang java>}}

  {{< code >}}

  ```golang
func swap(arr []int, i, j int) {
	//交换元素（异或）
	arr[i] = arr[i] ^ arr[j]
	arr[j] = arr[j] ^ arr[i]
	arr[i] = arr[i] ^ arr[j]
}

//经过优化的写法：使用一个变量记录当前轮次的比较是否发生过交换，如果没有发生交换表示已经有序，不再继续排序；
func bubbleSort(arr []int) {
	swapped := true //记录是否发生交换
	for i := 0; i < len(arr)-1; i++ {
		if !swapped { //若上一轮未发生交换，就退出排序
			break
		}
		swapped = false
		for j := 0; j < len(arr)-i-1; j++ {
			if arr[j] > arr[j+1] {
				swap(arr, j, j+1)
				swapped = true //发生交换
			}
		}
	}
}

  ```
  {{< /code >}}

     {{< code >}}

  ```java
public static void bubbleSort(int[] arr) {
    // 初始时 swapped 为 true，否则排序过程无法启动
    boolean swapped = true;
    for (int i = 0; i < arr.length - 1; i++) {
        // 如果没有发生过交换，说明剩余部分已经有序，排序完成
        if (!swapped) break;
        // 设置 swapped 为 false，如果发生交换，则将其置为 true
        swapped = false;
        for (int j = 0; j < arr.length - 1 - i; j++) {
            if (arr[j] > arr[j + 1]) {
                // 如果左边的数大于右边的数，则交换，保证右边的数字最大
                swap(arr, j, j + 1);
                // 表示发生了交换
                swapped = true;
            }
        }
    }
}
// 交换元素
private static void swap(int[] arr, int i, int j) {
    arr[i] = arr[i] ^ arr[j];
    arr[j] = arr[j] ^ arr[i];
    arr[i] = arr[i] ^ arr[j];
}
  ```
  {{< /code >}}

{{< /codes >}}

### 冒泡排序的第三种写法
{{< codes golang java>}}

  {{< code >}}

  ```golang
func swap(arr []int, i, j int) {
	//交换元素（异或）
	arr[i] = arr[i] ^ arr[j]
	arr[j] = arr[j] ^ arr[i]
	arr[i] = arr[i] ^ arr[j]
}

//进一步优化的写法：除了使用变量记录当前轮次是否发生交换外，
//再使用一个变量记录上次发生交换的位置，下一轮排序时到达上次交换的位置就停止比较
func bubbleSort(arr []int) {
	swapped := true //记录是否发生交换
	indexOfLastUnsortedElement := len(arr) - 1
	// 上次发生交换的位置
	swappedIndex := -1
	for swapped {
		swapped = false
		for i := 0; i < indexOfLastUnsortedElement; i++ {
			if arr[i] > arr[i+1] {
				swap(arr, i, i+1)
				swapped = true //发生交换
				swappedIndex = i
			}
		}
		indexOfLastUnsortedElement = swappedIndex
	}
}
  ```
  {{< /code >}}

     {{< code >}}

  ```java
public static void bubbleSort(int[] arr) {
    boolean swapped = true;
    // 最后一个没有经过排序的元素的下标
    int indexOfLastUnsortedElement = arr.length - 1;
    // 上次发生交换的位置
    int swappedIndex = -1;
    while (swapped) {
        swapped = false;
        for (int i = 0; i < indexOfLastUnsortedElement; i++) {
            if (arr[i] > arr[i + 1]) {
                // 如果左边的数大于右边的数，则交换，保证右边的数字最大
                swap(arr, i, i + 1);
                // 表示发生了交换
                swapped = true;
                // 更新交换的位置
                swappedIndex = i;
            }
        }
        // 最后一个没有经过排序的元素的下标就是最后一次发生交换的位置
        indexOfLastUnsortedElement = swappedIndex;
    }
}
// 交换元素
private static void swap(int[] arr, int i, int j) {
    arr[i] = arr[i] ^ arr[j];
    arr[j] = arr[j] ^ arr[i];
    arr[i] = arr[i] ^ arr[j];
}
  ```
  {{< /code >}}

{{< /codes >}}

## 时间复杂度 & 空间复杂度
冒泡排序从 1956 年就有人开始研究，之后经历过多次优化。它的<font color=CadetBlue size=3 >空间复杂度为 O(1)，时间复杂度为 O(n^2)</font>，第二种、第三种冒泡排序由于经过优化，最好的情况下只需要 O(n)的时间复杂度。

<font color=GreenYellow size=3 >最好情况：在数组已经有序的情况下，只需遍历一次，由于没有发生交换，排序结束。</font>

<font color=GreenYellow size=3 >最差情况：数组顺序为逆序，每次比较都会发生交换。</font>

但优化后的冒泡排序平均时间复杂度仍然是 O(n^2)，所以这些优化对算法的性能并没有质的提升。正如 Donald E. Knuth（1974 年图灵奖获得者）所言：“冒泡排序法除了它迷人的名字和导致了某些有趣的理论问题这一事实外，似乎没有什么值得推荐的。”

不管怎么说，冒泡排序法是所有排序算法的老祖宗，如同程序界经典的 「Hello, world」 一般经久不衰，总是出现在各类算法书刊的首个章节。但面试时如果你说你只会冒泡排序可就太掉价了，下一节我们就来认识一下他的继承者们。

## 稳定性

冒泡排序是稳定的。
