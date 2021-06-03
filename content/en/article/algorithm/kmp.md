---
date: 2021-05-06
description: "一文解析KMP算法"
image: "/images/xingyouji.jpg"
title: "图解|KMP算法"
author: 诸葛青
authorEmoji: 🎅
pinned: true
tags:
- 算法
series:
-   
---

<font style="color: red">此文章选自微信公众号“景禹”</font>


>以前的计算机刚被发明的时候，主要作用是做一些科学和工程的计算工作，科学家发明计算机的时候压根儿不可能想到后人还可以用来KMP。
>刚开始的计算机都是处理数值工作，后来引入了字符串的概念，计算机开始可以处理非数值的概念了（当然原理还是用数值来模拟非数值，通过ASCII码表）。
>总之在工作当中字符串的处理操作非常普遍，今日主要分享字符串模式匹配算法KMP的相关操作。
>在分享KMP算法之前，我们先看一下蛮力法进行模式匹配的过程：

![](/images/article/10.gif)

对应[力扣28. 实现 strStr()](https://leetcode-cn.com/problems/implement-strstr/)

### 代码实现如下

{{< codes c golang>}}
  {{< code >}}

```c
int strStr(char* haystack, char* needle) {
    //暴力法
    //haystack为主串，needle为模式串
    int n = strlen(haystack), m = strlen(needle);
    for (int i = 0; i <= n-m; i++) {
        int flag = 1;//标记是否匹配成功
        for (int j = 0; j < m; j++) {
            if (haystack[i + j] != needle[j]) {
                flag = 0;
                break;
            }
        }
        if (flag) { //成功返回i下标
            return i;
        }
    }
    return -1; //匹配失败
}
```

  {{< /code >}}

  {{< code >}}

  ```golang
func strStr(haystack string, needle string) int {
    //暴力法
    //haystack为主串，needle为模式串
    j:=0 //遍历模式串的下标
    for i:=0;i<=len(haystack)-len(needle);i++{
        for j=0;j<len(needle);j++{
            if haystack[i+j] != needle[j]{
                break //不匹配，主串从下一个下标再开始匹配
            }
        }

        if j == len(needle){
            return i //匹配成功
        }
    }

    return -1
}
  ```
  {{< /code >}}
{{< /codes >}}

显然蛮力法的执行效率太低了，为此有大佬提出了``KMP算法``。在详细介绍KMP算法之前，我们看一下字符串的``前缀``与``后缀``的概念：

![](/images/article/1.webp)

有了字符串前缀与后缀的概念，我们就可以计算出一个字符串前缀与后缀的``公共子串``的``最大长度``。

![](/images/article/2.png)

此时，我们就可以来看``KMP算法``的``执行过程``了。

``第一步：``

![](/images/article/3.webp)

``第二步：``

![](/images/article/4.webp)

``第三步：``

![](/images/article/1.png)

``第四步：``

![](/images/article/3.png)

``第五步：``

![](/images/article/4.png)

``理解KMP算法的执行过程中，一定要注意景禹在图片中标注的文字。最后我们来看一下动图演示：``

![](/images/article/2.gif)

### 代码实现如下

{{< codes c golang>}}
  {{< code >}}

```c
int* getNext(char *needle){
    int n = strlen(needle);
    int* next = (int*)malloc(sizeof(int)*n); //为next数组分配空间
    int maxMatch = 0; //最大匹配长度
    next[0] = 0;
    for(int i=1;i<n;i++){//下标0没有对应的前缀和后缀，直接从1开始
        while(maxMatch>0 && needle[i] != needle[maxMatch]){
            maxMatch = next[maxMatch-1]; //如果不匹配就用之前的next值直到匹配或者maxMatch再次为0
        }

        if (needle[i] == needle[maxMatch]){//匹配成功直接加一
            maxMatch++;
        }
        next[i] = maxMatch;//此下标的next值就等于当前匹配的最大值
    }

    return next;
}

int strStr(char * haystack, char * needle){
    //KMP算法
    //haystack为主串，needle为模式串
    int n = strlen(haystack), m = strlen(needle);//主串和模式串的长度
    
    if(m==0){
        return 0;
    }

    int* next = getNext(needle);
    int j = 0;//遍历模式串
    for(int i=0;i<n;i++){//遍历主串
        while(j>0 && haystack[i] != needle[j]){
            j = next[j-1];//不匹配，寻找下一次匹配需要的next值
        }

        if(haystack[i] == needle[j]){
            j++;
        }

        if(j == m){ //当j等于模式串长度时，说明已经完全匹配
            return i-j+1;
        }
     }

    return -1;
}
```

  {{< /code >}}

  {{< code >}}

  ```golang
func getNext(needle string)[]int{
	next := make([]int,len(needle)) //长度为len(needle)
	//根据前后缀来求Next数组
	maxMatch := 0 //最长匹配长度
	//i从1开始，因为0下标的next值一定为0
	for i:=1;i<len(needle);i++{
		for maxMatch > 0 && needle[i] != needle[maxMatch]{//当前不匹配，只能用前面的next值，直到匹配，或者maxMatch=0
			maxMatch = next[maxMatch-1] //回溯求最长匹配长度
		}

		if needle[i] == needle[maxMatch]{//匹配也只会在原来的基础上+1
			maxMatch++
		}

		next[i] = maxMatch//赋next值，
	}
	return next
}

func strStr(haystack string, needle string) int {
	if needle == ""{
		return 0
	}

	if len(haystack) < len(needle){
		return -1
	}
	//第一步先求next数组
	next := getNext(needle)
	//第二步开始匹配
	for i,j:=0,0;i<len(haystack);i++{   //
		for j > 0 && haystack[i] != needle[j]{
			j = next[j-1]
		}
		if haystack[i] == needle[j]{
			j++
		}

		if j == len(needle){//全部匹配完，返回
			return i-j+1
		}
	}


	return -1 //没找到，返回-1
}
  ```
  {{< /code >}}
{{< /codes >}}

## 总结

仔细观察KMP算法的代码后，你就会发现，它与带备忘录的动态规划解法惊人的相似，``我们用一个next数组(备忘录)来记录下一个字符匹配失败，如果再次匹配成功应该移动的步数(当前匹配字符数-next值)，然后下一次匹配直接从可以在移动步数之后进行匹配，从而跳过一些已经知道不可能再匹配成功的字符(好比递归树的剪枝)``