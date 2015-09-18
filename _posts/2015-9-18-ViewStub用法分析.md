---
layout: post
author: mxn
titile: ViewStub用法分析
category: 技术博文
tag: android
---

ViewStub组件和<include>标签的作用类似，主要是为了提高布局的重用性，及布局的模块化。它们之间最大的差别是，
ViewStub中的布局不会随着它所在布局的渲染而渲染，而<include>标签中的布局会随着它所在布局的渲染而渲染，
ViewStub中的布局只有在你需要的时候才会渲染到主界面中。

### 使用方法

1.在ButtonOne与ButtonTwo之间存在一个ViewStub布局，如下图：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img103.jpg)

2.单击ButtonOne后渲染ViewStub中的布局，如下图：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img104.jpg)

3.实现代码：