---
date: 2022-02-25                    
description: "Go map底层刨析"
image: "/images/Go.jpg"
title: "Go map"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- Go
series:
- 
---

## Go map基础
> Go map 又称为 hash表，Go语言 map采用的是开放寻址法里面的线性探测，线性探测策略是顺序（每次探测间隔为1）的

### map 的声明与初始化

```Go
func defineMap() {
	// 第一种声明方式
	var hash map[int]int
	hash[111] = 222 // panic，hash没有被初始化，map值为nil，

	// 第二种声明方式（make的第二个参数代表初建map的长度，如果不填的话，默认长度为 0）
	hash = make(map[int]int, 1)
	hash[111] = 222 // success
}
```

### map 的访问
> 当一个键不存在于map中时，访问的结果将会得到value类型的 零值
```Go
func accessMap() {
	hash := make(map[int]int)
	// 第一种访问方式
	v := hash[0]
	fmt.Println(v)
	// 第二种访问方式
	v, ok := hash[0]
	fmt.Println(v, ok)
}
```

### map的赋值
```Go
func assignMap() {
	hash := make(map[int]int)
	// map赋值 将 key 和 value绑定在一起
	hash[2002] = 04
	// 可以 使用delete对 map相同的键 进行多次删除操作（不会出现报错的情况）
	for i := 0; i < 100; i++ {
		delete(hash, 2002) // 前一个参数为需要进行删除操作的map，第二个参数为指定需要删除的key
	}
	fmt.Println(hash[2002])
}
```

### map key的比较
> 对于key，切片、函数、map是不可以比较的，也就是说这三种类型都不能当做map的key类型
```Go
func compareMapKey() {
	// hash := make(map[[]int]int) // error
	// hash := make(map[map[int]int]int) // error
	//hash := make(map[func()int]int) error
}
```

### map 的并发冲突
> Go语言的官方文档中写道：要求所有map操作都互斥将减慢大多数程序的速度，而只会增加少数程序的安全性“即Go语言只支持并发读取的原因是保证大多数场景下的查找效率
```Go
func concurrentMap() {
	hash := make(map[int]int)
	// 1. map并不支持并发读写（fatal error: concurrent map read and map write）
	/*
	   	// 写操作
	   	go func() {
	   		for {
	   			hash[1] = 2
	   		}
	   	}()
	   	// 读操作
	   	go func() {
	   		for {
	   			_ = hash[1]
	   		}
	   	}()
	   	for i:=0; i<100;i++{}

	   }
	*/

	// 2. map支持并发读取（success）
	// 写操作
	go func() {
		for {
			_, _ = hash[1]
		}
	}()
	// 写操作
	go func() {
		for {
			_ = hash[1]
		}
	}()
	for i := 0; i < 100; i++ {}
}
```

## Go map底层

### Go map底层结构
> 源码位置：go SDK\src\runtime\map.go
```Go
type hmap struct {
	// count 表示map中元素的个数
	count 	  int
	// flags 表示当前map的状态（是否处于读写状态等）
	flags     uint8
	// 2的B次幂表示当前map中桶的数量，2^B=Buckets size
	B         uint8
	// noverflow为map中溢出桶的数量。当溢出的桶太多时，map会进行same-size map growth，其实质是避免溢出桶过大导致内存泄露。
	noverflow uint16
	// hash的种子，它能为哈希函数的结果引入随机性，这个值在创建哈希表时确定，并在调用哈希函数时作为参数传入
	hash0     uint32
	// buckets是指向当前map对应的桶的指针
	buckets    unsafe.Pointer
	// oldbuckets是在map扩容时存储旧桶的，当所有旧桶中的数据都已经转移到了新桶中时，则清空
	oldbuckets unsafe.Pointer
	// nevacuate在扩容时使用，用于标记当前旧桶中小于nevacuate的数据都已经转移到了新桶中
	nevacuate  uintptr
	// extra存储map中的溢出桶
	extra *mapextra
}

// 桶结构
type bmap struct {
	tophash [bucketCnt]uint8
	key     [bucketCnt]T // key 数组
	value   [bucketCnt]T // value 数组
}
```



## 问答

1. Golang Map 底层实现

`Go 语言map底层实现是一个hash表，其解决hash碰撞使用的是开放寻址法中的线性探测策略(探测间隔为1)。主要由两个结构体实现，一个是hmap（a header for a go map），一个是bmap（a bucket for a go map）也就是bucket。其中hmap的主要字段有count（int，map中的元素个数），B（uint8，2的B次幂就是map中桶的数量），flags（uint8,表明map的一些状态，表示是否在进行扩容，是否在进行读写操作），buckets（unsafe.Pointer，当前map对应的桶的指针），oldbuckets（unsafe.Pointer，在map进行扩容时，指向旧桶），extra（map中的溢出桶的指针）。对于bmap主要字段有tophash（一个固定长度为8的uint8数组，存储key对应的hash值得高8位），key（数组），value（数组），overflow（指向溢出桶的指针）；其中除了tophash，其他字段都由程序在编译时重建bmap结构体产生，并确定key，value，桶的大小。`

2. Golang Map如何扩容

`关于map 扩容的源码函数为runtime.mapassign函数`
`有两种情况发生时触发哈希的扩容：`
* `装载因子已经超过 6.5（负载因子=哈希表中的元素数量/桶的数量）`
> 双倍扩容（也就是渐进式扩容），原有的桶不会进行一次性搬迁，每次最多搬迁两个，一个旧桶的数据会分流到两个新桶上，如果数据的hash&bucketMask小于或等于旧桶的大小，则此数据必须转移到和旧桶位置完全对应的新桶中去，理由是当前key所在新桶的序号与旧桶是完全相同的。


* `溢出桶数量过多；（由于 Go 语言哈希的扩容不是一个原子的过程，所以 runtime.mapassign 函数还需要判断当前哈希是否已经处于扩容状态，避免二次扩容造成混乱）`
> 等量扩容，重新排列，进行简单的直接转移即可。

