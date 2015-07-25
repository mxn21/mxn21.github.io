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








