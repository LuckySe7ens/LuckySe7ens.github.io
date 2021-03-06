---
layout: post
title: redis源码设计与实现
date: 2019-2-14
categories: blog
tags: [redis]
description: 文章金句。
---
# 动态字符串 SDS(simple dynamic String)
### function1：保存redis的键值对中的 key 和字符串类型的 value
### function2：被用作缓冲区（buffer） AOF模块中的AOF缓冲区、客户端状态中的输入缓冲区

### SDS数据结构
```javascript
struct sdshdr {    
  // 记录buf数组中已使用字节的数量    
  // 等于SDS所保存字符串的长度    
  int len;    

  // 记录buf数组中未使用字节的数量    
  int free;    

  // 字节数组，用于保存字符串    
  char buf[]; 
}; 
```

|sdshdr|
|:-----:|
|free <br>0 |
|len  <br>5  |
|buffer |      

   &darr;

|'R'|'e'|'d'|'i'|'s'|'\0'|
|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|
### SDS优势1：获取字符串长度时间复杂度O(1)
### SDS优势2：杜绝缓冲区溢出
### 3减少修改字符串时带来的内存重分配次数
### 4空间预分配
空间预分配用于优化 SDS 的字符串增长操作：当 SDS 的 API 对一个 SDS 进行修改， 并且需要对 SDS 进行空间扩展的时候，程序不仅会为 SDS 分配修改所必须要的空间，还会 为 SDS 分配额外的未使用空间。 其中，额外分配的未使用空间数量由以下公式决定： 
* 如果对 SDS 进行修改之后，SDS 的长度（也即是 len 属性的值）将小于 1MB， 那么程序分配和 len 属性同样大小的未使用空间，这时 SDS len 属性的值将和 free 属性的值相同。举个例子，如果进行修改之后，SDS 的 len 将变成 13 字 节，那么程序也会分配13字节的未使用空间，SDS 的buf数组的实际长度将变成 13+13+1=27字节（额外的一字节用于保存空字符）。 
* 如果对 SDS 进行修改之后，SDS 的长度将大于等于1MB，那么程序会分配1MB的未 使用空间。举个例子，如果进行修改之后，SDS 的len将变成30MB，那么程序会分 配1MB的未使用空间，SDS 的buf数组的实 际长度将为30 MB + 1MB + 1byte。 通过空间预分配策略，Redis 可以减少连续执行字 符串增长操作所需的内存重分配次数。 
### 惰性空间释放
