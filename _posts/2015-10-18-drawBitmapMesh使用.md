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

2.meshWidth 　　控制在横向上把该源位图划成成多少格,为0时不绘制图像。

3.meshHeight 　　控制在纵向上把该源位图划成成多少格,为0时不绘制图像。

4.verts 　　　　　verts是个一维数组，保存所有顶点坐标信息。偶数项保存x坐标，奇数项保存y坐标。比如有有meshWidth*meshHeight个网格，
如果vertOffset为0，那么算上两端就有(meshWidth+1)*(meshHeight+1)个顶点，verts数组就应该至少长度为(meshWidth+1)*(meshHeight+1)。

5.vertOffset         控制verts数组中从第几个数组元素开始才对bitmap进行扭曲

6.Colors      可以为空，不为空为每个顶点定义对应的颜色值，至少需要有(meshWidth+1) * (meshHeight+1) * 2 + meshOffset 个(x,y)坐标。

7.colorOffset   colors数组中开始跳过的(x,y)对的数目。

8.paint   可以为空

<!-- more -->

Mesh代表的是网格，这个东西的原理就是按照网格来重新拉伸你的图像,假想在一张图片上有很多网格，如下图。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img121.jpg)

在这张图上，每一条横线和纵线有一个焦点，我们可以控制这些焦点的位置来重新改变图片的形状和画面，也就是网格怎么扭动，
图像就会怎么动。比如把焦点位置改变成下面这样，图像就跟着扭曲了。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img122.jpg)

在DrawBitmapMesh中，只需要定义好这个顶点将要扭曲到哪个坐标点上，然后将顶点扭曲后的坐标告诉DrawBitmapMesh，便会自动计算出周边的线条扭曲形式，并根据结果扭曲图像。

总结上面的，需要做的就是三步：
1、根据图片，生成原始的、四四方方的网格
2、根据上面生成的网格，算出将要扭曲的网格
3、将网格传入drawBitmapMesh

下面用前面的知识做一个自定义view，让view实现像旗帜飘动的效果,效果图如下：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img123.png)

view代码如下：

    {% highlight java %}
public class MeshView extends View{
	private int WIDTH = 200 ;
	private int HEIGHT = 200 ;
	private int count = (WIDTH + 1) * (HEIGHT + 1) ;
	private float[] verts = new float[count * 2] ;
	private float[] orig = new float[count * 2] ;
	private Bitmap mBitmap;
	private float K ;
	public MeshView(Context context){
			super(context);
			initView() ;
	}
	public MeshView(Context context,AttributeSet attrs){
		super(context,attrs);
		initView() ;
}
	private void initView(){
		int index  = 0  ;
		mBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.screenshot);
		float bmWidth = mBitmap.getWidth() ;
		float bmHeight = mBitmap.getHeight() ;
		for(int i = 0 ; i < HEIGHT + 1; i++){
			float fy = bmHeight * i / HEIGHT ;
			for(int j = 0 ; j < WIDTH + 1 ; j ++ ){
				float fx = bmWidth * j / WIDTH ;
				orig[index * 2 + 0 ] = verts[index * 2 + 0 ] = fx ;
				orig[index * 2 + 1 ] = verts[index * 2 + 1 ] = fy ;
				index += 1 ;
			}
		}
	}
	@Override
	protected void onDraw(Canvas canvas){
		for(int i = 0 ; i < HEIGHT + 1; i++){
			for(int j = 0 ; j < WIDTH + 1 ; j ++ ){
				verts[(i*(WIDTH+1) + j ) * 2 + 0 ] += 0 ;
				float offsetY  = (float) Math.sin( (float)j / WIDTH * 2 * Math.PI + K * 2 * Math.PI) ;
				verts[(i*(WIDTH+1) + j ) * 2 + 1 ]  = orig[(i*(WIDTH+1) + j ) * 2 + 1 ] +
						offsetY * 50 ;
			}
		}
		K += 0.1F ;
		canvas.drawBitmapMesh(mBitmap, WIDTH, HEIGHT, verts, 0, null, 0, null) ;
		invalidate() ;
	}
}
    {% endhighlight %}