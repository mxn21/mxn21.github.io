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

和HashMap类似，SparseArray建立整数索引和对象的关系。和简单的对象数组相比，SparseArray允许索引之间有间隔。
SparseArray支持和HashMap类似的put和get方法。在其内部，维护着两个数组，一个用于存储索引，一个用于存储对象。

    {% highlight java  %}
    public class SparseArray<E> implements Cloneable {

        private int[] mKeys;
        private Object[] mValues;
        private int mSize;
    {% endhighlight %}

整数索引被从小到大映射到mKeys数组中。

### 索引的映射

在计算整数索引映射到数组中的位置的时候，用了一个改造过的的二分搜索算法：

这个算法输入的参数是：要搜索的数组a，搜索的起始位置start, 搜索的长度len, 要检索的关键字key，如下:

    {% highlight java  %}
private static int binarySearch(int[] a, int start, int len, int key) {
    int high = start + len, low = start - 1, guess;

    while (high - low > 1) {
        guess = (high + low) / 2;

        if (a[guess] < key)
            low = guess;
        else
            high = guess;
    }

    if (high == start + len)
        return ~(start + len);
    else if (a[high] == key)
        return high;
    else
        return ~high;
}
    {% endhighlight %}

如果输入的key在区间内则返回等于关键字或者最小的大于关键字的索引。
如果关键字不在区间内，则将区间首个索引或者区间最后一个索引加1取反码，非负数的反码都是负数，因为符号位被取反了。


### put过程

put的过程分为以下几步：

1.计算索引映射。

2.如果在在区间内有对应槽位，设置值，返回。

3.如有必要，进行扩容。

容量以类似2的指数次幂增长。对象引用和和整数都占用4个字节，数组本身还需要占用3个字节。 为了内存4字节对齐，数组大小应该是：2^n - 3(n >=2)。

4.如有必要，移动区段

如果计算出的映射索引，在现有对象的位置上，需要移动区段。

5.最后，设置值，将数据长度加1。

主要代码如下，省略了部分细节：

    {% highlight java  %}
public void put(int key, E value) {

    // 1. 计算索引
    int i = binarySearch(mKeys, 0, mSize, key);

    // 2. key已经有对应槽位，更新值
    if (i >= 0) {
        mValues[i] = value;
    } else {
        i = ~i;

        // 3. 扩容
        if (mSize >= mKeys.length) {
        }

        // 4. 移动区段
        if (mSize - i != 0) {
            // Log.e("SparseArray", "move " + (mSize - i));
            System.arraycopy(mKeys, i, mKeys, i + 1, mSize - i);
            System.arraycopy(mValues, i, mValues, i + 1, mSize - i);
        }

        // 4. 设置值，长度加1
        mKeys[i] = key;
        mValues[i] = value;
        mSize++;
    }
}
    {% endhighlight %}