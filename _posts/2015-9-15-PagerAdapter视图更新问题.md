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


