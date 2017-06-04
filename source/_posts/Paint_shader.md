---
title: Paint shader
date: 2017-05-22 21:49:00
categories: android
tags: [自定义view,paint]
---

## BitmapShader
``` bash
// 3种模式
Shader.TileMode.CLAMP：当图片小于绘制尺寸时要进行边界拉伸来填充
Shader.TileMode.REPEAT：当图片小于绘制尺寸时重复平铺
Shader.TileMode.MIRROR：当图片小于绘制尺寸时镜像平铺
```
<!-- more -->

直接上例子,画一个简单的圆形bitmap
``` bash
private void init() {
    mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
    final Bitmap mBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.test);
    final BitmapShader shader = new BitmapShader(mBitmap, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP); // 分x轴，Y轴的重复模式
	//  shader.setLocalMatrix(Matrix); // 可以用Matrix来缩放bitmap
    mPaint.setShader(shader);
}

@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    float x = getWidth() / 2;
    float y = getHeight() / 2;
    float radius = Math.min(getWidth(), getHeight()) / 2;
    canvas.drawCircle(x, y, radius,mPaint); 
}
```
![](http://upload-images.jianshu.io/upload_images/2086682-417b1887d27cd820.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

实际上就是把bitmap绑定在paint上，通过canvas画出你想要的部分

##  LinearGradient 线性渐变
``` bash
/*
x0 绘制x轴起始点
y0 绘制y轴起始点
x1 绘制x轴结束点
y1 绘制y轴结束点
color0 起始颜色
color1 结束颜色
tile 模式
*/
LinearGradient shader = new LinearGradient(0, 0, 600, 600, Color.RED, Color.YELLOW, Shader.TileMode.REPEAT);
mPaint.setShader(shader);


/*
colors 颜色int值数组
postions 数组中的值有效范围是0f~1f，渐变结束所在区域的比例，1f的结束位置，与x1,y1有关
*/
final int [] colors =  new int [] {Color.MAGENTA,Color.CYAN,Color.RED};
final float [] positions = new float[]{0f,0.50f,1f};
final LinearGradient shader = new LinearGradient(0, 0, 600, 600,colors ,positions ,Shader.TileMode.REPEAT);
```

## RadialGradient 光束渐变
``` bash
/*
centerX 渐变中心点的X轴坐标
centerY 渐变中心点的Y轴坐标
radius 渐变区域的半径
*/
final RadialGradient shader = new RadialGradient(300f,300f,300,Color.RED,Color.YELLOW, Shader.TileMode.CLAMP);

final int[] colors = new int[]{Color.YELLOW,Color.BLUE ,Color.RED};
final float[] positions = new float[]{0f, 0.5f ,1f};
final RadialGradient shader = new RadialGradient(300f,300f,300,colors,positions, Shader.TileMode.CLAMP);
// colors中的元素个数要和positions中的元素个数相等
```

## SweepGradient 梯度渐变
``` bash
SweepGradient(float cx, float cy, int color0, int color1) 
SweepGradient(float cx, float cy, int colors[], float positions[])
//cx,cy是旋转点的x,y轴坐标，渐变过程总是顺时针方向旋转
```

## 6. ComposeShader 混合渐变
``` bash
// 需要关闭硬件加速
ComposeShader(Shader shaderA, Shader shaderB, PorterDuff.Mode mode)
ComposeShader(Shader shaderA, Shader shaderB, Xfermode mode)

构造方法中，前两个参数相同，都是需要一个着色器，差别在于第三个参数。第一个构造方法需要一个PorterDuff.Mode，而第二个构造构造方法需要PorterDuffXfermode
前面的三种的渐变，都是一种单一的渐变，ComposeShader可以把前面两种渐变混合进一种渐变效果
```