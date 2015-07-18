---
layout: post
author: mxn
titile: android requestLayout的流程分析
category: 技术博文
tag: android
---

当一个View调用requestLayout的时候，会给当前的View设置一个FORCE_LAYOUT标记。
由此向ViewParent请求布局。这样从这个View开始向上一直requestLayout。最终到达ViewRootImpl。ViewParent 就是当前的传输链。