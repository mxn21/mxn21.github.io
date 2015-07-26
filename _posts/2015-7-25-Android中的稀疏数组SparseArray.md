---
layout: post
author: mxn
titile: Android中的稀疏数组SparseArray
category: 技术博文
tag: android
---

Sparse array —稀疏数组

所谓稀疏数组就是数组中大部分的内容值都未被使用（或都为零），
在数组中仅有少部分的空间使用。因此造成内存空间的浪费，为了节省内存空间，
并且不影响数组中原有的内容值，我们可以采用一种压缩的方式来表示稀疏数组的内容。

HashMap是java里比较常用的一个集合类，在一般情况下，使用HashMap<K, V>，如果K整数类型的话，使用SparseArray效率会更高。

### 稀疏数组

假设有一个9*7的数组，其内容如下：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img42.jpg)

在此数组中，共有63个空间，但却只使用了5个元素，造成58个元素空间的浪费。以下我们就使用稀疏数组重新来定义这个数组：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img43.jpg)

其中在稀疏数组中第一部分所记录的是原数组的列数和行数以及元素使用的个数、第二部分所记录的是原数组中元素的位置和内容。
经过压缩之后，原来需要声明大小为63的数组，而使用压缩后，只需要声明大小为6*3的数组，仅需18个存储空间。

<!-- more-->


