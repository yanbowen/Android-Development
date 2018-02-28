# Android 开发艺术探索  
---
## android的消息机制——Handler机制  
为了避免ANR，我们会通常把 耗时操作放在子线程里面去执行，因为子线程不能更新UI，所以当子线程需要更新的UI的时候就需要借助到安卓的消息机制，也就是Handler机制了。  
  
>注意：在安卓的世界里面，当 子线程 在执行耗时操作的时候，不是说你的主线程就阻塞在那里等待子线程的完成——也不是调用 Thread.wait()或是Thread.sleep()。安卓采取的方法是，主线程应该为子线程提供一个Handler，以便完成时能够提交给主线程。以这种方式设计你的应用程序，将能保证你的主线程保持对输入的响应性并能避免由于5秒输入事件的超时引发的ANR对话框。
  
一个程序的运行，就是一个进程的在执行，一个进程里面可以拥有很多个线程。  
  
>主线程：也叫UI线程，或称ActivityThread，用于运行四大组件和处理他们用户的交互。 ActivityThread管理应用进程的主线程的执行(相当于普通Java程序的main入口函数)，在Android系统中，在默认情况下，一个应用程序内的各个组件(如Activity、BroadcastReceiver、Service)都会在同一个进程(Process)里执行，且由此进程的主线程负责执行。
ActivityThread既要处理Activity组件的UI事件，又要处理Service后台服务工作，通常会忙不过来。为了解决此问题，主线程可以创建多个子线程来处理后台服务工作，而本身专心处理UI画面的事件。

>子线程： 用于执行耗时操作，比如 I/O操作和网络请求等。（安卓3.0以后要求耗访问网络必须在子线程种执行）更新UI的工作必须交给主线程，子线程在安卓里是不允许更新UI的。  

### 一、 基本概念
**什么是消息机制？** —— 不同线程之间的通信。

**什么安卓的消息机制**，就是 Handler 运行机制。

**安卓的消息机制有什么用？** —— 避免ANR(Application Not Responding) ，一旦发生ANR，程序就挂了，奔溃了。

**什么时候会触发ANR？（消息机制在什么时候用？）** —— 以下两个条件任意一个触发的的时候就会发生ANR

* 在activity中超过5秒的时间未能响应下一个事件  
* BroadcastReceive超过10未响应  


造成以上两点的原因有很多，比如网络请求, 大文件的读取, 耗时的计算等都会引发ANR  
  

#### Handler的简单使用
既然子线程不能更改界面，那么我们现在就借助Handler让我们更改一下界面：  
主要步骤是这样子的：  
1、new出来一个Handler对象，复写handleMessage方法  
2、在需要执行更新UI的地方 sendEmptyMessage 或者 sendMessage  
3、在handleMessage里面的switch里面case不同的常量执行相关操作  
  
	public class MainActivity extends Activity {

    private TextView mTv;
    private Handler mHandler;
    private static final int MSG_UPDATE_TEXT = 0x2001; // 更新文本  方式一用的常量
    private static final int MSG_UPDATE_WAY_TWO = 0x2002;  // 更新文本 方式二用的常量

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mHandler=new Handler(){
            @Override
            public void handleMessage(Message msg) {
                switch (msg.what){
                    case MSG_UPDATE_TEXT:
                        mTv.setText("让Handler更改界面");
                        break;

                    case MSG_UPDATE_WAY_TWO:
                        mTv.setText("让Handler更改界面方式二");
                        break;
                }
            }
        };

        mTv= (TextView) findViewById(R.id.mTv);
        mTv.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Log.d("Test", "点击文字");

                // 方式一和方式二可以达到相同的效果,就是更改界面

                // 方式一
                //mHandler.sendEmptyMessage(MSG_UPDATE_TEXT);

                // 方式二
                Message msg =Message.obtain();
                msg.what= MSG_UPDATE_WAY_TWO;
                mHandler.sendMessage(msg);

            }
        });

    }
	}  
  
### 二、消息机制的分析理解  
  
安卓的异步消息处理机制就是handler机制。

