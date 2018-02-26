---
title: 安卓线程通信
date: 2018-01-07 14:30:00
categories: android
tags: 多线程
---

## 线程和进程的区别

进程是系统资源分配时的一个基本单位，拥有一个完整的虚拟空间地址。系统在运行的时候会为每个进程分配不同的内存区域。
线程是进程的一个实体，是CPU 调度和分配的基本单位，其本身不拥有系统资源，只含有程序计数器、寄存器和栈等一些运行时必不可少的基本资源。同属一个进程的线程共享进程中的全部资源。
一个进程可以包括多个线程

<!-- more -->

## 线程基本通信方式

1.使用管道流Pipes
2.共享变量 （多个线程共享同一份内存，就是说，一个变量可以同时被多个线程所访问。这里要特别注意同步和原子操作的问题）
3.使用Hander和Message
4.AsyncTask

### pipes

“管道”是java.io包的一部分。它是Java的特性，而不是Android特有的。一条“管道”为两个线程建立一个单向的通道。生产者负责写数据，消费者负责读取数据。

``` bash
public class PipeExampleActivity extends Activity {

    private static final String TAG = "PipeExampleActivity";
    private EditText editText;

    PipedReader r;
    PipedWriter w;

    private Thread workerThread;

    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        r = new PipedReader();
        w = new PipedWriter();

        try {
            w.connect(r);
        } catch (IOException e) {
            e.printStackTrace();
        }

        setContentView(R.layout.activity_pipe);
        editText = (EditText) findViewById(R.id.edit_text);
        editText.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence charSequence, int start, int count, int after) {
            }

            @Override
            public void onTextChanged(CharSequence charSequence, int start, int before, int count) {
                try {
                    if(count > before) {
                        w.write(charSequence.subSequence(before, count).toString());
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

            @Override
            public void afterTextChanged(Editable editable) {
            }
        });

        workerThread = new Thread(new TextHandlerTask(r));
        workerThread.start();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        workerThread.interrupt();
        try {
            r.close();
            w.close();
        } catch (IOException e) {
        }
    }

    private static class TextHandlerTask implements Runnable {
        private final PipedReader reader;

        public TextHandlerTask(PipedReader reader){
            this.reader = reader;
        }
        @Override
        public void run() {
            while(!Thread.currentThread().isInterrupted()){
                try {
                    int i;
                    while((i = reader.read()) != -1){
                        char c = (char) i;
                        
                        Log.d(TAG, "char = " + c);
                    }

                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
在这个例子中，对EditText设置一个TextWatcher监听，一旦EditText的内容发生改变，就向“管道”中输入字符，它就是所谓的生产者。
同时，有一个工作线程负责从管道中读取字符，它就是所谓的消费者。这样，就实现了UI线程和工作线程之间的数据通信
```

### Hander

定义 ：handler是更新UI界面的机制，也是消息处理的机制,我们可以发送消息，也可以处理消息;每一个线程里可含有一个Looper对象以及一个MessageQueue数据结构
原理 ：
1、handler封装消息的发送（主要包括消息发送给谁）
2、Looper——消息封装的载体。（1）内部包含一个MessageQueue，所有的Handler发送的消息都走向这个消息队列；（2）Looper.Looper方法，就是一个死循环，不断地从MessageQueue取消息，如果有消息就处理消息，没有消息就阻塞。
3、MessageQueue，一个消息队列，添加消息，处理消息
4、handler内部与Looper关联，handler->Looper->MessageQueue,handler发送消息就是向MessageQueue队列发送消息。
总结：handler负责发送消息，Looper负责接收handler发送的消息，并把消息回传给handler自己。MessageQueue存储消息的容器

``` bash
class LooperThread extends Thread {
        public Handler mHandler;
  
        public void run() {
            Looper.prepare();
  
            mHandler = new Handler() {
                public void handleMessage(Message msg) {
                    // process incoming messages here
                }
            };
  
            Looper.loop();
        }
```

### AsyncTask

1.execute(Params... params)，执行一个异步任务，需要我们在代码中调用此方法，触发异步任务的执行。
2.onPreExecute()，在execute(Params... params)被调用后立即执行，一般用来在执行后台任务前对UI做一些标记。
3.doInBackground(Params... params)，在onPreExecute()完成后立即执行，用于执行较为费时的操作，此方法将接收输入参数和返回计算结果。在执行过程中可以调用publishProgress(Progress... values)来更新进度信息。
4.onProgressUpdate(Progress... values)，在调用publishProgress(Progress... values)时，此方法被执行，直接将进度信息更新到UI组件上。
5.onPostExecute(Result result)，当后台操作结束时，此方法将会被调用，计算结果将做为参数传递到此方法中，直接将结果显示到UI组件上。

