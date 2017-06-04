---
title: 初识 android View
date: 2017-05-14 13:57:51
categories: android
tags: 自定义view
---

## View的坐标系
![](http://img2.ph.126.net/xkg-KtxNMPdvkVHUtntDiA==/6631898996492790587.jpg)

<!-- more -->
## 从构造函数开始
``` bash
public void SloopView(Context context) {} // 一般在直接New一个View的时候调用。
// 一般在layout文件中使用的时候会调用，关于它的所有属性(包括自定义属性)都会包含在attrs中传递进来。
public void SloopView(Context context, AttributeSet attrs) {} 
public void SloopView(Context context, AttributeSet attrs, int defStyleAttr) {}
public void SloopView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {} // 有四个参数的构造函数在API21的时候才添加上
```
有三个参数的构造函数中第三个参数是默认的Style，这里的默认的Style是指它在当前Application或Activity所用的Theme中的默认Style，
且只有在明确调用的时候才会生效，以系统中的ImageButton为例说明：
``` bash
public ImageButton(Context context, AttributeSet attrs) {
    //调用了三个参数的构造函数，明确指定第三个参数
    this(context, attrs, com.android.internal.R.attr.imageButtonStyle);
}

public ImageButton(Context context, AttributeSet attrs, int defStyleAttr) {
    //此处调了四个参数的构造函数，无视即可
    this(context, attrs, defStyleAttr, 0); 
}
//注意：即使你在View中使用了Style这个属性也不会调用三个参数的构造函数，所调用的依旧是两个参数的构造函数
```

## view 的绘制过程 (注意不要在里面做ui更新操作)
![](http://img0.ph.126.net/EMLvPoL268zzoNuT0hAfkg==/6632319009931934028.jpg)
onMeasure: 测量View大小, View的大小不仅由自身所决定，同时也会受到父控件的影响，为了我们的控件能更好的适应各种情况，一般会自己进行测量。
``` bash
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    int widthsize  MeasureSpec.getSize(widthMeasureSpec);      //取出宽度的确切数值
    int widthmode  MeasureSpec.getMode(widthMeasureSpec);      //取出宽度的测量模式
    
    int heightsize  MeasureSpec.getSize(heightMeasureSpec);    //取出高度的确切数值
    int heightmode  MeasureSpec.getMode(heightMeasureSpec);    //取出高度的测量模式
}
```
测量模式一共有三种， 被定义在 Android 中的 View 类的一个内部类View.MeasureSpec中：

|  模式			|二进制数值		|描述
| --------  | 	-----:   		| :----: |
|UNSPECIFIED	|00	|			默认值，父控件没有给子view任何限制，子View可以设置为任意大小。
|EXACTLY		|01	|			表示父控件已经确切的指定了子View的大小。
|AT_MOST		|10	|			表示子View具体大小没有尺寸限制，但是存在上限，上限一般为父View大小。

注意：如果对View的宽高进行修改了，不要调用 super.onMeasure( widthMeasureSpec, heightMeasureSpec); 要调用 setMeasuredDimension( widthsize, heightsize); 这个函数

onSizeChanged: 界面大小变化系统自动调,onSizeChanged(w, h, oldw, oldh)四个参,分别为 宽度,高度，上一次宽度，上一次高度

onLayout: 确定子View布局位置,child.layout(l, t, r, b); // 注意都是相对父布局的

## 实用api
``` bash
event.getX();       //触摸点相对于其所在组件坐标系的坐标
event.getY();

event.getRawX();    //触摸点相对于屏幕默认坐标系的坐标
event.getRawY();

getTop();       //获取子View左上角距父View顶部的距离
getLeft();      //获取子View左上角距父View左侧的距离
getBottom();    //获取子View右下角距父View顶部的距离
getRight();     //获取子View右下角距父View左侧的距离
```