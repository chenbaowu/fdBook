---
title: view_anm
date: 2017-11-11 15:08:13
categories: android
tags: 动画
---

## 安卓动画简介

总的来说，Android动画可以分为两类，最初的传统动画和Android3.0 之后出现的属性动画；
传统动画又包括 帧动画（Frame Animation）和补间动画（Tweened Animation）。
温馨提示 ： 本文所有动画没有使用xml的实现只介绍代码实现

<!-- more -->

### 帧动画（Frame Animation）

他的原理就是将一张张单独的图片连贯的进行播放，从而在视觉上产生一种动画的效果；有点类似于某些软件制作gif动画的方式。

``` bash
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:drawable="@drawable/a_0"
        android:duration="100" />
    <item
        android:drawable="@drawable/a_1"
        android:duration="100" />
    <item
        android:drawable="@drawable/a_2"
        android:duration="100" />
</animation-list>

ImageView animationImg1 = (ImageView) findViewById(R.id.animation1);
        animationImg1.setImageResource(R.drawable.frame_anim1);
        AnimationDrawable animationDrawable1 = (AnimationDrawable) animationImg1.getDrawable();
        animationDrawable1.start();
```

### 补间动画（Tweened Animation）

不改变view的实际属性，只改变绘制效果

``` bash
android:fillAfter 如果设置为true，控件动画结束时，将保持动画最后时的状态
android:fillBefore       如果设置为true,控件动画结束时，还原到开始动画前的状态
android:fillEnabled    与android:fillBefore 效果相同，都是在动画结束时，将控件还原到初始化状态
android:repeatCount 重复次数
android:repeatMode	重复类型，有reverse和restart两个值，reverse表示倒序回放，restart表示重新放一遍，必须与repeatCount一起使用才能看到效果。因为这里的意义是重复的类型，即回放时的动作。
android:interpolator  设定插值器

缩放
ScaleAnimation(float fromX, float toX, float fromY, float toY,
            float pivotX, float pivotY) {}
ScaleAnimation(float fromX, float toX, float fromY, float toY,int pivotXType, float pivotXValue, int pivotYType, float pivotYValue) {}

旋转
RotateAnimation(float fromDegrees, float toDegrees, float pivotX, float pivotY){}
RotateAnimation(float fromDegrees, float toDegrees, int pivotXType, float pivotXValue,int pivotYType, float pivotYValue) {

位移
TranslateAnimation(float fromXDelta, float toXDelta, float fromYDelta, float toYDelta) {}		
TranslateAnimation(int fromXType, float fromXValue, int toXType, float toXValue,int fromYType, float fromYValue, int toYType, float toYValue) {}

透明度
AlphaAnimation(float fromAlpha, float toAlpha)

动画集 
AnimationSet animationSet = new AnimationSet(true);
animationSet.setAnimationListener(animationListener);
view.startAnimation(animationSet);
```

### 属性动画

改变的是对象的实际属性

1.ValueAnimator 

``` bash 

public static ValueAnimator ofInt(int... values)  
public static ValueAnimator ofFloat(float... values)
参数类型都是可变参数长参数，所以我们可以传入任何数量的值；
传进去的值列表，就表示动画时的变化范围；比如ofInt(5,88,55)就表示从数值5变化到数字88再变化到数字55

ValueAnimator animator = ValueAnimator.ofInt(0,500);  
animator.setDuration(1000);  

animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {  
    @Override  
    public void onAnimationUpdate(ValueAnimator animation) {  
        int curValue = (int)animation.getAnimatedValue();   
    }  
});  
animator.start(); 
```

2.ObjectAnimator

``` bash
ObjectAnimator oa=ObjectAnimator.ofFloat(tv, "alpha", 0f, 1f); // "rotation" , "translationX" , "scaleX" ...

动画集
AnimatorSet bouncer = new AnimatorSet();
bouncer.play(anim1).before(anim2);
bouncer.play(anim2).with(anim3);
bouncer.play(anim2).with(anim4)
bouncer.play(anim5).after(amin2);
animatorSet.start();
```

### ViewPropertyAnimator

如果需要对一个View的多个属性进行动画可以用ViewPropertyAnimator类，该类对多属性动画进行了优化，会合并一些invalidate()来减少刷新视图，该类在3.1中引入

``` bash
PropertyValuesHolder pvhX = PropertyValuesHolder.ofFloat("x", 50f);
PropertyValuesHolder pvhY = PropertyValuesHolder.ofFloat("y", 100f);
ObjectAnimator.ofPropertyValuesHolder(myView, pvhX, pvyY).start();
myView.animate().x(50f).y(100f);
```

### LayoutTransition

在3.0及以后只需要在XML中设置animateLayoutChanges="true"或者在Java代码中添加一个LayoutTransition对象

1.APPEARING：容器中出现一个视图。

2.DISAPPEARING：容器中消失一个视图。

3.CHANGING：布局改变导致某个视图随之改变，例如调整大小，但不包括添加或者移除视图。

4.CHANGE_APPEARING：其他视图的出现导致某个视图改变。

5.CHANGE_DISAPPEARING：其他视图的消失导致某个视图改变。

