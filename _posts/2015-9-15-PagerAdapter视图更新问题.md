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

### instantiateItem()

* 除非碰到 FragmentManager 刚好从 SavedState 中恢复了对应的 Fragment 的情况外，该函数将会调用 getItem() 函数，
生成新的 Fragment 对象。新的对象将被 FragmentTransaction.add()。
* FragmentStatePagerAdapter 就是通过这种方式，每次都创建一个新的 Fragment，而在不用后就立刻释放其资源，来达到节省内存占用的目的的。

### destroyItem()

* 将 Fragment 移除，即调用 FragmentTransaction.remove()，并释放其资源。


## 解决方案

对于FragmentPagerAdapter，复写pageradapter的getItemPosition方法设置tag为POSITION_NONE意思是没有找到child要求重新加载。
结果你发现是刷新了一下但内容还是原始的数据。通过对fragmentpageadapter的源码查看你会在instantiateItem方法里面发现这一段：

    {% highlight java  %}
 String name = makeFragmentName(container.getId(), position);
        Fragment fragment = mFragmentManager.findFragmentByTag(name);
        if (fragment != null) {
            if (DEBUG) Log.v(TAG, "Attaching item #" + position + ": f=" + fragment);
            mCurTransaction.attach(fragment);
        } else {
            fragment = getItem(position);
            if (DEBUG) Log.v(TAG, "Adding item #" + position + ": f=" + fragment);
            mCurTransaction.add(container.getId(), fragment,
                    makeFragmentName(container.getId(), position));
        }
    {% endhighlight %}

它会先去FragmentManager里面去查找有没有相关的fragment如果有就直接使用如果没有才会触发fragmentpageadapter的getItem
方法获取一个fragment。
假如现在有了三个城市A、B、C，position分别为0、1、2.然后添加了一个城市Z，Z的position为0，A为1，B为2，C为3。FragmentManager里已经存在了position
为0,1,2的fragment，所以前三个视图没有改变，但现在需要的fragment的数目变了，增加1。会创建position为3的fragment，而此时是城市C，所以这样就会导致显示两次C，而新增的Z城市没有显示。

所以你更新的fragmentList集合是没有作用的，还要清除FragmentManager里面缓存的fragment，或者重写instantiateItem。

针对 FragmentPagerAdapter 的解决办法如下列代码所示：

    {% highlight java  %}
@Override
public Fragment getItem(int position) {
    MyFragment f = new MyFragment();
    return f;
}
@Override
public Object instantiateItem(ViewGroup container, int position) {
    MyFragment f = (MyFragment) super.instantiateItem(container, position);
    String title = mList.get(position);
    f.setTitle(title);
    return f;
}
@Override
public int getItemPosition(Object object) {
    return PagerAdapter.POSITION_NONE;
}
    {% endhighlight %}


或者：
    {% highlight java  %}
@Override
public Fragment getItem(int position) {
    MyFragment f = new MyFragment();
    return f;
}
@Override
public int getItemPosition(Object object) {
    return PagerAdapter.POSITION_NONE;
}
private void removeALlFragments(){
    FragmentTransaction transaction = fm.beginTransaction();
    for (int i=0; i<fragments.size(); i++){
        Fragment fg = fragments.get(i);
        transaction.remove(fg);
    }
    transaction.commit();
    fragments.clear();
}
    {% endhighlight %}

总结：
对于 FragmentPagerAdapter 的解决方案就是，分别重载 getItem() 以及 instantiateItem() 对象。getItem() 只用于生成新的与数据无关的 Fragment；
而 instantiateItem() 函数则先调用父类中的 instantiateItem() 取得所对应的 Fragment 对象，然后，根据对应的数据，调用该对象对应的方法进行数据设置。
当然，不要忘记重载 getItemPosition() 函数，返回 POSITION_NONE。

这样，当 notifyDataSetChanged() 被调用后，会最终触发 instantiateItem()，而不管 getItem() 是否被调用，
我们都在重载的 instantiateItem() 函数中已经将所需要的数据传递给了相应的 Fragment。在 Fragment 接下来的 onCreateView(),
onStart() 以及 onResume() 的事件中，它可以正确的读取新的数据，Fragment 被成功复用了。

这里需要注意一个问题，在 Fragment 没有被添加到 FragmentManager 之前，我们可以通过 Fragment.setArguments() 来设置参数，
并在 Fragment 中，使用 getArguments() 来取得参数。这是常用的参数传递方式。但是这种方式对于我们说的情况不适用。
因为这种数据传递方式只可能用一次，在 Fragment 被添加到 FragmentManager 后，一旦被使用，我们再次调用 setArguments()
将会导致 java.lang.IllegalStateException: Fragment already active 异常。因此，我们这里的参数传递方式选择是，
在继承的 Fragment 子类中，新增几个 setter，然后通过这些 setter 将数据传递过去。反向也是类似，
这些 setter 中要注意不要操作那些 View，这些 View 只有在 onCreateView() 事件后才可以操作。

对于 FragmentStatePagerAdapter，同样需要重载 getItemPosition() 函数，返回 POSITION_NONE。二者不同之处在于，
FragmentStatePagerAdapter 在会在因 POSITION_NONE 触发调用的 destroyItem() 中真正的释放资源，
重新建立一个新的 Fragment；而 FragmentPagerAdapter 仅仅会在 destroyItem() 中 detach 这个 Fragment，
在 instantiateItem() 时会使用旧的 Fragment，并触发 attach，因此没有释放资源及重建的过程。

