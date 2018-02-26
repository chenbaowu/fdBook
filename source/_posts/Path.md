---
title: Path
date: 2017-08-20 15:20:20
tags: view
categories: android
---

## whant is path

Path封装了由直线和曲线(二次，三次贝塞尔曲线)构成的几何路径。你能用Canvas中的drawPath来把这条路径画出来(同样支持Paint的不同绘制模式)，
也可以用于剪裁画布和根据路径绘制文字。我们有时会用Path来描述一个图像的轮廓，所以也会称为轮廓线(轮廓线仅是Path的一种使用方法，两者并不等价)

<!-- more -->

## api

|作用		  		|相关方法			|备注
| --------  		| 	-----:   		| :----: |
|移动起点	  		|moveTo		        |移动下一次操作的起点位置
|设置终点	  		|setLastPoint		|重置当前path中最后一个点位置，如果在绘制之前调用，效果和moveTo相同
|连接直线	  		|lineTo				|添加上一个点到当前点之间的直线到Path
|闭合路径	  		|close				|连接第一个点连接到最后一个点，形成一个闭合区域
|添加内容	  		|addRect, addRoundRect, addOval, addCircle, addPath, addArc, arcTo  |添加(矩形， 圆角矩形， 椭圆， 圆， 路径， 圆弧) 到当前Path (注意addArc和arcTo的区别)
|是否为空	  		|isEmpty			|判断Path是否为空
|是否为矩形	  		|isRect				|判断path是否是一个矩形
|替换路径	  		|set				|用新的路径替换到当前路径所有内容
|偏移路径	  		|offset				|对当前路径之前的操作进行偏移(不会影响之后的操作)
|贝塞尔曲线	  		|quadTo, cubicTo	|分别为二次和三次贝塞尔曲线的方法
|rXxx方法	  		|rMoveTo, rLineTo, rQuadTo, rCubicTo								|不带r的方法是基于原点的坐标系(偏移量)， rXxx方法是基于当前点坐标系(偏移量)
|填充模式	  		|setFillType, getFillType, isInverseFillType, toggleInverseFillType |设置,获取,判断和切换填充模式
|提示方法	  		|incReserve			|提示Path还有多少个点等待加入(这个方法貌似会让Path优化存储结构)
|布尔操作(API19)	|op					|对两个Path进行布尔运算(即取交集、并集等操作)
|计算边界	        |computeBounds		|计算Path的边界
|重置路径	        |reset, rewind		|清除Path中的内容 . 不保留内部数据结构，但会保留FillType ;会保留内部的数据结构，但不保留FillType
|矩阵操作			|transform			|矩阵变换

## moveTo 和 setLastPoint

moveTo只改变下次操作的起点,setLastPoint是重置上一次操作的最后一个点

## addXxx与arcTo

``` bash
// 第一类(基本形状)
// 圆形
public void addCircle (float x, float y, float radius, Path.Direction dir)
// 椭圆
public void addOval (RectF oval, Path.Direction dir)
// 矩形
public void addRect (float left, float top, float right, float bottom, Path.Direction dir)
public void addRect (RectF rect, Path.Direction dir)
// 圆角矩形
public void addRoundRect (RectF rect, float[] radii, Path.Direction dir)
public void addRoundRect (RectF rect, float rx, float ry, Path.Direction dir) 

// 第二类(Path)
// path
public void addPath (Path src)
public void addPath (Path src, float dx, float dy)
public void addPath (Path src, Matrix matrix)

addArc	添加一个圆弧到path	直接添加一个圆弧到path中
arcTo	添加一个圆弧到path	添加一个圆弧到path，如果圆弧的起点和上次最后一个坐标点不相同，就连接两个点
```

## rXxx

rXxx方法的坐标使用的是相对位置(基于当前点的位移)，而之前方法的坐标是绝对位置(基于当前坐标系的坐标)。
比如，path.moveTo(100,100);path.rLineTo(100,200);在使用rLineTo之前，当前点的位置在 (100,100) ， 
使用了 rLineTo(100,200) 之后，下一个点的位置是在当前点的基础上加上偏移量得到的，即 (100+100, 100+200) 这个位置，故最终结果如上所示

