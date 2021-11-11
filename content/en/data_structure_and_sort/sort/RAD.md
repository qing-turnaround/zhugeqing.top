---
date: 2020-10-16T12:00:05+09:00
description: "战略上藐视技术，战术上重视技术——闪客"
image: "images/recommend_site/xingyouji.jpg"
title: "基数排序"
author: 诸葛青
authorEmoji: 🤖
pinned: false
tags:
- 排序算法
series:
-   
---
想一下我们是怎么``对日期进行排序``的。比如对这样三个日期进行排序：2014 年 1 月 7 日，2020 年 1 月 9 日，2020 年 7 月 10日。

我们大脑中对日期排序的``思维过程``是：

* 先看年份，2014比2020 要小，所以 2014 年这个日期应该放在其他两个日期前面。
* 另外两个日期年份相等，所以我们比较一下月份，1 比 7 要小，所以 1 月这个日期应该放在 7 月这个日期前面

这种``利用多关键字``进行排序的思想就是``基数排序``，和计数排序一样，这也是一种线性时间复杂度的排序算法。其中的每个``关键字``都被称作``一个基数``。

比如我们对 999, 997, 866, 666 这四个数字进行基数排序，过程如下：

先看第一位基数：6 比 8 小，8 比 9 小，所以 666 是最小的数字，866 是第二小的数字，暂时无法确定两个以 9 开头的数字的大小关系
再比较 9 开头的两个数字，看他们第二位基数：9 和 9 相等，暂时无法确定他们的大小关系
再比较 99 开头的两个数字，看他们的第三位基数：7 比 9 小，所以 997 小于 999

基数排序有两种实现方式。本例属于``「最高位优先法」``，简称`` MSD (Most significant digital)``，思路是``从最高位开始，依次对基数进行排序``。

与之对应的是``「最低位优先法」``，简称`` LSD (Least significant digital)``。思路是``从最低位开始，依次对基数进行排序``。使用 LSD 必须保证对基数进行排序的过程是稳定的。

``通常来讲，LSD 比 MSD 更常用``。以上述排序过程为例，因为使用的是 MSD，所以在第二步比较两个以 9 开头的数字时，其他基数开头的数字不得不放到一边。体现在计算机中，这里会``产生很多临时变量``。

但在``采用 LSD`` 进行``基数排序``时，``每一轮遍历都可以将所有数字一视同仁``，统一处理。所以 LSD 的基数排序更``符合计算机的操作习惯``。

基数排序最早是用在卡片排序机上的，一张卡片有 80 列，类似一个 80 位的整数。机器通过在卡片不同位置上穿孔表示当前基数的大小。卡片排序机的排序过程就是采用的 LSD 的基数排序。

``动图演示``
简单起见，我们先只考虑对非负整数排序的情况。

{{<figure src="/images/sort_gif/RAD.gif">}}

基数排序可以分为以下三个步骤：

* ``找出数组中最大的数字的位数 maxDigitLength``
* ``获取数组中每个数字的基数``
* ``遍历 maxDigitLength 轮数组，每轮按照基数对其进行排序``

## 找出数组中最大的数字的位数

``首先找到数组中的最大值：``

{{< codes golang java>}}

  {{< code >}}

  ```golang
func radixSort(arr []int){
    if len(arr) == 0{
        return 
    }
    max := 0
    for _,v:=range arr{
        if v > max{
            max = v
        }
    }
    //...
}
  ```
  {{< /code >}}

  {{< code >}}

  ```java
public static void radixSort(int[] arr) {
    if (arr == null) return;
    int max = 0;
    for (int value : arr) {
        if (value > max) {
            max = value;
        }
    }
    // ...
}
  ```
  {{< /code >}}
{{< /codes >}}

``通过遍历一次数组，找到了数组中的最大值 max，然后我们计算这个最大值的位数：``

{{< codes golang java>}}

  {{< code >}}

  ```golang
    maxDigitLength = 0
    for max != 0{
        maxDigitLength++//记录最大遍历次数
        max /= 10
    }

  ```
  {{< /code >}}

  {{< code >}}

  ```java
int maxDigitLength = 0;
while (max != 0) {
    maxDigitLength++;
    max /= 10;
}
  ```
  {{< /code >}}
{{< /codes >}}

将 ``maxDigitLength 初始化为 0``，然后``不断地除以10``，每除一次，``maxDigitLength 就加一``，``直到 max 为0``。

读者可能会有疑惑，如果 max 初始值就是 0 呢？严格来讲，0 在数学上属于 1 位数。

