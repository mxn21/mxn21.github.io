---
layout: post
author: mxn
titile: android中的MotionEvent及其它事件处理
category: 技术博文
tag: android
---


MotionEvent即一个触摸事件。对屏幕的几乎所有操作都会触发事件，如点击、放开、滑动等。

不同的事件在MotionEvent中有不同的id，我们可以根据event.getAction() & MotionEvent.ACTION_MASK的结果来判断是何种事件。

有如下事件是我们要用到的：

MotionEvent.ACTION_DOWN：在第一个点被按下时触发。

MotionEvent.ACTION_UP:当屏幕上唯一的点被放开时触发。

MotionEvent.ACTION_POINTER_DOWN:当屏幕上已经有一个点被按住，此时再按下其他点时触发。

MotionEvent.ACTION_POINTER_UP:当屏幕上有多个点被按住，松开其中一个点时触发（即非最后一个点被放开时）。

MotionEvent.ACTION_MOVE：当有点在屏幕上移动时触发。值得注意的是，由于它的灵敏度很高，
而我们的手指又不可能完全静止（即使我们感觉不到移动，但其实我们的手指也在不停地抖动），
所以实际的情况是，基本上只要有点在屏幕上，此事件就会一直不停地被触发。

