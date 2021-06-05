---
date: 2020-10-16T12:00:10+09:00
description: "战略上藐视技术，战术上重视技术——闪客"
image: "images/recommend_site/xingyouji.jpg"
title: "希尔排序"
author: 诸葛青
authorEmoji: 🤖
pinned: false
tags:
- 排序算法
series:
-   
---
1959 年 77 月，美国辛辛那提大学的数学系博士 Donald Shell 在 《ACM 通讯》上发表了希尔排序算法，成为首批将时间复杂度降到 O(n^2)以下的算法之一。虽然原始的希尔排序最坏时间复杂度仍然是 O(n^2) ，但经过优化的希尔排序可以达到 O(n^(1.3))甚至 O(n^(7/6))。

略为遗憾的是，所谓「一将功成万骨枯」，希尔排序和冒泡、选择、插入等排序算法一样，逐渐被快速排序所淘汰，但作为承上启下的算法，不可否认的是，希尔排序身上始终闪耀着算法之美。

<font color=GreenYellow size=3 >希尔排序本质上是对插入排序的一种优化，它利用了插入排序的简单，又克服了插入排序每次只交换相邻两个元素的缺点。它的基本思想是</font>：
* <font color=CadetBlue size=3 >将待排序数组按照一定的间隔分为多个子数组，每组分别进行插入排序。这里按照间隔分组指的不是取连续的一段数组，而是每跳跃一定间隔取一个值组成一组</font>
* <font color=CadetBlue size=3 >逐渐缩小间隔进行下一轮排序</font>
* <font color=CadetBlue size=3 >最后一轮时，取间隔为 1，也就相当于直接使用插入排序。但这时经过前面的「宏观调控」，数组已经基本有序了，所以此时的插入排序只需进行少量交换便可完成</font>

举个例子，对数组 <font color=Turquoise size=4>[84, 83, 88, 87, 61, 50, 70, 60, 80, 99]</font>进行希尔排序的过程如下：

* <font color=GreenYellow size=3 >第一遍（5间隔排序）</font>：按照间隔 5分割子数组，共分成五组，分别<font color=Turquoise size=4>[84, 50], [83, 70], [88, 60], [87, 80], [61, 99]</font> 。对它们进行插入排序，排序后它们分别变成：<font color=Turquoise size=4>[50, 84], [70, 83], [60, 88], [80, 87], [61, 99]</font>。此时整个数组变成 <font color=Turquoise size=4>[50, 70, 60, 80, 61, 84, 83, 88, 87, 99]</font>。
* <font color=GreenYellow size=3 >第二遍（2间隔排序）</font>：按照间隔 2分割子数组，共分成两组，分别是 <font color=Turquoise size=4>[50, 60, 61, 83, 87], [70, 80, 84, 88, 99]</font>。对他们进行插入排序，排序后它们分别变成： <font color=Turquoise size=4>[50, 60, 61, 83, 87], [70, 80, 84, 88, 99]</font>，此时整个数组变成<font color=Turquoise size=4>[50, 70, 60, 80, 61, 84, 83, 88, 87, 99]</font>。<font color=VioletRed size=3 >这里有一个非常重要的性质：当我们完成2间隔排序后，这个数组仍然是保持 5间隔有序的。也就是说，更小间隔的排序没有把上一步的结果变坏。</font>
* <font color=GreenYellow size=3 >第三遍（1间隔排序，等于直接插入排序）</font>：按照间隔 1分割子数组，分成一组，也就是整个数组。对其进行插入排序，经过前两遍排序，数组已经基本有序了，所以这一步只需经过少量交换即可完成排序。排序后数组变成 <font color=Turquoise size=4>[50, 60, 61, 70, 80, 83, 84, 87, 88, 99]</font>，整个排序完成。

<font color=GreenYellow size=3 >先来看看希尔排序的动图吧！</font>

{{<figure src="/images/sort_gif/SHE.gif">}}

<font color=GreenYellow size=3 >   </font>