但实际上，基数排序时我们``无需考虑 max 为 0 ``的场景，因为 max 为 0 只有一种可能，那就是数组中所有的数字都为 0，``此时数组已经有序``，我们``无需再进行后续的排序过程``。

## 获取基数

获取基数有两种做法：

``第一种：``

{{< codes golang java>}}

  {{< code >}}

  ```golang
  mod := 10
  dev := 1
  for i:=0;i<maxDigitLength;i++{
    for _,v := range arr{
      radix := v % mod / dev 
      //对基数进行排序
    }
    mod *= 10
    dev *= 10
  }
  ```
  {{< /code >}}

  {{< code >}}

  ```java
int mod = 10;
int dev = 1;
for (int i = 0; i < maxDigitLength; i++) {
    for (int value : arr) {
        int radix = value % mod / dev;
        // 对基数进行排序
    }
    mod *= 10;
    dev *= 10;
}
  ```
  {{< /code >}}
{{< /codes >}}

``第二种：``

{{< codes golang java>}}

  {{< code >}}

  ```golang
  dev := 1
  for i:=0;i<maxDigitLength;i++{
    for _,v := range arr{
      radix := v / dev % 10 
      //对基数进行排序
    }
    dev *= 10
  }
  ```
  {{< /code >}}

  {{< code >}}

  ```java
int dev = 1;
for (int i = 0; i < maxDigitLength; i++) {
    for (int value : arr) {
        int radix = value / dev % 10;
        // 对基数进行排序
    }
    dev *= 10;
}
  ```
  {{< /code >}}
{{< /codes >}}

两者的区别是先做除法运算还是先做模运算，``推荐使用第二种写法，因为它可以节省一个变量``

## 对基数进行排序

{{< codes golang java>}}

  {{< code >}}

  ```golang
func radixSort(arr []int) {
	//第一步，先确定基数个数
	if len(arr) == 0 {
		return
	}
	max := 0
	for _, v := range arr {
		if v > max {
			max = v
		}
	}

	maxLength := 0
	for max != 0 {
		maxLength++
		max /= 10
	}

	//结果暂存
	res := make([]int, len(arr))
	dev := 1 //从1开始
	for i := 0; i < maxLength; i++ {
		//记录各基数个数
		count := make([]int, 10)
		for _, v := range arr {
			radix := v / dev % 10 //求当前数的基数
			count[radix]++        //记录基数个数
		}
		//counting[i] 更新为数字 i 在最终排序结果中的起始下标位置。
		//这个位置等于前面比自己小的数字的总数
		fmt.Println(count)
		for i := 1; i < len(count); i++ {
			count[i] += count[i-1]
		}
		fmt.Println(count)
		for i := len(arr) - 1; i >= 0; i-- {
			radix := arr[i] / dev % 10
			count[radix]--
			res[count[radix]] = arr[i]
		}

		for i, v := range res {
			arr[i] = v
		}
		dev *= 10
		fmt.Println(arr)
	}

}
  ```
  {{< /code >}}

  {{< code >}}

  ```java
public class RadixSort {

    public static void radixSort(int[] arr) {
        if (arr == null) return;
        // 找出最大值
        int max = 0;
        for (int value : arr) {
            if (value > max) {
                max = value;
            }
        }
        // 计算最大数字的长度
        int maxDigitLength = 0;
        while (max != 0) {
            maxDigitLength++;
            max /= 10;
        }
        // 使用计数排序算法对基数进行排序
        int[] counting = new int[10];
        int[] result = new int[arr.length];
        int dev = 1;
        for (int i = 0; i < maxDigitLength; i++) {
            for (int value : arr) {
                int radix = value / dev % 10;
                counting[radix]++;
            }
            for (int j = 1; j < counting.length; j++) {
                counting[j] += counting[j - 1];
            }
            // 使用倒序遍历的方式完成计数排序
            for (int j = arr.length - 1; j >= 0; j--) {
                int radix = arr[j] / dev % 10;
                result[--counting[radix]] = arr[j];
            }
            // 计数排序完成后，将结果拷贝回 arr 数组
            System.arraycopy(result, 0, arr, 0, arr.length);
            // 将计数数组重置为 0
            Arrays.fill(counting, 0);
            dev *= 10;
        }
    }
}
  ```
  {{< /code >}}
{{< /codes >}}

首先我们将``每位元素出现的次数记录到 counting 数组中``

然后将 ``counting[i] 更新为数字 i 在最终排序结果中的起始下标位置``。``这个位置等于前面比自己小的数字的总数``。

我们将 ``result`` 数组的值拷贝回 ``arr`` 数组，并且将 ``counting`` 数组中的元素都``置为 0``，以便在``下一轮中复用``。

