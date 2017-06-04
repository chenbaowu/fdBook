---
title: 线程池ThreadPoolExecutor
date: 2017-05-10 16:58:30
categories: android
tags: 多线程
---

1.ThreadPoolExecutor参数

jdk自身带有线程池的实现类ThreadPoolExecutor，使用ThreadPoolExecutor，了解其每个参数的意义是必不可少的。 
ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory, handler) 
corePoolSize: 核心线程数，能够同时执行的任务数量； 
maximumPoolSize：除去缓冲队列中等待的任务，最大能容纳的任务数（其实是包括了核心线程池数量）； 
keepAliveTime：超出workQueue的等待任务的存活时间，就是指maximumPoolSize里面的等待任务的存活时间； 
unit：时间单位； 
workQueue:阻塞等待线程的队列，一般使用new LinkedBlockingQueue()这个，如果不指定容量，会一直往里边添加，没有限制,workQueue永远不会满； 
threadFactory：创建线程的工厂，使用系统默认的类； 
handler：当任务数超过maximumPoolSize时，对任务的处理策略，默认策略是拒绝添加；

<!-- more -->

2.阻塞队列

队列用于排队，避免一瞬间出现大量请求的问题。 
阻塞队列分为 有限队列（SynchronousQueue、ArrayBlockingQueue）和 无限队列（LinkedBloackingQueue）。

3.执行流程

当线程数小于corePoolSize时，每添加一个任务，则立即开启线程执行；当corePoolSize满的时候，后面添加的任务将放入缓冲队列workQueue等待；当workQueue也满的时候，看是否超过maximumPoolSize线程数，如果超过，默认拒绝执行。 
下面我们看个例子：假如corePoolSize=2，maximumPoolSize=3，workQueue容量为8;最开始，执行的任务A，B，此时corePoolSize已用完，再次执行任务C，则C将被放入缓冲队列workQueue中等待着，如果后来又添加了7个任务，此时workQueue已满，则后面再来的任务将会和maximumPoolSize比较，由于maximumPoolSize为3，所以只能容纳1个了，因为有2个在corePoolSize中运行了，所以后面来的任务默认都会被拒绝。

4.代码
``` bash
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        /**
         * 创建九个任务
         */
        for (int i = 0; i < 9; i++) {
            ThreadPoolManager.getInstance().execute(new DownloadTask(i));
        }
    }
    /**
     * 模仿下载任务，实现Runnable
     */
    class DownloadTask implements Runnable{
        private int num;
        public DownloadTask(int num) {
            super();
            this.num = num;
            Log.d("JAVA", "task - "+num + " 等待中...");
        }
        @Override
        public void run() {
            Log.d("JAVA", "task - "+num + " 开始执行了...开始执行了...");
            SystemClock.sleep(5000); //模拟延时执行的时间
            Log.e("JAVA", "task - "+num + " 结束了...");
        }
    }
}
```
运行结果
![运行结果](http://img1.ph.126.net/nPxfEyrfw0dgATkDbB6iLA==/6632146386609030268.jpg)