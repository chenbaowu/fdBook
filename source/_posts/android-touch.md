---
title: android_touch
date: 2018-01-01 11:11:11
categories: android
tags: view
---

# android 事件机制全解

一个点击事件产生后，传递顺序是：Activity（Window） -> ViewGroup -> View

默认情况下（没有自己重写派发拦截）从Activity A—->ViewGroup B—>View C，从上往下调用dispatchTouchEvent()
再由View C—>ViewGroup B —>Activity A，从下往上调用onTouchEvent()

<!-- more -->

![](http://upload-images.jianshu.io/upload_images/944365-aa8416fc6d2e5ecd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 事件传递方法 （执行顺序如下）

### 事件派发 dispatchTouchEvent() 使用对象 Activity、ViewGroup、View

当点击事件能够传递给当前View时，该方法就会被调用，也就是该view（viewGroup）首先要有注册事件否则不会调用

返回值
super：根据当前对象的不同而返回方法不同 （建议使用 ，不要随便返回true 或者 false）

|对象			|返回方法						|说明	
| --------  	| 	-----:   					| :----: |
|Activity		|super.dispatchTouchEvent()		|即调用父类ViewGroup的dispatchTouchEvent()	
|ViewGroup		|onIntercepTouchEvent()			|即调用自身的onIntercepTouchEvent()	
|View			|onTouchEvent（）				|即调用自身的onTouchEvent（）	
	
true：消费事件，即事件不继续往下传递
false：不消费事件，事件也不继续往下传递 / 交由给父控件onTouchEvent（）处理

### 事件拦截 onInterceptTouchEvent() 使用对象 ViewGroup（注：Activity、View都没该方法）

调用时刻 在ViewGroup的dispatchTouchEvent()内部调用

返回值
ture ：调用自身的onTouchEvent（）
super | false : 调用子类的dispatchTouchEvent()

### 事件处理 onTouchEvent()

| 事件  				| 简介	 | 
| --------  			| :----: |
|ACTION_DOWN			|手指 初次接触到屏幕 时触发。
|ACTION_MOVE    		|	手指 在屏幕上滑动 时触发，会多次触发。
|ACTION_UP      		|	手指 离开屏幕 时触发。
|ACTION_CANCEL			|事件 被上层拦截 时触发。
|ACTION_OUTSIDE			|手指 不在控件区域 时触发。
|ACTION_POINTER_DOWN	|有非主要的手指按下(即按下之前已经有手指在屏幕上)。
|ACTION_POINTER_UP		|有非主要的手指抬起(即抬起之后仍然有手指在屏幕上)。
|getAction()			|获取事件类型。
|getX()					|获得触摸点在当前 View 的 X 轴坐标。
|getY()					|获得触摸点在当前 View 的 Y 轴坐标。
|getRawX()				|获得触摸点在整个屏幕的 X 轴坐标。
|getRawY()				|获得触摸点在整个屏幕的 Y 轴坐标。

#### 多点触控

多点触控获取事件类型请使用 getActionMasked() 。
追踪事件流请使用 PointId。

##### index 和 pointId 的变化规则 (Index 会变化，pointId 始终不变。)

index
1、从 0 开始，自动增长。
2、如果之前落下的手指抬起，后面手指的 Index 会随之减小。
3、Index 变化趋向于第一次落下的数值(落下手指时，前面有空缺会优先填补空缺)。
4、对 move 事件无效

pointId
1. 从 0 开始，自动增长。
2. 落下手指时优先填补空缺(填补之前抬起手指的编号)。


