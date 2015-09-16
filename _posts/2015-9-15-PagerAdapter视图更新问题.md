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

<!-- more -->