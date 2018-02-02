---
title: Android的消息循环——线程间通信
id: 48
categories:
  - 未分类
date: 2016-09-13 14:47:26
tags:
---

<span style="font-size: large;">Android的线程可分为UI线程和工作线程，UI线程默认开启消息循环，但是创建的工作线程默认是没有消息循环 和消息队列的，如果想让该 线程具有消息队列和消息循环，需要在线程中首先调用Looper.prepare()来创建消息队列，然后调用Looper.loop()进入消息循环。 如下例所示：</span>
<pre class="java">class LooperThread extends Thread {
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
  }</pre>
Android的Looper和HandlerLooper:每一个线程都可以产生一个Looper,用来管理线程的Message，Looper对象会建立一个MessgaeQueue数据结构来存放message。

Handler:与Looper沟通的对象，可以push消息或者runnable对象到MessgaeQueue，也可以从MessageQueue得到消息。
<div>Activity是一个UI线程，运行于主线程中，Android系统在启动的时候会为Activity创建一个消息队列和消息循环（Looper）。详细实现请参考ActivityThread.java文件</div>
<div>Android应用程序进程在启动的时候，会在进程中加载ActivityThread类，并且执行这个类的main函数，应用程序的消息循环过程就是在这个main函数里面实现的</div>
<pre class="java"> public static void main(String[] args) {
        SamplingProfilerIntegration.start();

        // CloseGuard defaults to true and can be quite spammy.  We
        // disable it here, but selectively enable it later (via
        // StrictMode) on debug builds, but using DropBox, not logs.
        CloseGuard.setEnabled(false);

        Environment.initForCurrentUser();

        // Set the reporter for event logging in libcore
        EventLogger.setReporter(new EventLoggingReporter());

        Security.addProvider(new AndroidKeyStoreProvider());

        Process.setArgV0("&lt;pre-initialized&gt;");

        Looper.prepareMainLooper();

        ActivityThread thread = new ActivityThread();
        thread.attach(false);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        AsyncTask.init();

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }</pre>
<span style="font-size: large;">贴一篇别人分析的源码和自己的理解
</span>
<pre class="java">//Looper类分析
public class Looper {
   //static变量，判断是否打印调试信息。
    private static final boolean DEBUG = false;
    private static final boolean localLOGV = DEBUG ? Config.LOGD : Config.LOGV;

    // sThreadLocal.get() will return null unless you've called prepare().
//线程本地存储功能的封装，TLS，thread local storage,什么意思呢？因为存储要么在栈上，例如函数内定义的内部变量。要么在堆上，例如new或者malloc出来的东西
//但是现在的系统比如Linux和windows都提供了线程本地存储空间，也就是这个存储空间是和线程相关的，一个线程内有一个内部存储空间，这样的话我把线程相关的东西就存储到
//这个线程的TLS中，就不用放在堆上而进行同步操作了。
//本人理解：线程局部变量，内部是类map存储，key为它本身，value是当前线程相关的变量副本
 private static final ThreadLocal sThreadLocal = new ThreadLocal();
//消息队列，MessageQueue，看名字就知道是个queue..
    final MessageQueue mQueue;
    volatile boolean mRun;
//和本looper相关的那个线程，初始化为null
    Thread mThread;
    private Printer mLogging = null;
//static变量，代表一个UI Process（也可能是service吧，这里默认就是UI）的主线程
    private static Looper mMainLooper = null;

     /** Initialize the current thread as a looper.
      * This gives you a chance to create handlers that then reference
      * this looper, before actually starting the loop. Be sure to call
      * {@link #loop()} after calling this method, and end it by calling
      * {@link #quit()}.
      */
//往TLS中设上这个Looper对象的，如果这个线程已经设过了ｌｏｏｐｅｒ的话就会报错
//这说明，一个线程只能设一个looper
    public static final void prepare() {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper());
    }

    /** Initialize the current thread as a looper, marking it as an application's main 
     *  looper. The main looper for your application is created by the Android environment,
     *  so you should never need to call this function yourself.
     * {@link #prepare()}
     */
 //由framework设置的UI程序的主消息循环，注意，这个主消息循环是不会主动退出的
