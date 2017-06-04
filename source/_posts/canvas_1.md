---
title: canvas 基本绘制
date: 2017-05-22 23:26:00
categories: android
tags: [自定义view,canvas]
---

## drawText
``` bash
/** 
* text:要绘制的文字 
* x：绘制原点x坐标 
* y：绘制原点y坐标 
* paint:用来做画的画笔 
*/  
public void drawText(String text, float x, float y, Paint paint)
```
<!-- more -->

值得注意的是在drawText中是非常例外的，y所代表的是基线的位置！
![](http://img2.ph.126.net/FG_IL21IqoAowpXxQNGOeg==/6632257437280879751.jpg)
还有就是x的位置会被Paint影响,x是相对位置
 paint.setTextAlign(Paint.Align.LEFT);// Panit.Align.LEFT,Paint.Align.CENTER,Paint.Align.RIGHT
 比如设置CENTER
![](http://img2.ph.126.net/TdkmsfzuPc_7AhALo4364g==/6632509225443640256.png)

``` bash
//计算各线在位置  
Paint.FontMetrics fm = paint.getFontMetrics();  
float ascent = baseLineY + fm.ascent;  
float descent = baseLineY + fm.descent;  
float top = baseLineY + fm.top;  
float bottom = baseLineY + fm.bottom; 
```

## drawRoundRect
``` bash
// 第一种
RectF rectF = new RectF(100,100,800,400);
canvas.drawRoundRect(rectF,30,30,mPaint);
// 第二种
canvas.drawRoundRect(100,100,800,400,30,30,mPaint);

多出来了两个参数rx 和 ry，这里圆角矩形的角实际上不是一个正圆的圆弧，而是椭圆的圆弧，这里的两个参数实际上是椭圆的两个半径，他们看起来个如下图
```
![](http://img0.ph.126.net/bY-vUOCI2jSs4okFYMLkVw==/6632270631420417891.png)

## drawArc 绘制圆弧
``` bash
// 第一种
public void drawArc(@NonNull RectF oval, float startAngle, float sweepAngle, boolean useCenter, @NonNull Paint paint){}
    
// 第二种
public void drawArc(float left, float top, float right, float bottom, float startAngle,
            float sweepAngle, boolean useCenter, @NonNull Paint paint) {}

// 开始角度 startAngle
// 扫过角度 sweepAngle
// 是否使用中心 useCenter
```
用法：先确定一个Rect，起始角度是Rect中心水平向右，顺时针画，例子如下
``` bash
RectF rectF = new RectF(100,100,800,400);
// 绘制背景矩形
mPaint.setColor(Color.GRAY);
canvas.drawRect(rectF,mPaint);

// 绘制圆弧
mPaint.setColor(Color.BLUE);
canvas.drawArc(rectF,0,90,false,mPaint);

//-------------------------------------

RectF rectF2 = new RectF(100,600,800,900);
// 绘制背景矩形
mPaint.setColor(Color.GRAY);
canvas.drawRect(rectF2,mPaint);

// 绘制圆弧
mPaint.setColor(Color.BLUE);
canvas.drawArc(rectF2,0,90,true,mPaint);
```
![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f8f0ijg8pvj308c0ett8m.jpg)