其中，每一遍<font color=CadetBlue size=3 >排序的间隔</font>在希尔排序中被称之为<font color=CadetBlue size=3 >增量</font>，所有的增量组成的序列称之为<font color=CadetBlue size=3 >增量序列</font>，也就是本例中的 [5, 2, 1][5,2,1]。增量依次递减，最后一个增量必须为 11，<font color=CadetBlue size=3 >所以希尔排序又被称之为「缩小增量排序」</font>。要是以专业术语来描述希尔排序，可以分为以下两个步骤：

* <font color=GreenYellow size=3 >定义增量序列 D_m > D_{m-1} > D_{m-2} > ... > D_1 = 1</font>

* <font color=GreenYellow size=3 >对每个 D_k进行[D_k间隔排序]</font>

有一条非常重要的性质保证了希尔排序的效率：

* <font color=GreenYellow size=3 >[D_{k+1}间隔]有序的序列，在经过[D_k间隔]排序后，仍然是 [D_{k+1}间隔]有序的
</font>

<font color=CadetBlue size=3 >增量序列的选择会极大地影响希尔排序的效率</font>，我们采用的增量序列为 D_m = N/2, D_k = D_{k+1} / 2,这个序列正是当年希尔发表此算法的论文时选用的序列，所以也被称之为<font color=CadetBlue size=3 >希尔增量序列</font>

{{< codes golang java>}}

  {{< code >}}

  ```golang
func shellSort(arr []int){
		//生成间隔序列，在希尔排序中我们称之为增量序列
	for gap:=len(arr)/2;gap>0;gap/=2{ 
		//将一个数组转换成gap个数组（每一个数组各个元素之间的距离相差gap）
		for groupStartIndex:=0;groupStartIndex<gap;groupStartIndex++{	
		//插入排序(i:=groupStartIndex+gap就相当于插入排序的i:=1)
			for i:=groupStartIndex+gap;i<len(arr);i+=gap{
				currentNumber:=arr[i] 
				j:=i-gap
				for j >=0 && currentNumber < arr[j]{
					arr[j+gap] = arr[j]
					j -= gap 
				} 
				arr[j+gap] = currentNumber
			}	
		}
	} 
}
  ```
  {{< /code >}}

  {{< code >}}

  ```java
public static void shellSort(int[] arr) {
    // 间隔序列，在希尔排序中我们称之为增量序列
    for (int gap = arr.length / 2; gap > 0; gap /= 2) {
        // 分组
        for (int groupStartIndex = 0; groupStartIndex < gap; groupStartIndex++) {
            // 插入排序
            for (int currentIndex = groupStartIndex + gap; currentIndex < arr.length; currentIndex += gap) {
                // currentNumber 站起来，开始找位置
                int currentNumber = arr[currentIndex];
                int preIndex = currentIndex - gap;
                while (preIndex >= groupStartIndex && currentNumber < arr[preIndex]) {
                    // 向后挪位置
                    arr[preIndex + gap] = arr[preIndex];
                    preIndex -= gap;
                }
                // currentNumber 找到了自己的位置，坐下
                arr[preIndex + gap] = currentNumber;
            }
        }
    }
}
  ```
  {{< /code >}}
{{< /codes >}}


这段代码可以优化一下。我们现在的处理方式是：<font color=CadetBlue size=3 >处理完一组间隔序列后，再回来处理下一组间隔序列</font>，这非常符合人类思维。但对于计算机来说，<font color=CadetBlue size=3 >它更喜欢从第 gap 个元素开始，按照顺序将每个元素依次向前插入自己所在的组这种方式。</font>虽然这个过程看起来是在不同的间隔序列中不断跳跃，但站在计算机的角度，它是在访问一段连续数组。

{{< codes golang java>}}

  {{< code >}}

  ```golang
func shellSort(arr []int){
	//创建 间隔序列（增量序列）
	for gap:=len(arr)/2;gap>0;gap/=2{
		for i:=gap;i<len(arr);i++{
			currentNumber := arr[i]//记录当前需要插入的数
			j := i-gap//记录当前下标的前一个下标
			for j>=0 && arr[j] > currentNumber{
				arr[j+gap] = arr[j] //后移元素
				j -= gap //更新下标
			}
			arr[j+gap] = currentNumber
		}

	}
}
  ```
  {{< /code >}}

  {{< code >}}

  ```java
public static void shellSort(int[] arr) {
    // 间隔序列，在希尔排序中我们称之为增量序列
    for (int gap = arr.length / 2; gap > 0; gap /= 2) {
        // 从 gap 开始，按照顺序将每个元素依次向前插入自己所在的组
        for (int i = gap; i < arr.length; i++) {
            // currentNumber 站起来，开始找位置
            int currentNumber = arr[i];
            // 该组前一个数字的索引
            int preIndex = i - gap;
            while (preIndex >= 0 && currentNumber < arr[preIndex]) {
                // 向后挪位置
                arr[preIndex + gap] = arr[preIndex];
                preIndex -= gap;
            }
            // currentNumber 找到了自己的位置，坐下
            arr[preIndex + gap] = currentNumber;
        }
    }
}
  ```
  {{< /code >}}
{{< /codes >}}

