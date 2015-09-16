---
layout: post
author: mxn
titile: PagerAdapter视图更新问题
category: 技术博文
tag: android
---

android3.0以后，可以在项目中使用ViewPager+Fragment实现tab分页+子tab页内容滑动效果。当你更新
fragment List集合后调用fragmentpageadpater的notifyDataSetChanged方法时发现数据根本就没有刷新。
要搞明白这个问题，需要阅读相关几个类的相关方法的实现。

## PagerAdapter

这个类是FragmentPagerAdapter 以及 FragmentStatePagerAdapter 的基类。如果继承自该类，至少需要实现 instantiateItem(),
destroyItem(), getCount() 以及 isViewFromObject()。

### getItemPosition()

* 该函数用以返回给定对象的位置，给定对象是由 instantiateItem() 的返回值。
* 在 ViewPager.dataSetChanged() 中将对该函数的返回值进行判断，以决定是否最终触发 PagerAdapter.instantiateItem() 函数。
* 在 PagerAdapter 中的实现是直接传回 POSITION_UNCHANGED。如果该函数不被重载，则会一直返回 POSITION_UNCHANGED，从而导致 ViewPager
.dataSetChanged() 被调用时，认为不必触发 PagerAdapter.instantiateItem()。很多人因为没有重载该函数，
而导致调用PagerAdapter.notifyDataSetChanged() 后，什么都没有发生。

### instantiateItem()

在每次 ViewPager 需要一个用以显示的 Object 的时候，该函数都会被 ViewPager.addNewItem() 调用。

<!-- more -->

## FragmentPagerAdapter

FragmentPagerAdapter 继承自 PagerAdapter。相比通用的 PagerAdapter，该类更专注于每一页均为 Fragment 的情况。如文档所述，
该类内的每一个生成的 Fragment 都将保存在内存之中，因此适用于那些相对静态的页，数量也比较少的那种；如果需要处理有很多页，
并且数据动态性较大、占用内存较多的情况，应该使用FragmentStatePagerAdapter。FragmentPagerAdapter 重载实现了几个必须的函数，
因此来自 PagerAdapter 的函数，我们只需要实现 getCount()，即可。且，由于 FragmentPagerAdapter.instantiateItem() 的实现中，
调用了一个新增的虚函数 getItem()，因此，我们还至少需要实现一个 getItem()。因此，总体上来说，相对于继承自 PagerAdapter，更方便一些。

### getItem()

* 该类中新增的一个虚函数。函数的目的为生成新的 Fragment 对象。重载该函数时需要注意这一点。在需要时，该函数将被 instantiateItem() 所调用。
* 如果需要向 Fragment 对象传递相对静态的数据时，我们一般通过 Fragment.setArguments() 来进行，这部分代码应当放到 getItem()。
它们只会在新生成 Fragment 对象时执行一遍。
* 如果需要在生成 Fragment 对象后，将数据集里面一些动态的数据传递给该 Fragment，那么，这部分代码不适合放到 getItem() 中。
因为当数据集发生变化时，往往对应的 Fragment 已经生成，如果传递数据部分代码放到了 getItem() 中，这部分代码将不会被调用。
这也是为什么很多人发现调用 PagerAdapter.notifyDataSetChanged() 后，getItem() 没有被调用的一个原因。

### instantiateItem()

* 函数中判断一下要生成的 Fragment 是否已经生成过了，如果生成过了，就使用旧的，旧的将被 Fragment.attach()；如果没有，
就调用 getItem() 生成一个新的，新的对象将被 FragmentTransation.add()。
* FragmentPagerAdapter 会将所有生成的 Fragment 对象通过 FragmentManager 保存起来备用，以后需要该 Fragment 时，
都会从 FragmentManager 读取，而不会再次调用 getItem() 方法。
* 如果需要在生成 Fragment 对象后，将数据集中的一些数据传递给该 Fragment，这部分代码应该放到这个函数的重载里。在我们继承的子类中，
重载该函数，并调用 FragmentPagerAdapter.instantiateItem() 取得该函数返回 Fragment 对象，然后，我们该 Fragment 对象中对应的方法，
将数据传递过去，然后返回该对象。
* 否则，如果将这部分传递数据的代码放到 getItem()中，在 PagerAdapter.notifyDataSetChanged() 后，这部分数据设置代码将不会被调用.

### destroyItem()

* 该函数被调用后，会对 Fragment 进行 FragmentTransaction.detach()。这里不是 remove()，只是 detach()，
因此 Fragment 还在 FragmentManager 管理中，Fragment 所占用的资源不会被释放。


## FragmentStatePagerAdapter

FragmentStatePagerAdapter 和前面的 FragmentPagerAdapter 一样，是继承子 PagerAdapter。但是，和 FragmentPagerAdapter 不一样的是，
正如其类名中的 'State' 所表明的含义一样，该 PagerAdapter 的实现将只保留当前页面，当页面离开视线后，就会被消除，释放其资源；
而在页面需要显示时，生成新的页面。这么实现的好处就是当拥有大量的页面时，不必在内存中占用大量的内存。

### getItem()

* 一个该类中新增的虚函数。
* 函数的目的为生成新的 Fragment 对象。
* Fragment.setArguments() 这种只会在新建 Fragment 时执行一次的参数传递代码，可以放在这里。
* 由于 FragmentStatePagerAdapter.instantiateItem() 在大多数情况下，都将调用 getItem() 来生成新的对象，
因此如果在该函数中放置与数据集相关的 setter 代码，基本上都可以在 instantiateItem() 被调用时执行，但这和设计意图不符。
毕竟还有部分可能是不会调用 getItem() 的。因此这部分代码应该放到 instantiateItem() 中。


