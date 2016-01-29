---
layout: post
author: mxn
titile: TextView超链接实现方式总结
category: 技术博文
tag: android
---

TextView中的超链接可以通过几种方式实现：1.Html.fromHtml，2.Linkify.addLinks，3.Spannable。下面分别进行测试，包括
修改字体样式，下划线样式，点击事件等。


### Html.fromHtml方式

TextView，本身就支持部分的Html格式标签。这其中包括常用的字体大小颜色设置，文本链接等。使用起来也比较方便，只需要使用Html类转换一下即可：
textView.setText(Html.fromHtml(str));


