---
layout: post
comments: true
title: Android Multi Threading: Handler is not a thread
tags: [Code, threading, Android]
---

One mistake I see Android developers make is that they consider making a `handler` equivalent to starting a new thread. It is not uncommon to come accross questions like (these)[https://stackoverflow.com/questions/6964011/handler-vs-asynctask-vs-thread/9800870#9800870]. 

In this post I'll explain what a Handler is and how to use it.

`Handler` is unique to Android and is not part of Java's native multi-threading toolkit. It has one job and that is to enable a thread to communicate with another thread. What do we mean by communicate with another thread? Two things - 

1. We can send a `Message` object to the Handler and have the `Handler` extract data from it in its `handleMessage` method.
2. We can execute a block of code wrapped around in a `Runnable` on the other thread.

Suppose we have a thread `A` as the main thread (UI thread) and `B` as a worker thread. Now, we want to log some text in `B` that is sent by thread `A`

If this problem was limited to using Java's native toolkit we would have to make use of `wait()` and `notify()` or `Pipes`. But since we have the support of Android we can employ a `Handler`. We make a class `TestHandler` which extends to `Handler`. `handleMessage` will recieve the message as an argument and we can log it as needed.

    public class TestHandler extends Handler {

        Looper looper;

        TestHandler(Looper myLooper) {
            super(myLooper);

            this.looper = myLooper;
        }

        public void handleMessage(Message msg) {

            String txt = (String) msg.obj;
            Log.i("Message", txt);
        }
    }

Now if I were to re-read my opening lines I would know that just making a `Handler` would not mean that the code in `handleMessage` will be executed in a new thread. To insure that we need to make a new thread, attach a `looper` to it and then pass the instance of the looper to the `Handler`.

## What is a looper?

A java `thread` is linear and is killed off once it executes the code in its `run()` method. Kind of like a `IntentService` in Android (but not exactly like it).

A looper is a mechanism (again, exclusive to Android) which ensures that a thread remains active, ready to receive messages and does not die after executing a block of code. Think `while(true) {...}` loop.

Attaching a looper to a thread - 

    public class ThreadB extends Thread {

        Looper myLooper = null;

        @Override
        public void run() {
            super.run();

            if (Looper.myLooper() == null)
                Looper.prepare();

            myLooper = Looper.myLooper();

            Looper.loop();
        }
    }

## Why pass instance of looper into the Handler?

Because the `Handler` needs to know which thread to execute its `handleMessage` or `runnable` code block on. **By default, a handler will attach itself to the looper of the thread it has been created in.** So if you make a Handler in the main thread without passing a looper to it, it will automatically attach itself to the main thread and hence execute all the code on the main thread (Giving an illusion of multi-threaded execution to developers unaware its behaviour). The main thread is the only thread which by default has a looper attached to it; the said looper can be obtained by calling `Looper.getMainLooper()`.

**Try this:** Try creating a handler inside a worker thread which does not have looper associated to it.

## Thread B -

    public class ThreadB extends Thread {

        Looper myLooper = null;

        @Override
        public void run() {
            super.run();

            if (Looper.myLooper() == null)
                Looper.prepare();

            myLooper = Looper.myLooper();

            Looper.loop();
        }

        void quitThread() {
            myLooper.quitSafely();
        }

        void passMessage(Message msg) {
            TestHandler testHandler = new TestHandler(myLooper);
            testHandler.handleMessage(msg);
        }

        void doRunnableWork(Runnable runnable) {
            TestHandler testHandler = new TestHandler(myLooper);
            testHandler.post(runnable);
        }
    }

## Thread A (UI Thread) -

    public class MainActivity extends AppCompatActivity {

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);

            ThreadB threadB = new ThreadB();
            threadB.start();

            Message msg = new Message();
            msg.obj = "MESSAGE";

            // This will log the message in thread B
            threadB.doWork(msg); 

            // This will execute the block of code 
            // inside the run() method. This too will happen
            // in thread B.
            final Runnable runnable = new Runnable() {
                @Override
                public void run() {

                    Log.i("Runnable", "working");
                }
            };

            threadB.doRunnableWork(runnable);

        }
    }

You can confirm which thread all the code is being executed on by logging thread ids.

# Summery

1. Just making a handler in a main thread will not ensure that the code will be executed in a new thread.
2. A handler works alongside a looper and a thread. 
3. Looper ensures that the thread does not die. 
4. For a handler to execute code in the right thread the handler needs to be either created in the said thread or the looper of the said thread needs to be associated with the handler at the time of its creation.









    
