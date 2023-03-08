---
date: 2022-09-18
description: "Elasticsearch 分词 学习"
image: "images/es.png"
title: "分词 学习"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- elasticsearch
series:
- 
---

## 简单了解
* Analysis（文本分析）是将 全文本 转换程一系列单词的过程，也叫做分词
* Analysis（分词）需要通过 Analyzer（分词器）来实现
    * 可以使用Elasticsearch内置的分词器，也可以定制分词器
* 分词器主要由三部分组成
    * Character Filters 针对原始文本进行处理，比如去除HTML
    * Tokenize 按照一定规则 切分为单词
    * Token Filter 将切分的单词进行加工，小写，删除 stopwords，增加同义词等等
* ES内置的分词器
    * Standard Analyzer：默认的分词器，按词切分，小写处理
    * Simple Analyzer：按照非字母切分，非字母都会被去除，小写处理
    * WhiteSpace Analyzer：按照空格切分
    * Keyword Analyzer：不进行分词
    * pattern Analyzer：按照正则表达式进行分词

## 中文分词
> 选型，使用Ik分词器，地址：https://github.com/medcl/elasticsearch-analysis-ik

### IK 中 analyzer ik_max_word 和 ik_smart 什么区别

* ik_max_word: 会将文本做最细粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国,中华人民,中华,华人,人民共和国,人民,人,民,共和国,共和,和,国国,国歌”，会穷尽各种可能的组合，适合 Term Query

* ik_smart: 会做最粗粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国,国歌”，适合 Phrase Query

### 创建自己的IK词库

* 进入到/usr/share/elasticsearch/plugins/ik/config目录下（安装的IK目录下的config）

* 创建一个放自定义字典的目录`mkdir my_dict`

* 创建一个文件存放自定义的word，`vi word.dic`，写入一些word，比如：商品的全名，动漫名，总之是一些你能识别，但分词器难以识别的词
* 创建一个文件存放自定义的stop_word，`vi stop_word.dic`，写入一些stop_word，比如：的，地，是，呦
* `vi /usr/share/elasticsearch/plugins/ik/config/IKAnalyzer.cfg.xml` 将自定义词典路径添加进去
* 重启容器，测试，分词是否包含词库定义的词



## tips

### docker环境下的elasticsearch容器中文出现乱码
* 添加中文环境
```shell
yum install kde-l10n-Chinese -y
yum install glibc-common -y
localedef -c -f UTF-8 -i zh_CN zh_CN.utf8
echo "export LC_ALL=zh_CN.utf8" >> /etc/profile
source /etc/profile
```