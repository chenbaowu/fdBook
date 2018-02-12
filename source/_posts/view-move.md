---
title: view_move
date: 2017-12-12 11:54:11
categories: android
tags: view
---

## 浅谈view移动的几种方式

### 通过改变布局参数来实现View的滑动

会影响四个位置的值

``` bash
MarginLayoutParams params = (MarginLayoutParams) mButton.getLayoutParams();
params.leftMargin += 520;
mButton.requestLayout();
```

<!-- more -->

### 使用scrollTo/scrollBy实现View的滑动

实际上，调用scrollBy/scrollTo方法只能实现View的内容的滚动，而View的四个位置参数(l,t,r,b)是保持不变的(listview ,recyclerview等)
向右滚动时mScrollX负的，向左滚动时mScrollX是正的。同理，向下滚动时，mScrollY是负的，向上滚动时，mScrollY是正的

点击事件还是在原位置
``` bash
/** 
   * Set the scrolled position of your view. This will cause a call to 

   * {@link #onScrollChanged(int, int, int, int)} and the view will be 

   * invalidated. 

   * @param x the x position to scroll to 

   * @param y the y position to scroll to 

   */  
public void scrollTo(int x, int y) {  
      if (mScrollX != x || mScrollY != y) {  
          int oldX = mScrollX;  
          int oldY = mScrollY;  
          mScrollX = x;  
          mScrollY = y;  
          onScrollChanged(mScrollX, mScrollY, oldX, oldY);  
          if (!awakenScrollBars()) {  
              invalidate();  
          }  
      }  
}  

/** 
    * Move the scrolled position of your view. This will cause a call to 
    * {@link #onScrollChanged(int, int, int, int)} and the view will be 
    * invalidated. 
    * @param x the amount of pixels to scroll by horizontally 
    * @param y the amount of pixels to scroll by vertically 
*/  
public void scrollBy(int x, int y) {  
      scrollTo(mScrollX + x, mScrollY + y);  
}

通过以上代码的33~35行我们可以看到，scrollBy方法内部也是调用了scrollTo方法来实现。
以上源码中我们注意到了mScrollX和mScrollY成员变量，前者是View的左边缘减去View的内容的左边缘，后者是View的上边缘减去View的内容的上边缘
```

### 使用Scroller来实现弹性滑动

Scroller 实现弹性滑动的原理：invaldate方法会导致View的draw方法被调用，而draw会调用computeScroll方法，因此重写了computeScroll方法，
而computeScrollOffset方法会根据时间的流逝动态的计算出很小的一段时间应该滑动多少距离。也就是把一次滑动拆分成无数次小距离滑动从而实现“弹性滑动”
所以必须自己写个view重载computeScroll方法
``` bash
Scroller scroller = new Scroller(mContext);

private void smoothScrollTo(int dstX, int dstY) {
    int scrollX = getScrollX();
    int delta = dstX - scrollX;
    scroller.startScroll(scrollX, 0, delta, 0, 1000);
    invalidate(); // 必须调用invalidate()才,否则不会调用computeScroll()方法。看不到滚动效果 
}

@Override
public void computeScroll() {
    if (scroller.computeScrollOffset()) {
        scrollTo(scroller.getCurrX(), scroller.getCurY());
        postInvalidate(); //一定要调用  
    }
}

// startScroll 的源码
public void startScroll(int startX, int startY, int dx, int dy, int duration) {  
    mMode = SCROLL_MODE;  
    mFinished = false;  
    mDuration = duration;  
    mStartTime = AnimationUtils.currentAnimationTimeMillis();  
    mStartX = startX;  
    mStartY = startY;  
    mFinalX = startX + dx;  
    mFinalY = startY + dy;  
    mDeltaX = dx;  
    mDeltaY = dy;  
    mDurationReciprocal = 1.0f / (float) mDuration;  
    
    mViscousFluidScale = 8.0f;  
   
    mViscousFluidNormalize = 1.0f;  
    mViscousFluidNormalize = 1.0f / viscousFluid(1.0f);  
}

从以上的源码我们可以看到，startScroll方法中并没有进行实际的滚动操作，而是把startX、startY、deltaX、deltaY等参数都保存了下来。
我们看到第7行调用了invalidate方法，这个方法会请求重绘View，这会导致View的draw的方法被调用，draw的方法内部会调用computeScroll方法。
我们来看看第13行，调用了scrollTo方法，并传入mScroller.getCurrX()和mScroller.getCurrY()方法作为参数。这两个参数是在第12行调用的computeScrollOffset方法中设置的，我们来看看这个方法中设置这两个参数的相关代码
public boolean computeScrollOffset() {
    ...
    int timePassed = (int) (AnimationUtils.currentAnimationTimeMillis() - mStartTime);
    if (timePassed < mDuration) {
        switch (mMode) {
            case SCROLL_MODE:
                final float x = mInterpolator.getInterpolation(timePassed * mDurationReciprocal);
                mCurrX = mStartX + Math.round(x * mDeltaX);
                mCurrY = mStartY + Math.rounc(y * mDeltaY);
                break;
        ...
        }
    }
    return true;
}
```

### layout() and offsetTopAndBottom/offsetLeftAndRight

主要是通过坐标位置的改变产生移动效果

``` bash
 layout(getLeft() + 5,getTop() + 5,getRight() + 5,getBottom() + 5);
		
//  offsetLeftAndRight(5); 作用同上
//  offsetTopAndBottom(5);
```

### setTranslationX/Y

这个方法的底层实现主要是通过metrix矩阵变换来的，坐标位置没有改变 ，点击事件的位置会变
ObjectAnimator 实际上也是通过它实现

``` bash
 /**
     * Sets the horizontal location of this view relative to its {@link #getLeft() left} position.
     * This effectively positions the object post-layout, in addition to wherever the object's
     * layout placed it.
     *
     * @param translationX The horizontal position of this view relative to its left position,
     * in pixels.
     *
     * @attr ref android.R.styleable#View_translationX
     */
    public void setTranslationX(float translationX) {
        if (translationX != getTranslationX()) {
            invalidateViewProperty(true, false);
            mRenderNode.setTranslationX(translationX);
            invalidateViewProperty(false, true);

            invalidateParentIfNeededAndWasQuickRejected();
            notifySubtreeAccessibilityStateChangedIfNeeded();
        }
    }
	
从源码可以看出，它的参数不是一个相对位移，而是绝对的偏移值，所以设置多少次都不会叠加
```