经过优化之后，这段代码看起来就和插入排序非常相似了，<font color=CadetBlue size=3 >区别仅在于希尔排序最外层嵌套了一个缩小增量的 for 循环；并且插入时不再是相邻数字挪动，而是以增量为步长挪动</font>。


## 增量序列

<font color=CadetBlue size=3 >增量序列的选择会极大地影响希尔排序的效率。增量序列如果选得不好，希尔排序的效率可能比插入排序效率还要低，</font>举个例子：

{{<figure src="/images/sort_gif/shell.png">}}

<font color=LightSeaGreen size=3 >   </font>

在这个例子中，我们发现，原数组 8 间隔、4 间隔、2 间隔都已经有序了，使用希尔排序时，真正起作用的只有最后一轮 1 间隔排序，也就是直接插入排序。希尔排序反而比直接使用插入排序多执行了许多无用的逻辑。

于是人们发现：<font color=CadetBlue size=3 >增量元素不互质，则小增量可能根本不起作用</font>。

事实上，希尔排序的增量序列如何选择是一个数学界的难题，但它也是希尔排序算法的核心优化点。数学界有不少的大牛做过这方面的研究。比较著名的有 Hibbard 增量序列、Knuth 增量序列、Sedgewick 增量序列。

* <font color=VioletRed size=3 >Hibbard 增量序列</font>：D_k = 2^k - 1,也就是1, 3, 7, 15,...。数学界猜想它最坏的时间复杂度为 O(n^(3/2))，平均时间复杂度为 O(n^(5/4)); 

* <font color=VioletRed size=3 >Knuth 增量序列</font>：D_1 = 1; D_{k+1} = 3 * D_{k + 1},也就是1, 4, 13, 40, ...,数学界猜想它的平均时间复杂度为 O(n^(3/2))

* <font color=VioletRed size=3 >Sedgewick 增量序列</font>：1,5,19,41,109,...,这个序列的元素有的是通过 9 * 4^k - 9 * 2^k + 1 计算出来的，有的是通过 4^k - 3 * 2^k + 1 计算出来的。数学界猜想它最坏的时间复杂度为 O(n^(4/3)),平均时间复杂度为 O(n^(7/6)).

使用 Knuth 序列进行希尔排序的代码如下：

{{< codes golang java>}}

  {{< code >}}

  ```golang
func shellSort(arr []int){
	// 找到当前数组需要用到的 Knuth 序列中的最大值
	maxKnuthNumber := 1;
	for maxKnuthNumber <= len(arr) / 3 {
		  maxKnuthNumber = maxKnuthNumber * 3 + 1
	}
	//创建 间隔序列（增量序列）
	for gap:=maxKnuthNumber;gap>0;gap = (gap-1)/3{
		// 从 gap 开始，按照顺序将每个元素依次向前插入自己所在的组
		for i:=gap;i<len(arr);i++{
			currentNumber := arr[i]
			j := i - gap //记录当前元素下标的上一个下标
			for j>=0 && arr[j] > currentNumber{
				arr[j+gap] = arr[j] //后移元素
				j -= gap //更新下标
			}
            arr[j+gap] = currentNumber
		}
	}

}
  ```
  {{< /code >}}

  {{< code >}}

  ```java
public static void shellSortByKnuth(int[] arr) {
    // 找到当前数组需要用到的 Knuth 序列中的最大值
    int maxKnuthNumber = 1;
    while (maxKnuthNumber <= arr.length / 3) {
        maxKnuthNumber = maxKnuthNumber * 3 + 1;
    }
    // 增量按照 Knuth 序列规则依次递减
    for (int gap = maxKnuthNumber; gap > 0; gap = (gap - 1) / 3) {
        // 从 gap 开始，按照顺序将每个元素依次向前插入自己所在的组
        for (int i = gap; i < arr.length; i++) {
            // currentNumber 站起来，开始找位置
            int currentNumber = arr[i];
            // 该组前一个数字的索引
            int preIndex = i - gap;
            while (preIndex >= 0 && currentNumber < arr[preIndex]) {
                // 向后挪位置
                arr[preIndex + gap] = arr[preIndex];
                preIndex -= gap;
            }
            // currentNumber 找到了自己的位置，坐下
            arr[preIndex + gap] = currentNumber;
        }
    }
}
  ```
  {{< /code >}}
{{< /codes >}}

