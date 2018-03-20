---
title: redis对象
date: 2017-11-21 23:03:42
tags: [redis,cache]
toc: true
---

前面的章节，介绍了Redis泳道的主要数据结构，比如简单动态字符串，双端列表，字典，压缩列表，整数集合等等。Redis并没有直接使用这些数据结构来实现键值对数据库，而是基于数据结构创建了一个对象系统，这个系统包含字符串对象，列表对象，哈希对象，集合对象和有序集合对象这五种类型的对象，每种对象都用到了至少一种前面说的数据结构。

# 对象的类型和编码

Redis使用对象来表示数据库中的键和值，每次当我们在Redis的数据库中新创建一个键值对时，我们至少会创建两个对象，一个对象用作键值对的键，另一个对象用作键值对的值。

```c
type struct redisObject {
    // 类型
  	unsigned type:4;
  	// 编码
  	unsigned encoding:4;
  	// 指向底层实现数据结构的指针
  	void *ptr;
} robj;
```

## 类型

| 类型常量         | 对象的名称  |
| ------------ | ------ |
| REDIS_STIRNG | 字符串对象  |
| REDIS_LIST   | 列表对象   |
| REDIS_HASH   | 哈希对象   |
| REDIS_SET    | 集合对象   |
| REDIS_ZSET   | 有序集合对象 |

## 编码和底层实现

对象的prt指针指向对象的底层实现数据结构，而这些数据结构由对象的encoding属性决定。encoding属性记录了对象所使用的编码，也即是说这个对象使用了什么数据结构作为对象的底层实现。

| 编码常量                      | 编码所对应的底层数据结构     |
| ------------------------- | ---------------- |
| REDIS_ENCODING_INT        | long类型的整数        |
| REDIS_ENCODING_EMBSTR     | embstr编码的简单动态字符串 |
| REDIS_ENCODING_RAW        | 简单动态字符串          |
| REDIS_ENCODING_HT         | 字典               |
| REDIS_ENCODING_LINKEDLIST | 双端链表             |
| DRDIS_ENCODING_ZIPLIST    | 压缩列表             |
| REDIS_ENCODING_INTSET     | 整数集合             |
| REDIS_ENCODING_SKIPLIST   | 跳跃表和字典           |

每种类型的对象都至少使用了两种不同的编码。

| 类型           | 编码                        | 对象                         |
| ------------ | ------------------------- | -------------------------- |
| REDIS_STRING | REDIS_ENCODING_INT        | 使用整数值实现的字符串对象              |
| REDIS_STRING | REDIS_ENCODING_EMBSTR     | 使用embstr编码的简单动态字符串实现的字符串对象 |
| REDIS_STRING | REDIS_ENCODING_RAW        | 使用简单动态字符串实现的字符串对象          |
| REDIS_LIST   | REDIS_ENDOGING_ZIPLIST    | 使用压缩列表实现的列表对象              |
| REDIS_LIST   | REDIS_ENDOGING_LINKEDLIST | 使用双端列表实现的列表对象              |
| REDIS_HASH   | REDIS_ENDOGING_ZIPLIST    | 使用压缩列表实现的哈希对象              |
| REDIS_HASH   | REDIS_ENDOGING_HT         | 使用字典实现的哈希对象                |
| REDIS_SET    | REDIS_ENDOGING_INTSET     | 使用整数集合实现的哈希对象              |
| REDIS_SET    | REDIS_ENDOGING_HT         | 使用字典集合实现的哈希对象              |
| REDIS_ZSET   | REDIS_ENDOGING_ZIPLIST    | 使用压缩列表实现的有序列表集合            |
| REDIS_ZSET   | REDIS_ENDOGING_SKIPLIST   | 使用跳跃表实现的有序集合               |