``` bash
View mContainer = (LinearLayout) findViewById(R.id.main_container);  
LayoutTransition transition = new LayoutTransition();  
mContainer.setLayoutTransition(transition);  
findViewById(R.id.main_btn).setOnClickListener(this);  
  
//使用翻转进入的动画代替默认动画  
Animator appearAnim = ObjectAnimator  
        .ofFloat(null, "rotationY", 90f, 0)  
        .setDuration(transition.getDuration(LayoutTransition.APPEARING));  
transition.setAnimator(LayoutTransition.APPEARING, appearAnim);  
  
//使用翻转消失的动画代替默认动画  
Animator disappearAnim = ObjectAnimator.ofFloat(null, "rotationX", 0, 90f).setDuration( transition.getDuration(LayoutTransition.DISAPPEARING));  
transition.setAnimator(LayoutTransition.DISAPPEARING, disappearAnim);  
  
//使用滑动动画代替默认布局改变的动画  
//这个动画会让视图滑动进入并短暂地缩小一半，具有平滑和缩放的效果  
PropertyValuesHolder pvhSlide = PropertyValuesHolder.ofFloat("y", 0, 1);  
PropertyValuesHolder pvhScaleY = PropertyValuesHolder.ofFloat("scaleY", 1f, 0.5f, 1f);  
PropertyValuesHolder pvhScaleX = PropertyValuesHolder.ofFloat("scaleX", 1f, 0.5f, 1f);  
  
//这里将上面三个动画综合  
Animator changingDisappearAnim = ObjectAnimator.ofPropertyValuesHolder(this, pvhSlide, pvhScaleY, pvhScaleX);  
changingDisappearAnim.setDuration(transition.getDuration(LayoutTransition.CHANGE_DISAPPEARING));  
transition.setAnimator(LayoutTransition.CHANGE_DISAPPEARING,changingDisappearAnim);  
```

### Keyframes

可以让我们定义除了开始和结束以外的关键帧。KeyFrame是抽象类，要通过ofInt(),ofFloat(),ofObject()获得适当的KeyFrame，
然后通过PropertyValuesHolder.ofKeyframe获得PropertyValuesHolder对象

``` bash
//设置btn对象的width属性值使其：开始时 Width=400，动画开始1/4时 Width=200，动画开始1/2时 Width=400，动画开始3/4时 Width=100，动画结束时 Width=500
Keyframe kf0 = Keyframe.ofInt(0, 400);
Keyframe kf1 = Keyframe.ofInt(0.25f, 200);
Keyframe kf2 = Keyframe.ofInt(0.5f, 400);
Keyframe kf4 = Keyframe.ofInt(0.75f, 100);
Keyframe kf3 = Keyframe.ofInt(1f, 500);
PropertyValuesHolder pvhRotation = PropertyValuesHolder.ofKeyframe(width, kf0, kf1, kf2, kf4, kf3);
ObjectAnimator rotationAnim = ObjectAnimator.ofPropertyValuesHolder(btn, pvhRotation);
```

### TimeInterplator

Time interplator定义了属性值变化的方式，如线性均匀改变，开始慢然后逐渐快等。在Property Animation中是TimeInterplator，在View Animation中是Interplator，这两个是一样的，在3.0之前只有Interplator，3.0之后实现代码转移至了TimeInterplator。Interplator继承自TimeInterplator，内部没有任何其他代码。

``` bash
AccelerateInterpolator　　　　　     加速，开始时慢中间加速
DecelerateInterpolator　　　 　　   减速，开始时快然后减速
AccelerateDecelerateInterolator　   先加速后减速，开始结束时慢，中间加速
AnticipateInterpolator　　　　　　  反向 ，先向相反方向改变一段再加速播放
AnticipateOvershootInterpolator　   反向加回弹，先向相反方向改变，再加速播放，会超出目的值然后缓慢移动至目的值
BounceInterpolator　　　　　　　  跳跃，快到目的值时值会跳跃，如目的值100，后面的值可能依次为85，77，70，80，90，100
CycleIinterpolator　　　　　　　　 循环，动画循环一定次数，值的改变为一正弦函数：Math.sin(2 * mCycles * Math.PI * input)
LinearInterpolator　　　　　　　　 线性，线性均匀改变
OvershottInterpolator　　　　　　  回弹，最后超出目的值然后缓慢改变到目的值
TimeInterpolator　　　　　　　　   一个接口，允许你自定义interpolator，以上几个都是实现了这个接口

自定义 
public class MyInterploator implements TimeInterpolator {  
    @Override  
    public float getInterpolation(float input) {  
        return 1-input;  
    }  
}  
```

### TypeEvalutors

根据属性的开始、结束值与TimeInterpolation计算出的因子计算出当前时间的属性值，android提供了以下几个evalutor：

IntEvaluator：属性的值类型为int；
FloatEvaluator：属性的值类型为float；
ArgbEvaluator：属性的值类型为十六进制颜色值；
TypeEvaluator：一个接口，可以通过实现该接口自定义Evaluator。
自定义TypeEvalutor很简单，只需要实现一个方法，如FloatEvalutor的定义：

``` bash
public class FloatEvaluator implements TypeEvaluator {
    public Object evaluate(float fraction, Object startValue, Object endValue) {
        float startFloat = ((Number) startValue).floatValue();
        return startFloat + fraction * (((Number) endValue).floatValue() - startFloat);
    }
}
根据动画执行的时间跟应用的Interplator，会计算出一个0~1之间的因子，即evalute函数中的fraction参数，通过上述FloatEvaluator应该很好看出其意思。
```

