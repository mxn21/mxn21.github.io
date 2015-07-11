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

