## Android线程的创建与销毁   
   
### 摘要：

在Android开发中经常会使用到线程，一想到线程，很多同学就立即使用new Thread(){...}.start()这样的方式。这样如果在一个Activity中多次调用上面的代码，那么将创建多个匿名线程，程序运行的越久可能会越来越慢。因此，需要一个Handler来启动一个线程，以及删除一个线程，保证线程不会重复的创建。   
  
### 正文：

1、创建Handler的一般方式 

一般会使用Handler handler = new Handler(){...}创建。  
这样创建的handler是在主线程即UI线程下的Handler，即这个Handler是与UI线程下的默认Looper绑定的。  
Looper是用于实现消息队列和消息循环机制的。   
因此，如果是默认创建Handler那么如果线程是做一些耗时操作如网络获取数据等操作，这样创建Handler是不行的。  
  
  
2、使用HandlerThread 

HandlerThread实际上就一个Thread，只不过它比普通的Thread多了一个Looper。  
我们可以使用下面的例子创建Handler :  
  
	HandlerThread thread = new HandlerThread("MyHandlerThread"); 
	thread.start(); 
	mHandler = new Handler(thread.getLooper()); 
	mHandler.post(mBackgroundRunnable);    
   
创建HandlerThread时要把它启动了，即调用start()方法。   
然后创建Handler时将HandlerThread中的looper对象传入。   
那么这个mHandler对象就是与HandlerThread这个线程绑定了（这时就不再是与UI线程绑定了，这样它处理耗时操作将不会阻塞UI）。    
最后把实现耗时操作的线程post到mHandler的消息队列里面。   
注意的是，mBackgroundRunnable这个线程并没有启动，因为没有调用start()方法。   
   
### 完整代码   
   
	public class MainActivity extends Activity implements OnClickListener{ 
    	public static final String TAG = "MainActivity"; 
    	private Handler mHandler; 
    	private boolean mRunning = false; 
    	private Button mBtn; 
    
    	@Override 
    	protected void onCreate(Bundle savedInstanceState) { 
    	    super.onCreate(savedInstanceState); 
    	    setContentView(R.layout.activity_main); 
    	    
    	    HandlerThread thread = new HandlerThread("MyHandlerThread"); 
    	    thread.start();//创建一个HandlerThread并启动它 
			//使用HandlerThread的looper对象创建Handler，如果使用默认的构造方法，很有可能阻塞UI线程 
    	    mHandler = new Handler(thread.getLooper());

    	    mHandler.post(mBackgroundRunnable);//将线程post到Handler中 

    	    mBtn = (Button)findViewById(R.id.button); 
    	    mBtn.setOnClickListener(this); 
    	} 
    
    	@Override 
    	protected void onResume() { 
    	    super.onResume(); 
    	    mRunning = true; 
    	} 
    
    	@Override 
    	protected void onStop() { 
    	    super.onStop(); 
    	    mRunning = false; 
    	} 
    
    	@Override 
    	public boolean onCreateOptionsMenu(Menu menu) { 
    	    // Inflate the menu; this adds items to the action bar if it is present. 
    	    getMenuInflater().inflate(R.menu.main, menu); 
    	    return true; 
    	} 
    
    	//实现耗时操作的线程 
    	Runnable mBackgroundRunnable = new Runnable() { 

    	    @Override 
    	    public void run() { 
    	        //----------模拟耗时的操作，开始--------------- 
    	        while(mRunning){ 
    	            Log.i(TAG, "thread running!"); 
    	            try { 
    	                Thread.sleep(200); 
    	            } catch (InterruptedException e) { 
    	                e.printStackTrace(); 
    	            } 
    	        } 
    	        //----------模拟耗时的操作，结束--------------- 
    	    } 
    	}; 
    
    	@Override 
    	protected void onDestroy() { 
    	    super.onDestroy(); 
    	    //销毁线程 
    	    mHandler.removeCallbacks(mBackgroundRunnable); 
    	} 
    
    	@Override 
    	public void onClick(View v) { 
    	    Toast.makeText(getApplication(), "click the button!!!", Toast.LENGTH_SHORT).show(); 
    	} 
	}   
   
如果在onCreate()方法中里面没有使用HandlerThread而是在直接使用Handler的默认构造方法来创建Handler，那么mBackgroundRunnable将会阻塞UI线程。    
   
---   
   
## 三大框架：Struts+Hibernate+Spring

* Struts主要负责表示层的显示
* Spring利用它的IOC和AOP来处理控制业务（负责对数据库的操作）
* Hibernate主要是数据持久化到数据库
  
