---
title: memory_problem
date: 2017-10-10 15:09:28
categories: android
tags: 内存
---

## sdk 内存泄漏

1.Android InputMethodManager

2.MTK Webview的内存泄露(org.chromium.android_webview.AwPasswordHandler.java中private static AwPasswordHandler sInstance = null导致的内存泄露)。

<!-- more -->

## 常见内存泄漏
 
1.在Activity中写一些内部类，并且这些内部类具有生命周期过长的现象

2.非静态内部类导致的内存泄露，比如Handler（持有this），解决方法是将内部类写成静态内部类，在静态内部类中使用软引用/弱引用持有外部类的实例

3.资源对象没有关闭，比如数据库操作中得Cursor,IO操作的对象

4.调用了registerReceiver注册广播后未调用unregisterReceiver()来取消

5.调用了View.getViewTreeObserver().addOnXXXListener ,而没有调用View.getViewTreeObserver().removeXXXListener

6.Context的泄露，比如我们在单例类中使用Context对象，解决方案：使用getApplicationContext()来代替Activity的Context

## 内存检测工具

1.DDMS(Dalvik Debug Monitor Server)和MAT(Memory Analyzer Tool)工具

2.Android studio 自带的 Android Profiler

3.LeakCanary