主线程，ActivityThread被创建的时候就会创建Looper  
Looper被创建的时候创建MessageQueue。  
也就是说主线程会直接或间接创建出来Looper和MessageQueue。  
具体创建解释，参考: Android异步消息处理机制完全解析，带你从源码的角度彻底理解(http://blog.csdn.net/guolin_blog/article/details/9991569)  

Handler的工作机制简单来说是这样的  

1、Handler发送消息仅仅是调用MessageQueue的enqueueMessage向插入一条信息到MessageQueue  

2、Looper不断轮询调用MeaasgaQueue的next方法  

3、如果发现message就调用handler的dispatchMessage，ldispatchMessage被成功调用，接着调用handlerMessage()  

Handler的工作机制简单来说是这样的

1、Handler发送消息仅仅是调用MessageQueue的enqueueMessage向插入一条信息到MessageQueue

2、Looper不断轮询调用MeaasgaQueue的next方法

3、如果发现message就调用handler的dispatchMessage，ldispatchMessage被成功调用，接着调用handlerMessage()
   
---  

我在学习和使用handler的时候，对与它相关的源代码进行的研究，说到handler机制，就要设计到5个类（画图），Handler、MessageQueue、Looper、Thread、还有一个Message；  

1. Message是消息，它由MessageQueue统一列队，由Handler处理。  
2. Handler是处理者，他负责发送和处理Message消息。  
3. MessageQueue指消息队列，它用来存放Handler发送过来的队列，并且按照先入先出的规则执行。   
4. Looper的作用就像抽水的水泵，它不断的从MessageQueue中去抽取Message并执行。  
5. Thread线程，是消息循环的执行场所。    


在创建Activity之前，当系统启动的时候，先加载ActivityThread这个类，在这个类的main函数中，调用Looper.prepareMainLooper()进行初始化Looper对象，然后创建主线程的handler对象，随后才创建ActivityThread对象，最后调用Looper.loop()方法，不断的进行轮询消息队列中的消息。也就是说，在ActivityThread和Activity创建之前，就已经开启了Looper的loop()方法，不断的进行轮询消息。  
  
  
我们可以画图来说明handler机制的原理：  
我们通过Message.obtain()准备消息数据之后，  
第一步是使用sendMessage()：通过Handler将消息发送给消息队列  
第二步、在发送消息的时候，使用message.target=this为handler发送的message贴上当前handler的标签  
第三步、开启HandlerThread线程，执行run方法。  
4、在HandlerThread类的run方法中开启轮询器进行轮询：调用Looper.loop()方法进行轮询消息队列的消息  
5、在消息队列MessageQueue中enqueueMessage(Message msg, long when)方法里，对消息进行入列，即依据传入的时间进行消息入列（排队）  
6、轮询消息：与此同时，Looper在不断的轮询消息队列  
7、在Looper.loop()方法中，获取到MessageQueue对象后，从中取出消息（Message msg = queue.next()），如果没有消息会堵塞  
8、分发消息：从消息队列中取出消息后，调用msg.target.dispatchMessage(msg);进行分发消息  
9、将处理好的消息分发给指定的handler处理，即调用了handler的dispatchMessage(msg)方法进行分发消息。  
10、在创建handler时，复写的handleMessage方法中进行消息的处理  
11、回收消息：在消息使用完毕后，在Looper.loop()方法中调用msg.recycle()，将消息进行回收，即将消息的所有字段恢复为初始状态。  
  
---

## 自定义控件之View原理与使用  
  
### 一、简介  
  
>Android中的每个控件都会在界面上得到一块矩形的区域，而在Android中，控件大致被分为两类，即ViewGroup 控件和View控件。ViewGroup控件作为父控件可以包含多个View控件，并管理其包含的View控件。下面分条来对View做一个简单的介绍。  

### 二、View的原理介绍  
  
1. View表示的的屏幕上的某一块矩形的区域，而且所有的View都是矩形的；  
2. 如同简介中介绍，View是不能添加子View的，而ViewGroup是可以添加子View的。ViewGroup之所以能够添加子View，是因为它实现了两个接口：ViewParent 和 ViewManager；
3. Activity之所以能加载并且控制View，是因为它包含了一个Window，所有的图形化界面都是由View显示的而Service之所以称之为没有界面的activity是因为它不包含有Window,不能够加载View；
4. 一个View有且只能有一个父View;
5. 在Android中Window对象通常由PhoneWindow来实现的，PhoneWindow将一个DecorView设置为整个应用窗口的根View,即DecorView为整个Window界面的最顶层View。也可以说DecorView将要显示的具体内容呈现在了PhoneWindow上；
6. DecorView是FrameLayout的子类,它继承了FrameLayout，即顶层的FrameLayout的实现类是Decorview，它是在phoneWindow里面创建的；
7. 顶层的FrameLayout的父view是Handler，Handler的作用除了线程之间的通讯以外，还可以跟WindowManagerService进行通讯;
8. windowManagerService是后台的一个服务，它控制并且管理者屏幕;
9. 一个应用可以有很多个window，其由windowManager来管理，而windowManager又由windowManagerService来管理;
10. 如果想要显示一个view那么他所要经历三个方法：1.测量measure, 2.布局layout, 3.绘制draw。

### View的测量/布局/绘制过程  
  
>显示一个View主要进过以下三个步骤：  

**1、Measure测量一个View的大小**  
**2、Layout摆放一个View的位置**  
**3、Draw画出View的显示内容**

其中measure和layout方法都是final的，无法重写，虽然draw不是final的，但是也不建议重写该方法。  
这三个方法都已经写好了View的逻辑，如果我们想实现自身的逻辑，而又不破坏View的工作流程，可以重写onMeasure、onLayout、onDraw方法。下面来一一介绍这三个方法。  
  

  
