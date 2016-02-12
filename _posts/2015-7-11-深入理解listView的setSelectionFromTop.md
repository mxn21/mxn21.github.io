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

注解：
ListView.setSelectionFromTop(int position, int y);其中position指的是指定的item的在ListView中的索引，
注意如果有Header存在的情况下，索引是从Header就开始算的。y指的是到ListView可见范围内最上边边缘的距离。

ListView.getChildAt(int position)， 这个position指的是在可视的item中的索引，跟cursor里的位置是大不一样的。
可以看看ListView.getChildCount()函数得到个数是小于或等于Cursor里的个数的（不考虑header的话）。
虽然一共可能有20条数据，但是界面只能看到8条，那么这个ChildCount大约就是8了。
另一方面， FirstVisiblePosition取出的是第一个可见的item在总的条数中的索引，再将会消失的header考虑进来

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

  <!-- more -->
  
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


另外，可以通过别的方式可以记录listView滚动到的位置的坐标，然后利用listView.scrollTo精确的进行恢复 。

    {% highlight java  %}
listView.setOnScrollListener(new OnScrollListener() {
    /**
     * 滚动状态改变时调用
     */
    @Override
    public void onScrollStateChanged(AbsListView view, int scrollState) {
        // 不滚动时保存当前滚动到的位置
        if (scrollState == OnScrollListener.SCROLL_STATE_IDLE) {
            if (currentMenuInfo != null) {
                scrolledX = statusListView.getScrollX();
                scrolledY = statusListView.getScrollY();
            }
        }
    }
    /**
     * 滚动时调用
     */
    @Override
    public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
    }
});
    {% endhighlight %}

 在恢复位置时调用
     {% highlight java  %}
 listView.scrollTo(scrolledX, scrolledY);
     {% endhighlight %}

