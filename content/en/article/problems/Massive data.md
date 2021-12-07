---
date: 2021-11-23
description: "如何处理大数据，内存限制 的问题"
image: "images/recommend_site/xingyouji.jpg"
title: "海量数据"
author: 诸葛青
authorEmoji: 🎅
pinned: false
tags:
- 海量数据
series:
- 
---

### 有一个包含20亿个全是32位整数的大文件，在其中找到出现次数最多的数
{{< notice success >}}
分析：首先采用hash表用于统计，key为整数的值，value为整数出现的次数，而key为int32，占4字节，value也应为int32（最坏情况20亿个数都相同，只用int32正好cover，而又不浪费空间），占4字节，所以对于hash表一条记录需要8字节，最坏情况20亿个数都不相同，需要20亿条记录，大约就需要16GB的内存（简单估计大小，实际并没有16GB），而显然2GB内存空间无法加载一个16GB的hash表。

解决方法：把包含20亿个整数的大文件用hash函数拆分成16个小文件，根据hash函数的性质可知，相同的数是不会被分配到不同的文件的，只要hash函数足够好，每一个小文件的hash表的所占空间就会小于2GB。拆分之后，对于每一个小文件，都用hash表来统计每一种数出现的次数，就可以得出16个小文件各自出现次数最多的数，再将这16个数进行比较，即可得出出现次数最多的数
{{< /notice >}}


`进阶版`——>[点击此处](https://www.cnblogs.com/ExMan/p/10984358.html)

