---
title: recyclerview 自定义动画
date: 2017-10-29 22:22:22
categories: android
tags: recyclerview
---

## DefaultItemAnimator 

RecyclerView 默认设置了这个动画的，看源码如果不设置动画就默认使用这个动画
DefaultItemAnimator 是自带的动画效果，我们自定义动画也是参考它的实现,下面一起来分析

``` bash
 public class BaseItemAnimator extends SimpleItemAnimator {
    //Item移除回调
    @Override
    public boolean animateRemove(RecyclerView.ViewHolder holder) {
        return false;
    }

    //Item添加回调
    @Override
    public boolean animateAdd(RecyclerView.ViewHolder holder) {
        return false;
    }


    //用于控制添加，移动更新时，其它Item的动画执行
    @Override
    public boolean animateMove(RecyclerView.ViewHolder holder, int fromX, int fromY, int toX, int toY) {
        return false;
    }

    //Item更新回调
    @Override
    public boolean animateChange(RecyclerView.ViewHolder oldHolder, RecyclerView.ViewHolder newHolder, int fromLeft, int fromTop, int toLeft, int toTop) {
        return false;
    }

    //真正控制执行动画的地方
    @Override
    public void runPendingAnimations() {

    }

    //停止某个Item的动画
    @Override
    public void endAnimation(RecyclerView.ViewHolder item) {

    }

    //停止所有动画
    @Override
    public void endAnimations() {

    }

    @Override
    public boolean isRunning() {
        return false;
    }
}
```

<!-- more -->

具体可以直接看源码，你会发现它的动画执行是这样的，先removed，再move，再change，最后再add
``` bash
		// Next, add stuff
        if (additionsPending) {
            final ArrayList<ViewHolder> additions = new ArrayList<>();
            additions.addAll(mPendingAdditions);
            mAdditionsList.add(additions);
            mPendingAdditions.clear();
            Runnable adder = new Runnable() {
                @Override
                public void run() {
                    for (ViewHolder holder : additions) {
                        animateAddImpl(holder);
                    }
                    additions.clear();
                    mAdditionsList.remove(additions);
                }
            };
            if (removalsPending || movesPending || changesPending) {
                long removeDuration = removalsPending ? getRemoveDuration() : 0;
                long moveDuration = movesPending ? getMoveDuration() : 0;
                long changeDuration = changesPending ? getChangeDuration() : 0;
                long totalDelay = removeDuration + Math.max(moveDuration, changeDuration);
                View view = additions.get(0).itemView;
                ViewCompat.postOnAnimationDelayed(view, adder, totalDelay); // 看到没这个延迟时间的计算
            } else {
                adder.run();
            }
        }
```

## ViewPropertyAnimatorCompat

DefaultItemAnimator 的动画效果都是基于 ViewPropertyAnimatorCompat，拿个例子说说
``` bash
 private void animateRemoveImpl(final ViewHolder holder) {
        final View view = holder.itemView;
        final ViewPropertyAnimatorCompat animation = ViewCompat.animate(view);
        mRemoveAnimations.add(holder);
        animation.setDuration(getRemoveDuration())
                .alpha(0).setListener(new VpaListenerAdapter() {
            @Override
            public void onAnimationStart(View view) {
                dispatchRemoveStarting(holder);
            }

            @Override
            public void onAnimationEnd(View view) {
                animation.setListener(null);
                ViewCompat.setAlpha(view, 1);
                dispatchRemoveFinished(holder);
                mRemoveAnimations.remove(holder);
                dispatchFinishedWhenDone();
            }
        }).start();
    }
```
这里表示的是透明度从1到0变化，你看下add那个是这样的.alpha(1)，ViewPropertyAnimatorCompat里面还有很多动画
``` bash
    interface ViewPropertyAnimatorCompatImpl {
        public void setDuration(ViewPropertyAnimatorCompat vpa, View view, long value);
        public long getDuration(ViewPropertyAnimatorCompat vpa, View view);
        public void setInterpolator(ViewPropertyAnimatorCompat vpa, View view, Interpolator value);
        public Interpolator getInterpolator(ViewPropertyAnimatorCompat vpa, View view);
        public void setStartDelay(ViewPropertyAnimatorCompat vpa, View view, long value);
        public long getStartDelay(ViewPropertyAnimatorCompat vpa, View view);
        public void alpha(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void alphaBy(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void rotation(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void rotationBy(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void rotationX(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void rotationXBy(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void rotationY(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void rotationYBy(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void scaleX(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void scaleXBy(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void scaleY(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void scaleYBy(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void cancel(ViewPropertyAnimatorCompat vpa, View view);
        public void x(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void xBy(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void y(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void yBy(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void z(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void zBy(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void translationX(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void translationXBy(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void translationY(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void translationYBy(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void translationZ(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void translationZBy(ViewPropertyAnimatorCompat vpa, View view, float value);
        public void start(ViewPropertyAnimatorCompat vpa, View view);
        public void withLayer(ViewPropertyAnimatorCompat vpa, View view);
        public void withStartAction(ViewPropertyAnimatorCompat vpa, View view, Runnable runnable);
        public void withEndAction(ViewPropertyAnimatorCompat vpa, View view, Runnable runnable);
        public void setListener(ViewPropertyAnimatorCompat vpa, View view,
                ViewPropertyAnimatorListener listener);
        public void setUpdateListener(ViewPropertyAnimatorCompat vpa, View view,
                ViewPropertyAnimatorUpdateListener listener);
    };
```