先根据数组的长度，计算出需要用到的 Knuth 序列中的最大增量值，然后根据 Knuth 序列的规则依次缩小增量，从高增量到低增量分别进行排序。

使用 Knuth 序列的希尔排序，时间复杂度已经降到了 O(n^2)以下。但具体时间复杂度是多少，尚未有明确的证明，数学界仅仅是猜想它的平均时间复杂度为 O(n^(3/2))

## 时间复杂度 & 空间复杂度

事实上，希尔排序时间复杂度非常难以分析，<font color=Aqua size=3 >它的平均复杂度界于 O(n) 到 O(n^2)之间</font>，普遍认为它最好的时间复杂度为 <font color=Aqua size=3 >O(n^(1.3))</font>，希尔排序的空间复杂度为<font color=Aqua size=3 >O(1)</font>只需要常数级的临时变量。

## 稳定性

虽然插入排序是稳定的排序算法，<font color=CadetBlue size=3 >但希尔排序是不稳定的。在增量较大时，排序过程可能会破坏原有数组中相同关键字的相对次序。</font>


## 希尔排序与 O(n^2)级排序算法的本质区别

相对于前面介绍的冒泡排序、选择排序、插入排序来说，希尔排序的排序过程显得较为复杂，希望读者还没有被绕晕。接下来我们来分析一个有趣的问题：<font color=CadetBlue size=3 >希尔排序凭什么可以打破时间复杂度O(n^2)的魔咒呢？它和之前介绍的O(n^2)级排序算法的本质区别是什么？</font>
<font color=VioletRed size=3 >只要理解了这一点，我们就能知道为什么希尔排序能够承上启下，启发出之后的一系列 O(n^2)级以下的排序算法。</font>

<font color=GreenYellow size=3 >这个问题我们可以用逆序对来理解。</font>

{{< alert theme="info" >}}
  当我们从小到大排序时，在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。
{{< /alert >}}

<font color=GreenYellow size=3 >排序算法本质上就是一个消除逆序对的过程。</font>

对于随机数组，<font color=GreenYellow size=3 >逆序对的数量是 O(n^2)级的</font>，如果采用<font color=GreenYellow size=3 >「交换相邻元素」</font>的办法来消除逆序对，每次<font color=GreenYellow size=3 >最多只能消除一组逆序对</font>，因此必须执行 O(n^2)级的交换次数，这就是为什么<font color=CadetBlue size=3 >冒泡、插入、选择</font>算法只能到 O(n^2)级的原因。反过来说，基于交换元素的排序算法要想突破 O(n^2)级，必须通过一些比较，<font color=GreenYellow size=3 >交换间隔比较远的元素，使得一次交换能消除一个以上的逆序对</font>。

<font color=Aqua size=3 >希尔排序算法就是通过这种方式，打破了在空间复杂度为 O(1)的情况下，时间复杂度为 O(n^2)的魔咒，此后的快排、堆排等等算法也都是基于这样的思路实现的。</font>


{{< alert theme="info" >}}
注：

1.虽然约翰·冯·诺伊曼在 1945 年提出的归并排序已经达到了O(nlogn) 的时间复杂度，但归并排序的空间复杂度为 O(n)，采用的是空间换时间的方式突破 O(n^2)。

2.希尔排序在面试或是实际应用中都很少遇到，读者仅需了解即可。

{{< /alert >}}

