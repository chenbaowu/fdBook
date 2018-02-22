---
title: optimization_ui
date: 2017-09-03 21:33:33
tags: 优化
---

## android ui 优化

Android UI渲染分为3个过程，分别是测量、布局和绘制，这3个都是深度优先准则，父UI在子UI之前绘制，再按顺序绘制兄弟UI。

android每16ms发一次VSYNC信号触发UI渲染，只要16ms能达到一个流畅的画面，用户就不会感到卡顿。1000 / 16 ≈60Hz。

<!-- more -->

渲染分为CPU部分与GPU部分。CPU部分包括测量、布局、记录和执行。GPU部分需要完成光栅化，计算每一个像素点的值。

Android的显示过程包含两个部分，分别是应用侧绘制和系统侧渲染；采用两个机制，分别是进程侧通信机制和显示刷新机制。

绘制模型分为基于软件的绘制模型和基于硬件加速的绘制模型。

在基于软件的绘制模型下，CPU主导绘制，视图按照两个步骤绘制，分别是让View层次结构失效，绘制View层次结构。但是会有两个缺点，绘制了不需要重绘的区域，掩盖了一些应用的bug。

在基于硬件加速的绘制模型中，GPU主导绘图，按照3个步骤绘制：1）让View层次结构失效； 2） 记录更新列表； 3） 绘制显示列表。

硬件加速的3个缺陷：
1. 兼容新，部分绘制方式不支持或者不完全支持硬件方式；
2. 内存消耗，OPENGL API 调用就会占用8MB，而实际上会占用更多内存。
3. 电量消耗，GPU耗电较多。

显示的两个机制，一般采用双缓冲技术，意味着要采用两个缓冲区，一个称为Front Buffer, 另一个称谓Back Buffer，UI现在Back Buffer中绘制，再和Front Buffer进行交换，最多就是三缓冲了，缓冲并不是越多越好。

引发掉帧的原因：
1）大量的重绘
2）过度绘制，在在绘制用户看不到的对象上花太多时间
3）有一大堆重复动画重复了一遍又一遍，消耗CPU、GPU资源
4）频繁地触发垃圾回收
5）ViewGroup树太深
所有消耗资源的操作，如IO操作、网络操作、SQL操作，列表刷新等，都应该放在后台处理，而不是占用UI线程，以保证UI的流畅性。

界面开发优化的Advice：
1. 优化布局的结构
1）避免复杂的View层次
2）尽量避免在顶层视图中使用RelativeLayout，尽量使用LinearLayout和FrameLayout
3）布局层级一样的情况下，用LinearLayout代替RelativeLayout
4）在复杂层级上，使用RelativeLay而非LinearLayout来尽量减少层级
5）不使用AbsoluteLayout
6）尽量比曼使用LayoutWeight
7）合理的布局应该是宽而浅，而非窄而深
2. 优化处理逻辑
1）按需载入视图，某些不怎么重用的耗资源视图，等到需要的时候再加载
2）避免在UI线程中进行耗时操作
3. 使用Android自带的性能测试工具进行优化

RelativeLayout与LinearLayout性能比较
1） RelativeLayout会让子View调用两次Measure()。LinearLayout在有layoutWeight是调用两次onMeasure（），没有时调用一次onMeasure（）
2） RelativeLayout的子View如果与父View的高度不同，会引发效率问题，当布局很复杂的时候，问题会更严重，因此尽量用padding代替margin
3） 在相同的层级下，用LinearLayout或者FrameLayout代替RelativeLayout
4） 在复杂布局下，用RelativeLayout减少布局层级