## 对包含负数的数组进行基数排序

如果数组中包含``负数``，如何进行``基数排序``呢？

我们很容易想到``一种思路``：将数组中的每个元素都加上一个合适的正整数，使其全部变成非负整数，等到排序完成后，再减去之前加的这个数就可以了。

但这种方案有一个缺点：加法运算``可能导致数字越界``，所以必须单独处理数字越界的情况。

事实上，``有一种更好的方案解决负数的基数排序``。那就是在对基数进行计数排序时，``申请长度为 19 的计数数组``，用来存储 [-9, 9] 这个区间内的所有整数。在把每一位基数计算出来后，加上 9，就能``对应上 counting 数组的下标``了。也就是说，counting 数组的 ``下标 [0, 18] 对应基数 [−9,9]``。


{{< codes golang java>}}

  {{< code >}}

  ```golang
func radixSort(arr []int) {
	//第一步，先确定基数个数
	if len(arr) == 0 {
		return
	}
	max := 0
	for _, v := range arr {
		if v < 0 {
			v = -v
		}
		if v > max {
			max = v
		}
	}

	maxLength := 0
	for max != 0 {
		maxLength++
		max /= 10
	}

	//结果暂存
	res := make([]int, len(arr))
	dev := 1 //从1开始
	for i := 0; i < maxLength; i++ {
		//记录各基数个数
		count := make([]int, 19)
		for _, v := range arr {
			radix := v/dev%10 + 9 //求当前数的基数
			count[radix]++        //记录基数个数
		}
		//counting[i] 更新为数字 i 在最终排序结果中的起始下标位置。
		//这个位置等于前面比自己小的数字的总数
		fmt.Println(count)
		for i := 1; i < len(count); i++ {
			count[i] += count[i-1]
		}
		fmt.Println(count)
		for i := len(arr) - 1; i >= 0; i-- {
			radix := arr[i]/dev%10 + 9
			count[radix]--
			res[count[radix]] = arr[i]
		}

		for i, v := range res {
			arr[i] = v
		}
		dev *= 10
		fmt.Println(arr)
	}

}
  ```
  {{< /code >}}

  {{< code >}}

  ```java
public class RadixSort {

    public static void radixSort(int[] arr) {
        if (arr == null) return;
        // 找出最长的数
        int max = 0;
        for (int value : arr) {
            if (Math.abs(value) > max) {
                max = Math.abs(value);
            }
        }
        // 计算最长数字的长度
        int maxDigitLength = 0;
        while (max != 0) {
            maxDigitLength++;
            max /= 10;
        }
        // 使用计数排序算法对基数进行排序，下标 [0, 18] 对应基数 [-9, 9]
        int[] counting = new int[19];
        int[] result = new int[arr.length];
        int dev = 1;
        for (int i = 0; i < maxDigitLength; i++) {
            for (int value : arr) {
                // 下标调整
                int radix = value / dev % 10 + 9;
                counting[radix]++;
            }
            for (int j = 1; j < counting.length; j++) {
                counting[j] += counting[j - 1];
            }
            // 使用倒序遍历的方式完成计数排序
            for (int j = arr.length - 1; j >= 0; j--) {
                // 下标调整
                int radix = arr[j] / dev % 10 + 9;
                result[--counting[radix]] = arr[j];
            }
            // 计数排序完成后，将结果拷贝回 arr 数组
            System.arraycopy(result, 0, arr, 0, arr.length);
            // 将计数数组重置为 0
            Arrays.fill(counting, 0);
            dev *= 10;
        }
    }
}
  ```
  {{< /code >}}
{{< /codes >}}

代码中主要做了``两处修改``：

* ``当数组中存在负数时，我们就不能简单的计算数组的最大值了，而是要计算数组中绝对值最大的数，也就是数组中最长的数``
* ``在获取基数的步骤，将计算出的基数加上 99，使其与 counting 数组下标一一对应``

## 时间复杂度 & 空间复杂度

无论 LSD 还是 MSD，基数排序时``都需要经历 maxDigitLength 轮遍历``，每轮遍历的``时间复杂度为 O(n + k)`` ，其中 ``k`` 表示每个基数可能的``取值范围大小``。如果是对``非负整数排序``，则 ``k = 10``，如果是对``包含负数的数组排序``，则 ``k = 19``。

``所以基数排序的时间复杂度为 O(d(n + k))表示最长数字的位数，k 表示每个基数可能的取值范围大小)。``

## 稳定性

``基数排序是一种稳定的排序算法``