## 贝塞尔曲线

数据点	确定曲线的起始和结束位置
控制点	确定曲线的弯曲程度

一阶曲线其实就是前面讲解过的lineTo
二阶曲线对应的方法是quadTo
![](http://photo.163.com/1534598088@qq.com/#m=2&aid=310136082&pid=9835406060)
三阶曲线对应的方法是cubicTo
![](http://photo.163.com/1534598088@qq.com/#m=2&aid=310136082&pid=9835375418)

## PathMeasure

### api

| 返回值	|方法名																		|释义
| --------  | 	-----:   																| :----: |
| void		|setPath(Path path, boolean forceClosed)									|关联一个Path
| boolean	|isClosed()																	|是否闭合
| float		|getLength()																|获取Path的长度
| boolean	|nextContour()																|跳转到下一个轮廓
| boolean	|getSegment(float startD, float stopD, Path dst, boolean startWithMoveTo)	|截取片段
| boolean	|getPosTan(float distance, float[] pos, float[] tan)						|获取指定长度的位置坐标及该点切线值
| boolean	|getMatrix(float distance, Matrix matrix, int flags)						|获取指定长度的位置坐标及该点Matrix

### 构造函数

``` bash
PathMeasure ()
PathMeasure (Path path, boolean forceClosed) // 第二个参数是用来确保 Path 闭合，如果设置为 true,则不论之前Path是否闭合，都会自动闭合该 Path(如果Path可以闭合的话)
```

### getSegment

``` bash
boolean getSegment (float startD, float stopD, Path dst, boolean startWithMoveTo)
```

|参数				|作用								|备注
| --------  		| 	-----:   						| :----: |
|返回值(boolean)	|判断截取是否成功					|true 表示截取成功，结果存入dst中，false 截取失败，不会改变dst中内容
|startD				|开始截取位置距离 Path 起点的长度	|取值范围: 0 <= startD < stopD <= Path总长度
|stopD				|结束截取位置距离 Path 起点的长度	|取值范围: 0 <= startD < stopD <= Path总长度
|dst				|截取的 Path 将会添加到 dst 中		|注意: 是添加，而不是替换
|startWithMoveTo	|起始点是否使用 moveTo				|用于保证截取的 Path 第一个点位置不变

### nextContour

我们知道 Path 可以由多条曲线构成，但不论是 getLength , getgetSegment 或者是其它方法，都只会在其中第一条线段上运行，
而这个 nextContour 就是用于跳转到下一条曲线到方法，如果跳转成功，则返回 true， 如果跳转失败，则返回 false。

## FillType 

|模式				|简介
| --------  		| :----: |  						
|EVEN_ODD			|奇偶规则
|INVERSE_EVEN_ODD	|反奇偶规则
|WINDING			|非零环绕数规则
|INVERSE_WINDING	|反非零环绕数规则

## 布尔操作

|逻辑名称				|类比		|说明	
| --------  			| 	-----:  | :----: |
|DIFFERENCE				|差集		|Path1中减去Path2后剩下的部分	
|REVERSE_DIFFERENCE		|差集		|Path2中减去Path1后剩下的部分	
|INTERSECT				|交集		|Path1与Path2相交的部分	
|UNION					|并集		|包含全部Path1和Path2	
|XOR					|异或		|包含Path1与Path2但不包括两者相交的部分
	
``` bash
 对 path1 和 path2 执行布尔运算，运算方式由第二个参数指定，运算结果存入到path1中。
path1.op(path2, Path.Op.DIFFERENCE);

// 对 path1 和 path2 执行布尔运算，运算方式由第三个参数指定，运算结果存入到path3中。
path3.op(path1, path2, Path.Op.DIFFERENCE)

这个方法主要作用是计算Path所占用的空间以及所在位置,方法如下：
void computeBounds (RectF bounds, boolean exact)
```


