---
layout: post
author: mxn
titile: 深入理解listView的setSelectionFromTop
category: 技术博文
tag: android
---

有时候实现比较好的用户体验，需要保持当前的ListView的位置。换句话说，我们想要随心所欲地指定ListView滚动的位置，在ListView中，有一个setSelectionFromTop
()方法。

下面是一个使用范例。代码如下：

    {% highlight java  %}
    // save index and top position
    int index = mList.getFirstVisiblePosition();
    View v = mList.getChildAt(0);
    int top = (v == null) ? 0 : v.getTop();
    mList.setSelectionFromTop(index, top);
    {% endhighlight %}

注解：ListView.getChildAt(int position)， 这个position指的是在可视的item中的索引，跟cursor里的位置是大不一样的。
可以看看ListView.getChildCount()函数得到个数是小于或等于Cursor里的个数的（不考虑header的话）。
虽然一共可能有20条数据，但是界面只能看到8条，那么这个ChildCount大约就是8了。

看一下setSelectionFromTop()的具体实现，代码如下：

    {% highlight java  %}
    public void setSelectionFromTop(int position, int y) {
        if (mAdapter == null) {
            return;
        }

        if (!isInTouchMode()) {
            position = lookForSelectablePosition(position, true);
            if (position >= 0) {
                setNextSelectedPositionInt(position);
            }
        } else {
            mResurrectToPosition = position;
        }

        if (position >= 0) {
            mLayoutMode = LAYOUT_SPECIFIC;
            mSpecificTop = mListPadding.top + y;

            if (mNeedSync) {
                mSyncPosition = position;
                mSyncRowId = mAdapter.getItemId(position);
            }

            if (mPositionScroller != null) {
                mPositionScroller.stop();
            }
            requestLayout();
        }
    }
       {% endhighlight %}

从上面的代码可以得知，setSelectionFromTop()的作用是设置ListView选中的位置，同时在Y轴设置一个偏移量（padding值）。
ListView还有一个方法叫setSelection()，传入一个index整型数值，就可以让ListView定位到指定Item的位置。
这两个方法有什么区别呢？看一下setSelection()的具体实现，代码如下：

    {% highlight java  %}
    @Override
    public void setSelection(int position) {
     setSelectionFromTop(position, 0);
    }
    {% endhighlight %}

原来，setSelection()内部就是调用了setSelectionFromTop()，只不过是Y轴的偏移量是0而已。
现在应该对setSelection()和setSelectionFromTop()有了更深刻的认识了。


待续未完
