---
layout: post
author: mxn
titile: drawBitmapMesh使用
category: 技术博文
tag: android
---

Canvas提供了一个方法
drawBitmapMesh(Bitmap bitmap, int meshWidth, int meshHeight, float[] verts, int vertOffset, int[] colors,int colorffset,Paint paint)

这个方法可以对bitmap进行扭曲
参数说明如下:
1.bitmap　　　　　需要扭曲的源位图
2.meshWidth 　　控制在横向上把该源位图划成成多少格
3.meshHeight 　　控制在纵向上把该源位图划成成多少格
4.verts 　　　　　长度为(meshWidth + 1) * (meshHeight + 1) * 2的数组，它记录了扭曲后的位图各顶点位置
5.vertOffset         控制verts数组中从第几个数组元素开始才对bitmap进行扭曲


待续