//    
    public static final void prepareMainLooper() {
        prepare();
        setMainLooper(myLooper());
//判断主消息循环是否能退出....
//通过quit函数向looper发出退出申请
        if (Process.supportsProcesses()) {
            myLooper().mQueue.mQuitAllowed = false;
        }
    }

    private synchronized static void setMainLooper(Looper looper) {
        mMainLooper = looper;
    }

    /** Returns the application's main looper, which lives in the main thread of the application.
     */
    public synchronized static final Looper getMainLooper() {
        return mMainLooper;
    }

    /**
     *  Run the message queue in this thread. Be sure to call
     * {@link #quit()} to end the loop.
     */
//消息循环，整个程序就在这里while了。
//这个是static函数喔！
    public static final void loop() {
        Looper me = myLooper();//从该线程中取出对应的looper对象
        MessageQueue queue = me.mQueue;//取消息队列对象...
        while (true) {
            Message msg = queue.next(); // might block取消息队列中的一个待处理消息..
            //if (!me.mRun) {//是否需要退出？mRun是个volatile变量，跨线程同步的，应该是有地方设置它。
            //    break;
            //}
            if (msg != null) {
                if (msg.target == null) {
                    // No target is a magic identifier for the quit message.
                    return;
                }
                if (me.mLogging!= null) me.mLogging.println(
                        "&gt;&gt;&gt;&gt;&gt; Dispatching to " + msg.target + " "
                        + msg.callback + ": " + msg.what
                        );
                //msg.target目标handler
                msg.target.dispatchMessage(msg);
                if (me.mLogging!= null) me.mLogging.println(
                        "&lt;&lt;&lt;&lt;&lt; Finished to    " + msg.target + " "
                        + msg.callback);
                msg.recycle();
            }
        }
    }

    /**
     * Return the Looper object associated with the current thread.  Returns
     * null if the calling thread is not associated with a Looper.
     */
//返回和线程相关的looper
    public static final Looper myLooper() {
        return (Looper)sThreadLocal.get();
    }

    /**
     * Control logging of messages as they are processed by this Looper.  If
     * enabled, a log message will be written to &lt;var&gt;printer&lt;/var&gt; 
     * at the beginning and ending of each message dispatch, identifying the
     * target Handler and message contents.
     * 
     * @param printer A Printer object that will receive log messages, or
     * null to disable message logging.
     */
//设置调试输出对象，looper循环的时候会打印相关信息，用来调试用最好了。
    public void setMessageLogging(Printer printer) {
        mLogging = printer;
    }

    /**
     * Return the {@link MessageQueue} object associated with the current
     * thread.  This must be called from a thread running a Looper, or a
     * NullPointerException will be thrown.
     */
    public static final MessageQueue myQueue() {
        return myLooper().mQueue;
    }