再用jsp的servlet做网页开发的时候有个 web.xml的映射文件，里面有一个mapping的标签就是用来做文件映射的。当你在浏览器上输入URL得知的时候，文件就会根据你写的名称对应到一 个JAVA文件，根据java文件里编写的内容显示在浏览器上，就是一个网页。   
   
### 一、 Struts框架

struts是开源软件。使用Struts的目的是为了帮助我们减少在运用MVC设计模型来开发Web应用的时间。如果我们想混合使用Servlets和JSP的优点来建立可扩展的应用，struts是一个不错的选择。

1．流程：服务器启动后，根据web.xml加载ActionServlet读取struts-config.xml文件内容到内存。

2．架构：Struts对Model，View和Controller都提供了对应的组件。ActionServlet，这个类是Struts的核心控制器，负责拦截来自用户的请求。

Model部分：由JavaBean组成，ActionForm用于封装用户的请求参数，封装成ActionForm对象，该对象被ActionServlet转发给 Action，Action根据ActionFrom里面的请求参数处理用户的请求。JavaBean则封装了底层的业务逻辑，包括数据库访问等。

View部分：该部分采用JSP实现。Struts提供了丰富的标签库，通过标签库可以减少脚本的使用，自定义的标签库可以实现与Model的有效交互，并增加了现实功能。对应上图的JSP部分。

Controller组件：Controller组件有两个部分组成——系统核心 控制器，业务逻辑控制器。 　　系统核心控制器，对应上图的ActionServlet。该控制器由Struts框架提供，继承HttpServlet 类，因此可以配置成标注的Servlet。该控制器负责拦截所有的HTTP请求，然后根据用户请求决定是否要转给业务逻辑控制器。业务逻辑控制器，负责处 理用户请求，本身不具备处理能力，而是调用Model来完成处理。对应Action部分。   
   
### 二、 Spring框架   
   
Spring是一个解决了许多在J2EE开发中常见的的问题的强大框架。 Springle提供了管理业务对象的一致方法并且鼓励了注入对接口编程而不是对类变成的好习惯。Spring的架构基础是基于使用JavaBean属性 的Inversion of Control 容器。然而Spring在使用IoC容器作为构建玩关注所有架构层层的完整解决方案方面是独一无二的。Spring提供了唯一的数据管理 抽象包括简单和有效率的JDBC框架，极大的改进了效率并且减少了可能的错误。Spring的数据访问架构还集成了Hibernate和其他O/R mapping 解决方案。   
   
### 三、 Hibernate框架   
    
Hibernate 是一个开源代码的对象关系映射框架，对JDBC惊醒了费城轻量级的 的对象封装，使得Java程序员可以随心所欲的使用对象变成思维来操作数据库。Hebernate可以应用在任何使用JDBC的场合，既可以在java的 客户端程序使用，也可以在Servlet/JSP的Web应用中使用最具革命意义的事，Hibernate可以在应用EJB的J2EE架构中取代CMP, 完成数据持久化的重任

Hibernate的核心接口一共有5个，分别为:Session、 SessionFactory、Transaction、Query和Configuration。这5个核心接口在任何开发中都会用到。通过这些接口， 不仅可以对持久化对象进行存取，还能够进行事务控制。下面对这五个核心接口分别加以介绍。   
   
1．Session接口：负责执行被持久化对象的CRUD操作(CRUD的任务是完成与 数据库的交流，包含了很多常见的SQL语句。)。但需要注意的是Session对象是非线程安全的。同时，Hibernate的session不同于 JSP应用中的HttpSession。这里当使用session这个术语时，其实指的是Hibernate中的session，而以后会将 HttpSession对象称为用户session。

2．SessionFactory接口：负责初始化Hibernate。它充当数据存储 源的代理，并负责创建Session对象。这里用到了工厂模式。需要注意的是SessionFactory并不是轻量级的，因为一般情况下，一个项目通常 只需要一个SessionFactory就够，当需要操作多个数据库时，可以为每个数据库指定一个SessionFactory。

3．Configuration接口：负责配置并启动Hibernate，创建SessionFactory对象。在Hibernate的启动的过程中，Configuration类的实例首先定位映射文档位置、读取配置，然后创建SessionFactory对象。

4．Transaction接口：负责事务相关的操作。它是可选的，开发人员也可以设计编写自己的底层事务处理代码。

5．Query和Criteria接口：负责执行各种数据库查询。它可以使用HQL语言或SQL语句两种表达方式。    
     
---    
   
