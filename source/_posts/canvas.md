---
title: canvas 启动
date: 2017-07-22 10:10:10
categories: android
tags: [自定义view,canvas]
---

## 基本api
|操作类型	 		|相关API																							|备注
| -----------   	| -----:   																							| :----: |
|绘制颜色	 		|drawColor, drawRGB, drawARGB	|使用单一颜色填充整个画布
|绘制基本形状		|drawPoint, drawPoints, drawLine, drawLines, drawRect, drawRoundRect, drawOval, drawCircle, drawArc	|依次为 点、线、矩形、圆角矩形、椭圆、圆、圆弧
|绘制图片			|drawBitmap, drawPicture																			|绘制位图和图片
|绘制文本			|drawText, drawPosText, drawTextOnPath																|依次为 绘制文字、绘制文字时指定每个文字位置、根据路径绘制文字
|绘制路径			|drawPath																							|绘制路径，绘制贝塞尔曲线时也需要用到该函数
|顶点操作			|drawVertices, drawBitmapMesh																		|通过对顶点操作可以使图像形变，drawVertices直接对画布作用、 drawBitmapMesh只对绘制的Bitmap作用
|画布剪裁			|clipPath, clipRect																					|设置画布的显示区域
|画布快照			|save, restore, saveLayerXxx, restoreToCount, getSaveCount											|依次为 保存当前状态、 回滚到上一次保存的状态、 保存图层状态、 回滚到指定状态、 获取保存次数
|画布变换			|translate, scale, rotate, skew																		|依次为 位移、缩放、 旋转、错切
|Matrix(矩阵)		|getMatrix, setMatrix, concat																		|实际上画布的位移，缩放等操作的都是图像矩阵Matrix， 只不过Matrix比较难以理解和使用，故封装了一些常用的方法。

<!-- more -->

## save and restore

save方法用于临时保存画布坐标系统的状态
restore方法可以用来恢复save之后设置的状态
可以简单理解为调用restore之后，restore方法前调用的rotate/translate/scale方法全部就还原了，画布的坐标系统恢复到save方法之前，
但是这里要注意的是，restore方法的调用只影响restore之后绘制的内容，对restore之前已经绘制到屏幕上的图形不会产生任何影响。

## translate, scale, rotate
通过代码来简单了解(值得注意的是当缩放比例为负数的时候会根据缩放中心轴进行翻转)
``` bash
// 将坐标系原点移动到画布正中心
canvas.translate(mWidth / 2, mHeight / 2);

RectF rect = new RectF(0,-400,400,0);   // 矩形区域

mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
canvas.drawRect(rect,mPaint);

canvas.scale(-0.5f,-0.5f);              // 画布缩放  图1
// canvas.scale(-0.5f,-0.5f,200,0);     // 画布缩放  <-- 缩放中心向右偏移了200个单位,本次对缩放中心点y轴坐标进行了偏移，故中心轴也向右偏移了,图2

//canvas.rotate(180);                     // 旋转180度 <-- 默认旋转中心为原点 图3
// canvas.rotate(180,200,0);            // 旋转180度 <-- 旋转中心向右偏移200个单位 图4

mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
canvas.drawRect(rect,mPaint);
```
![图1](http://img2.ph.126.net/xnLkJXCCaNwDUOAwSJRxig==/6632403672327370448.jpg)

![图2](http://img0.ph.126.net/GClAkrn36_HGx36y7QBwxQ==/6632269531908790611.jpg)

![图3](http://img1.ph.126.net/ZMbhakNpxk-9JdH0oUKiYg==/6631961668655680492.jpg)

## skew 
skew这里翻译为错切，错切是特殊类型的线性变换。

错切只提供了一种方法：

public void skew (float sx, float sy)
参数含义：
float sx:将画布在x方向上倾斜相应的角度，sx倾斜角度的tan值，
float sy:将画布在y轴方向上倾斜相应的角度，sy为倾斜角度的tan值.
``` bash
// 将坐标系原点移动到画布正中心
canvas.translate(mWidth / 2, mHeight / 2);

RectF rect = new RectF(0,0,200,200);   // 矩形区域

mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
canvas.drawRect(rect,mPaint);

canvas.skew(1,0);                       // 水平错切 <- 45度

mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
canvas.drawRect(rect,mPaint);
```
![](https://ws3.sinaimg.cn/large/cf673337jw1f8mjhvhfluj208c0etjrf)


