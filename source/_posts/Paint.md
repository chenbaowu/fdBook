---
title: Paint 详解
date: 2017-05-14 15:22:26
categories: android
tags: [自定义view,paint]
---

## api合集
``` bash
void reset();
void set(Paint src);
void setCompatibilityScaling(float factor);
void setBidiFlags(int flags);
void setFlags(int flags);
void setHinting(int mode);
//是否抗锯齿
void setAntiAlias(boolean aa);
//设定是否使用图像抖动处理，会使绘制出来的图片颜色更加平滑和饱满，图像更加清晰  
void setDither(boolean dither);
//设置线性文本
void setLinearText(boolean linearText);
//设置该项为true，将有助于文本在LCD屏幕上的显示效果  
void setSubpixelText(boolean subpixelText);
//设置下划线
void setUnderlineText(boolean underlineText);
//设置带有删除线的效果 
void setStrikeThruText(boolean strikeThruText);
//设置伪粗体文本，设置在小字体上效果会非常差  
void setFakeBoldText(boolean fakeBoldText);
//如果该项设置为true，则图像在动画进行中会滤掉对Bitmap图像的优化操作
//加快显示速度，本设置项依赖于dither和xfermode的设置  
void setFilterBitmap(boolean filter);
//设置画笔风格，空心或者实心 FILL，FILL_OR_STROKE，或STROKE
//Paint.Style.STROKE 表示当前只绘制图形的轮廓，而Paint.Style.FILL表示填充图形。  
void setStyle(Style style);
 //设置颜色值
void setColor(int color);
//设置透明图0~255，要在setColor后面设置才生效
void setAlpha(int a);   
//设置RGB及透明度
void setARGB(int a, int r, int g, int b);  
//当画笔样式为STROKE或FILL_OR_STROKE时，设置笔刷的粗细度  
void setStrokeWidth(float width);
void setStrokeMiter(float miter);
//当画笔样式为STROKE或FILL_OR_STROKE时，设置笔刷末端的图形样式
void setStrokeCap(Cap cap);
//设置绘制时各图形的结合方式，如平滑效果等  
void setStrokeJoin(Join join);
//设置图像效果，使用Shader可以绘制出各种渐变效果  
Shader setShader(Shader shader);
//设置颜色过滤器，可以在绘制颜色时实现不用颜色的变换效果 
ColorFilter setColorFilter(ColorFilter filter);
//设置图形重叠时的处理方式，如合并，取交集或并集，经常用来制作橡皮的擦除效果 
Xfermode setXfermode(Xfermode xfermode);
//设置绘制路径的效果，如点画线等 
PathEffect setPathEffect(PathEffect effect);
//设置MaskFilter，可以用不同的MaskFilter实现滤镜的效果，如滤化，立体等  
MaskFilter setMaskFilter(MaskFilter maskfilter);
//设置Typeface对象，即字体风格，包括粗体，斜体以及衬线体，非衬线体等  
Typeface setTypeface(Typeface typeface);
//设置光栅化
Rasterizer setRasterizer(Rasterizer rasterizer);
//在图形下面设置阴影层，产生阴影效果，radius为阴影的角度，dx和dy为阴影在x轴和y轴上的距离，color为阴影的颜色
//注意：在Android4.0以上默认开启硬件加速，有些图形的阴影无法显示。关闭View的硬件加速 view.setLayerType(View.LAYER_TYPE_SOFTWARE, null);
void setShadowLayer(float radius, float dx, float dy, int color);
//设置文本对齐
void setTextAlign(Align align);
//设置字体大小
void setTextSize(float textSize);
//设置文本缩放倍数，1.0f为原始
void setTextScaleX(float scaleX);
//设置斜体文字，skewX为倾斜弧度  
void setTextSkewX(float skewX);
//下面几个就是测量字体的长度了
measureText(String text)，measureText(CharSequence text, int start, int end)
//得到文本的边界，上下左右，提取到bounds中，可以通过这计算文本的宽和高
getTextBounds(String text, int start, int end, Rect bounds) 
```
<!-- more -->

## 挑几个说说

setStyle()
Paint.Style.FILL：填充内部
Paint.Style.FILL_AND_STROKE  ：填充内部和描边
Paint.Style.STROKE  ：描边

setStrokeCap()
Paint.Cap.ROUND :
Paint.Cap.SQUARE :
Paint.Cap.BUTT :
![](http://img2.ph.126.net/qQ5AJEYSZJMKe-zHbV05Dw==/6632084813957888759.png)

setStrokeJoin()
Paint.Join.ROUND :
Paint.Join.BEVEL :
Paint.Join.MITER :
![](http://img1.ph.126.net/Ej_f2s_G08eWgW1QywoMlA==/6632581793210969410.png)

### PorterDuffXfermode : Paint.setXfermode(xfermode)
图片混合模式，十分实用的api
``` bash
private static final Xfermode[] sModes = {
    new PorterDuffXfermode(PorterDuff.Mode.CLEAR),      //清空所有，要闭硬件加速，否则无效
    new PorterDuffXfermode(PorterDuff.Mode.SRC),        //显示前都图像，不显示后者
    new PorterDuffXfermode(PorterDuff.Mode.DST),        //显示后者图像，不显示前者
    new PorterDuffXfermode(PorterDuff.Mode.SRC_OVER),   //后者叠于前者
    new PorterDuffXfermode(PorterDuff.Mode.DST_OVER),   //前者叠于后者
    new PorterDuffXfermode(PorterDuff.Mode.SRC_IN),     //显示相交的区域，但图像为后者
    new PorterDuffXfermode(PorterDuff.Mode.DST_IN),     //显示相交的区域，但图像为前者
    new PorterDuffXfermode(PorterDuff.Mode.SRC_OUT),    //显示后者不重叠的图像
    new PorterDuffXfermode(PorterDuff.Mode.DST_OUT),    //显示前者不重叠的图像
    new PorterDuffXfermode(PorterDuff.Mode.SRC_ATOP),   //显示前者图像，与后者重合的图像
    new PorterDuffXfermode(PorterDuff.Mode.DST_ATOP),   //显示后者图像，与前者重合的图像
    new PorterDuffXfermode(PorterDuff.Mode.XOR),        //显示持有不重合的图像
    new PorterDuffXfermode(PorterDuff.Mode.DARKEN),     //后者叠于前者上，后者与前者重叠的部份透明。要闭硬件加速，否则无效
    new PorterDuffXfermode(PorterDuff.Mode.LIGHTEN),    //前者叠于前者，前者与后者重叠部份透明。要闭硬件加速，否则无效
    new PorterDuffXfermode(PorterDuff.Mode.MULTIPLY),   //显示重合的图像，且颜色会合拼
    new PorterDuffXfermode(PorterDuff.Mode.SCREEN) };   //显示持有图像，重合的会变白
```
上个小例子,画一个带有白色边的黄色实心小圆
``` bash

```

