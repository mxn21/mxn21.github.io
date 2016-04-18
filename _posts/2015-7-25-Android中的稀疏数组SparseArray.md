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

<!-- more -->

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

### get和遍历

如果索引不存在，indexOfKey(int key)，将会返回负数值。
遍历需要获取数组的总的对象大小，然后用keyAt(int index)获取索引或者valueAt(int index)获取值。

    {% highlight java  %}
for(int i = 0; i < sparseArray.size(); i++) {
       int key = sparseArray.keyAt(i);
       Object obj = sparseArray.get(key);
}
     {% endhighlight %}

如果不关系key，可以直接通过valueAt得到value，这样速度更快，写法更简单：

    {% highlight java  %}
for(int i = 0; i < sparseArray.size; i++) {
    Object obj = sparseArray.valueAt(i);
}
     {% endhighlight %}

### 测试

下面我们就通过几段程序来证明SparseArray在各方面表现如何

代码1：

    {% highlight java  %}
int MAX = 100000;
long start = System.currentTimeMillis();
HashMap<Integer, String> hash = new HashMap<Integer, String>();
for (int i = 0; i < MAX; i++) {
    hash.put(i, String.valueOf(i));
}
long ts = System.currentTimeMillis() - start;
        {% endhighlight %}

代码2：

    {% highlight java  %}
int MAX = 100000;
long start = System.currentTimeMillis();
SparseArray<String> sparse = new SparseArray<String>();
for (int i = 0; i < MAX; i++) {
    sparse.put(i, String.valueOf(i));
}
long ts = System.currentTimeMillis() - start;
        {% endhighlight %}

我们分别在long start处和long ts处设置断点，然后通过DDMS工具查看内存使用情况。
代码1中，我们使用HashMap来创建100000条数据，开始创建前的系统内存情况为：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img44.png)

创建HashMap之后，应用内存情况为：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img45.png)

可见创建HashMap用去约 13.2M内存。
再看 代码2，同样是创建100000条数据，我们用SparseArray来试试，开始创建前的内存使用情况为：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img46.png)

创建SparseArray之后的情况：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img47.png)


创建SparseArray共用去 8.626M内存。
可见使用 SparseArray 的确比 HashMap 节省内存，大概节省 35%左右的内存。


下面把插入顺序变换一下，从大到小插入：

代码3：

    {% highlight java  %}
int MAX = 100000;
long start = System.currentTimeMillis();
HashMap<Integer, String> hash = new HashMap<Integer, String>();
for (int i = 0; i < MAX; i++) {
    hash.put(MAX - i -1, String.valueOf(i));
}
long ts = System.currentTimeMillis() - start;
     {% endhighlight %}

代码4:

    {% highlight java  %}
int MAX = 100000;
long start = System.currentTimeMillis();
SparseArray<String> sparse = new SparseArray<String>();
for (int i = 0; i < MAX; i++) {
    sparse.put(MAX - i -1, String.valueOf(i));
}
long ts = System.currentTimeMillis() - start;
         {% endhighlight %}

我们分别把这4段代码分别运行5次，对比一下ts的时间（单位毫秒）：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img48.png)

通过结果我们看出，在正序插入数据时候，SparseArray比HashMap要快一些；
HashMap不管是倒序还是正序开销几乎是一样的；但是SparseArray的倒序插入要比正序插入要慢10倍以上，这时为什么呢？我们再看下面一段代码：

    {% highlight java  %}
    SparseArray<String> sparse = new SparseArray<String>(3);
    sparse.put(1, "s1");
    sparse.put(3, "s3");
    sparse.put(2, "s2");
         {% endhighlight %}

我们在Eclipse的debug模式中，看Variables窗口,如图：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img49.png)

及时我们是按照1,3,2的顺序排列的，但是在SparseArray内部还是按照正序排列的，这时因为SparseArray在检索数据的时候使用的是二分查找，
所以每次插入新数据的时候SparseArray都需要重新排序，所以代码4中，逆序是最差情况。

下面我们在简单看下检索情况：

代码5:

    {% highlight java  %}
long start4search = System.currentTimeMillis();
for (int i = 0; i < MAX; i++) {
    hash.get(33333); //针对固定值检索
}
long end4search = System.currentTimeMillis() - start4search;
     {% endhighlight %}

代码6:
     {% highlight java  %}
long start4search = System.currentTimeMillis();
for (int i = 0; i < MAX; i++) {
    hash.get(i); //顺序检索
}
long end4search = System.currentTimeMillis() - start4search;
      {% endhighlight %}

代码7:
     {% highlight java  %}
long start4search = System.currentTimeMillis();
for (int i = 0; i < MAX; i++) {
    sparse.get(33333); //针对固定值检索
}
long end4search = System.currentTimeMillis() - start4search;
      {% endhighlight %}

代码8:
       {% highlight java  %}
long start4search = System.currentTimeMillis();
for (int i = 0; i < MAX; i++) {
    sparse.get(i); //顺序检索
}
long end4search = System.currentTimeMillis() - start4search;
       {% endhighlight %}

表1：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img50.png)

代码9，我们试一些离散的数据。

   {% highlight java  %}
//使用Foo为了避免由原始类型被自动封装（auto-boxing，比如把int类型自动转存Integer对象类型）造成的干扰。
class FOO{
    Integer objKey;
    int intKey;
}
...
int MAX = 100000;
HashMap<Integer, String> hash = new HashMap<Integer, String>();
SparseArray<String> sparse = new SparseArray<String>();
for (int i = 0; i < MAX; i++) {
    hash.put(i, String.valueOf(i));
    sparse.put(i, String.valueOf(i));
}
List<FOO> keylist4search = new ArrayList<FOO>();
for (int i = 0; i < MAX; i++) {
    FOO f = new FOO();
    f.intKey = i;
    f.objKey = Integer.valueOf(i);
    keylist4search.add(f);
}
long start4search = System.currentTimeMillis();
for (int i = 0; i < MAX; i++) {
    hash.get(keylist4search.get(i).objKey);
}
long end4searchHash = System.currentTimeMillis() - start4search;
long start4search2 = System.currentTimeMillis();
for (int i = 0; i < MAX; i++) {
    sparse.get(keylist4search.get(i).intKey);
}
long end4searchSparse = System.currentTimeMillis() - start4search2;
   {% endhighlight %}

代码9,运行5次之后的结果如下：

表2:

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img51.png)

从上面两个表中我们可以看出，当SparseArray中存在需要检索的下标时，SparseArray的性能略胜一筹（表1）。
但是当检索的下标比较离散时，SparseArray需要使用多次二分检索，性能显然比hash检索方式要慢一些了（表2），
但是按照官方文档的说法性能差异不是很大，不超过50%（ For containers holding up to hundreds of items, the performance difference is not significant, less than 50%.）
总体而言，在Android这种内存比CPU更金贵的系统中，能经济地使用内存还是上策，何况SparseArray在其他方面的表现也不算差
（另外，SparseArray删除数据的时候也做了优化——使用了延迟整理数组的方法,二者的性能相差无几）。

### 总结

1.稀疏数组的使用，对于索引是整数的情景，有时能带来一些效率的提升。
减少了hashCode时间消耗，减小了所使用的内存大小。

2.在所管理的对象数量很大时，效率却反而有可能更低：在插入的时候，有可能导致大段数组的复制;在删除之后，也有可能导致数组的大段元素被按个移动（不是复制数组，而是一个一个单独移动）;
索引的映射，采用了二分查找，时间复杂度为O(logn)。