//创建一个新的looper对象，
//内部分配一个消息队列，设置mRun为true
    private Looper() {
        mQueue = new MessageQueue();
        mRun = true;
        mThread = Thread.currentThread();
    }

    public void quit() {
        Message msg = Message.obtain();
        // NOTE: By enqueueing directly into the message queue, the
        // message is left with a null target.  This is how we know it is
        // a quit message.
        mQueue.enqueueMessage(msg, 0);
    }

    /**
     * Return the Thread associated with this Looper.
     */
    public Thread getThread() {
        return mThread;
    }
    //后面就简单了，打印，异常定义等。
    public void dump(Printer pw, String prefix) {
        pw.println(prefix + this);
        pw.println(prefix + "mRun=" + mRun);
        pw.println(prefix + "mThread=" + mThread);
        pw.println(prefix + "mQueue=" + ((mQueue != null) ? mQueue : "(null"));
        if (mQueue != null) {
            synchronized (mQueue) {
                Message msg = mQueue.mMessages;
                int n = 0;
                while (msg != null) {
                    pw.println(prefix + "  Message " + n + ": " + msg);
                    n++;
                    msg = msg.next;
                }
                pw.println(prefix + "(Total messages: " + n + ")");
            }
        }
    }

    public String toString() {
        return "Looper{"
            + Integer.toHexString(System.identityHashCode(this))
            + "}";
    }

    static class HandlerException extends Exception {

        HandlerException(Message message, Throwable cause) {
            super(createMessage(cause), cause);
        }

        static String createMessage(Throwable cause) {
            String causeMsg = cause.getMessage();
            if (causeMsg == null) {
                causeMsg = cause.toString();
            }
            return causeMsg;
        }
    }
}</pre>
&nbsp;
<pre class="java">class Handler{
..........
//handler默认构造函数
public Handler() {
//这个if是干嘛用的暂时还不明白，涉及到java的深层次的内容了应该
        if (FIND_POTENTIAL_LEAKS) {
            final Class&lt;? extends Handler&gt; klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &amp;&amp;
                    (klass.getModifiers() &amp; Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }
//获取本线程的looper对象
//如果本线程还没有设置looper，这回抛异常
        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
//无耻啊，直接把looper的queue和自己的queue搞成一个了
//这样的话，我通过handler的封装机制加消息的话，就相当于直接加到了looper的消息队列中去了
        mQueue = mLooper.mQueue;
        mCallback = null;
    }
//还有好几种构造函数，一个是带callback的，一个是带looper的
//由外部设置looper
    public Handler(Looper looper) {
        mLooper = looper;
        mQueue = looper.mQueue;
        mCallback = null;
    }
// 带callback的，一个handler可以设置一个callback。如果有callback的话，
//凡是发到通过这个handler发送的消息，都有callback处理，相当于一个总的集中处理
//待会看dispatchMessage的时候再分析
public Handler(Looper looper, Callback callback) {
        mLooper = looper;
        mQueue = looper.mQueue;
        mCallback = callback;
    }
//
//通过handler发送消息
//调用了内部的一个sendMessageDelayed
public final boolean sendMessage(Message msg)
    {
        return sendMessageDelayed(msg, 0);
    }
//FT，又封装了一层，这回是调用sendMessageAtTime了
//因为延时时间是基于当前调用时间的，所以需要获得绝对时间传递给sendMessageAtTime
public final boolean sendMessageDelayed(Message msg, long delayMillis)
    {
        if (delayMillis &lt; 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }

public boolean sendMessageAtTime(Message msg, long uptimeMillis)
    {
        boolean sent = false;
        MessageQueue queue = mQueue;
        if (queue != null) {
//把消息的target设置为自己，然后加入到消息队列中
//对于队列这种数据结构来说，操作比较简单了
            msg.target = this;
            sent = queue.enqueueMessage(msg, uptimeMillis);
        }
        else {
            RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
        }
        return sent;
    }
//还记得looper中的那个消息循环处理吗
//从消息队列中得到一个消息后，会调用它的target的dispatchMesage函数
//message的target已经设置为handler了，所以
//最后会转到handler的msg处理上来
//这里有个处理流程的问题
public void dispatchMessage(Message msg) {
//如果msg本身设置了callback，则直接交给这个callback处理了
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
//如果该handler的callback有的话，则交给这个callback处理了---相当于集中处理
          if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
           }
//否则交给派生处理,基类默认处理是什么都不干
            handleMessage(msg);
        }
    }
..........
}</pre>
总结：android线程间通信可通过handler来解决，工作线程向UI线程发送消 息，handler.sendMessage(Message msg)或者post(Runnable obj)一个runnable对象，UI线程向工作线程发送消息，首先要手动开启工作线程的消息循环，handler必须在工作线程的run方法中创建， 然后由UI线程发送消息或runnable对象到工作线程。