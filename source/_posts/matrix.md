---
title: 浅谈matrix
date: 2017-05-13 16:18:48
categories: android
tags: matrix
---

## 基本api

| 方法类别  | 				相关API	    							|  摘要  |
| --------  | 				-----:   								| :----: |
|基本方法	|equals hashCode toString toShortString					|比较、 获取哈希值、 转换为字符串
|数值操作	|set reset setValues getValues							|设置、 重置、 设置数值、 获取数值
|数值计算	|mapPoints mapRadius mapRect mapVectors					|计算变换后的数值
|设置(set)	|setConcat setRotate setScale setSkew setTranslate		|设置变换
|前乘(pre)	|preConcat preRotate preScale preSkew preTranslate		|前乘变换
|后乘(post)	|postConcat postRotate postScale postSkew postTranslate	|后乘变换
|特殊方法	|setPolyToPoly setRectToRect rectStaysRect setSinCos	|一些特殊操作
|矩阵相关	|invert isAffine(API21) isIdentity						|求逆矩阵、 是否为仿射矩阵、 是否为单位矩阵

<!-- more -->

## what is matrix

Matrix是一个矩阵，主要功能是坐标映射，数值转换。

![](http://img2.ph.126.net/lLgxenxlqDF5YxZLpzNWCA==/6631958370120993878.jpg)

![](http://img2.ph.126.net/vsbc2LIGtxuljuHNXf4wmw==/6632187068539571318.jpg)
1.缩放(Scale)

x = k1 * x0
y = k2 * y0

用矩阵表示:
[x]	  [k1 0  0] [x0]
[y] = [0  k2 0] [y0]
[1]	  [0  0  1] [1]

2.错切(Skew)
![](http://img2.ph.126.net/PX_kROAusLUUf5wlOSfXYQ==/6632148585632598953.png)

3.旋转(Rotate)
![](http://img0.ph.126.net/F_w5X6pSBGRTDgOuYLnTQQ==/6632080415911681023.png)

4.平移(Translate)
![](http://img0.ph.126.net/PD1xNmJl62omqzWRDIaxmA==/6631940777934952104.png)

5.每一种操作在Matrix均有三类,前乘(pre)，后乘(post)和设置(set),设置使用的不是矩阵乘法，而是直接覆盖掉原来的数值,这点要注意

## Matrix 常用api

1.setValues getValues
``` bash
void setValues (float[] values)
setValues的参数是浮点型的一维数组，长度需要大于9，拷贝数组中的前9位数值赋值给当前Matrix。

void getValues (float[] values) 
```
2.mapPoints
``` bash
void mapPoints (float[] pts)

void mapPoints (float[] dst, float[] src)

void mapPoints (float[] dst, int dstIndex,float[] src, int srcIndex, int pointCount)

计算一组点基于当前Matrix变换后的位置，(由于是计算点，所以参数中的float数组长度一般都是偶数的,若为奇数，则最后一个数值不参与计算)
```
3.mapRadius
``` bash
float mapRadius (float radius)
测量半径，由于圆可能会因为画布变换变成椭圆，所以此处测量的是平均半径
```
4.mapRect : 测量矩形变换后位置
``` bash
boolean mapRect (RectF rect)

boolean mapRect (RectF dst, RectF src)
```
5.4.mapVectors : 测量向量
``` bash
void mapVectors (float[] vecs)

void mapVectors (float[] dst, float[] src)

void mapVectors (float[] dst, int dstIndex, float[] src, int srcIndex, int vectorCount)

//mapVectors 与 mapPoints 基本上是相同的，可以直接参照上面的mapPoints使用方法。而两者唯一的区别就是mapVectors不会受到位移的影响，这符合向量的定律

float[] src = new float[]{1000, 800};
float[] dst = new float[2];

// 构造一个matrix
Matrix matrix = new Matrix();
matrix.setScale(0.5f, 1f);
matrix.postTranslate(100,100);

// 计算向量, 不受位移影响
matrix.mapVectors(dst, src);
Log.i(TAG, "mapVectors: "+Arrays.toString(dst));

// 计算点
matrix.mapPoints(dst, src);
Log.i(TAG, "mapPoints: "+Arrays.toString(dst));

//结果:
mapVectors: [500.0, 800.0]
mapPoints: [600.0, 900.0]
```

## Matrix 特殊使用

1.invert
求矩阵的逆矩阵，简而言之就是计算与之前相反的矩阵，如果之前是平移200px，则求的矩阵为反向平移200px，如果之前是缩小到0.5f，则结果是放大到2倍

2.isAffine
判断矩阵是否是仿射矩阵, 貌似并没有太大卵用，因为你无论如何操作结果始终都为true。

3.isIdentity ：判断是否为单位矩阵

4. 获取View在屏幕上的绝对位置
``` bash
@Override
protected void onDraw(Canvas canvas) {
    float[] values = new float[9];
    int[] location1 = new int[2];

    Matrix matrix = canvas.getMatrix();
    matrix.getValues(values);

    location1[0] = (int) values[2];
    location1[1] = (int) values[5];
    Log.i(TAG, "location1 = " + Arrays.toString(location1));

    int[] location2 = new int[2];
    this.getLocationOnScreen(location2);
    Log.i(TAG, "location2 = " + Arrays.toString(location2));
}
```
5.setRectToRect
``` bash
boolean setRectToRect (RectF src,           // 源区域
                RectF dst,                  // 目标区域
                Matrix.ScaleToFit stf)      // 缩放适配模式
简单来说就是将源矩形的内容填充到目标矩形中，然而在大多数的情况下，源矩形和目标矩形的长宽比是不一致的，到底该如何填充呢，这个填充的模式就由第三个参数 stf 来确定
```
ScaleToFit 是一个枚举类型，共包含了四种模式:

| 方法类别  |  摘要  |
| --------  | :----: |
|CENTER	| 居中，对src等比例缩放，将其居中放置在dst中
|START	| 顶部，对src等比例缩放，将其放置在dst的左上角。
|END	| 底部，对src等比例缩放，将其放置在dst的右下角。
|FILL	| 充满，拉伸src的宽和高，使其完全填充满dst。

6.setPolyToPoly : 可以用来做3d的效果,Poly全称是Polygon,多边形的意思,与PS中自由变换中的扭曲有点类似。
``` bash
boolean setPolyToPoly (
        float[] src,    // 原始数组 src [x,y]，存储内容为一组点
        int srcIndex,   // 原始数组开始位置
        float[] dst,    // 目标数组 dst [x,y]，存储内容为一组点
        int dstIndex,   // 目标数组开始位置
        int pointCount) // 测控点的数量 取值范围是: 0到4
```

从参数我们可以了解到setPolyToPoly最多可以支持4个点，这四个点通常为图形的四个角，可以通过这四个角将视图从矩形变换成其他形状。

| pointCount  | 摘要 |
| --------  | :----: |
|	  0	    | 相当于reset
|     1		| 相当于translate				
|     2		| 可以进行 缩放、旋转、平移 变换
|     3		| 可以进行 缩放、旋转、平移、错切 变换 
|     4		| 可以进行 缩放、旋转、平移、错切以及任何形变 
``` bash
public class MatrixSetPolyToPolyTest extends View {

    private Bitmap mBitmap;             // 要绘制的图片
    private Matrix mPolyMatrix;         // 测试setPolyToPoly用的Matrix

    public MatrixSetPolyToPolyTest(Context context) {
        super(context);

        initBitmapAndMatrix();
    }

    private void initBitmapAndMatrix() {
        mBitmap = BitmapFactory.decodeResource(getResources(),
                R.drawable.poly_test);

        mPolyMatrix = new Matrix();


        float[] src = {0, 0,                                    // 左上
                mBitmap.getWidth(), 0,                          // 右上
                mBitmap.getWidth(), mBitmap.getHeight(),        // 右下
                0, mBitmap.getHeight()};                        // 左下

        float[] dst = {0, 0,                                    // 左上
                mBitmap.getWidth(), 400,                        // 右上
                mBitmap.getWidth(), mBitmap.getHeight() - 200,  // 右下
                0, mBitmap.getHeight()};                        // 左下

        // 核心要点
        mPolyMatrix.setPolyToPoly(src, 0, dst, 0, src.length >> 1); // src.length >> 1 为位移运算 相当于处以2

        // 此处为了更好的显示对图片进行了等比缩放和平移(图片本身有点大)
        mPolyMatrix.postScale(0.26f, 0.26f);
        mPolyMatrix.postTranslate(0,200);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        // 根据Matrix绘制一个变换后的图片
        canvas.drawBitmap(mBitmap, mPolyMatrix, null);
    }
}
```

