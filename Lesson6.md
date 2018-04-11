## HTTP长连接和短连接原理浅析   
   
### 1. HTTP协议与TCP/IP协议的关系

HTTP的长连接和短连接本质上是TCP长连接和短连接。HTTP属于应用层协议，在传输层使用TCP协议，在网络层使用IP协议。IP协议主要解决网络路由和寻址问题，TCP协议主要解决如何在IP层之上可靠的传递数据包，使在网络上的另一端收到发端发出的所有包，并且顺序与发出顺序一致。TCP有可靠，面向连接的特点。

### 2. 如何理解HTTP协议是无状态的

HTTP协议是无状态的，指的是协议对于事务处理没有记忆能力，服务器不知道客户端是什么状态。也就是说，打开一个服务器上的网页和你之前打开这个服务器上的网页之间没有任何联系。HTTP是一个无状态的面向连接的协议，无状态不代表HTTP不能保持TCP连接，更不能代表HTTP使用的是UDP协议（无连接）。

### 3. 什么是长连接、短连接？

在HTTP/1.0中，默认使用的是短连接。也就是说，浏览器和服务器每进行一次HTTP操作，就建立一次连接，但任务结束就中断连接。如果客户端浏览器访问的某个HTML或其他类型的 Web页中包含有其他的Web资源，如JavaScript文件、图像文件、CSS文件等；当浏览器每遇到这样一个Web资源，就会建立一个HTTP会话。

但从 HTTP/1.1起，默认使用长连接，用以保持连接特性。使用长连接的HTTP协议，会在响应头有加入这行代码：  

	Connection:keep-alive  
  
在使用长连接的情况下，当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的 TCP连接不会关闭，如果客户端再次访问这个服务器上的网页，会继续使用这一条已经建立的连接。Keep-Alive不会永久保持连接，它有一个保持时间，可以在不同的服务器软件（如Apache）中设定这个时间。实现长连接要客户端和服务端都支持长连接。

HTTP协议的长连接和短连接，实质上是TCP协议的长连接和短连接。    
  
---
    
## Fragment生命周期详解    
    
![](https://i.imgur.com/4MAIfzP.jpg)   
   
### 流程： 
onAttach()   
作用：fragment已经关联到activity  

    这个是 回调函数
    @Override
    public void onAttach(Activity activity) {
            super.onAttach(activity);
            Log.i("onAttach_Fragment");
    }
    这个时候 activity已经传进来了
    获得activity的传递的值
    就可以进行 与activity的通信里

    当然也可以使用getActivity(),前提是这个fragment已经和宿主的activity关联，并且没有脱离
    他只调用一次。    
    
### onCreate() 
系统创建fragment的时候回调他，在他里面实例化一些变量   
这些个变量主要是：当你 暂停 停止的时候 你想保持的数据，如果我们要为fragment启动一个后台线程，可以考虑将代码放于此处。  
参数是：Bundle savedInstance, 用于保存 Fragment 参数, Fragement 也可以 重写 onSaveInstanceState(BundleoutState)方法,保存Fragement状态;可以用于文件保护，他只调用一次。    
    
### onCreateView()

    第一次使用的时候 fragment会在这上面画一个layout出来，
    为了可以画控件 要返回一个 布局的view，也可以返回null

    当系统用到fragment的时候 fragment就要返回他的view，越快越好
    ，所以尽量在这里不要做耗时操作，比如从数据库加载大量数据显示listview，
    当然线程还是可以的。

    给当前的fragment绘制ui布局，可以使用线程更新UI
    说白了就是加载fragment的布局的。
    这里一般都先判断是否为null    
    
	if(text==null){
            Bundle args=getArguments();
            text=args.getString("text");
        }
        if (view == null) {
            view = inflater.inflate(R.layout.hello, null);
        }
这样进行各判断省得每次都要加载，减少资源消耗   
   
### onActivityCreated()

    当Activity中的onCreate方法执行完后调用。    

    注意了：
    从这句官方的话可以看出：当执行onActivityCreated()的时候 activity的
    onCreate才刚完成。
    所以在onActivityCreated()调用之前 activity的onCreate可能还没有完成，
    所以不能再onCreateView()中进行 与activity有交互的UI操作，UI交互操作可以砸
    onActivityCreated()里面进行。
    所以呢：这个方法主要是初始化那些你需要你的父Activity或者Fragment的UI已经被完
    整初始化才能初始化的元素。
    如果在onCreateView里面初始化空间 会慢很多，比如listview等     
    
### onStart()

    和activity一致 启动, Fragement 启动时回调, 此时Fragement可见;
### onResume()

    和activity一致  在activity中运行是可见的
    激活, Fragement 进入前台, 可获取焦点时激活;
### onPause()

    和activity一致  其他的activity获得焦点，这个仍然可见
    第一次调用的时候，指的是 用户 离开这个fragment（并不是被销毁）
    通常用于 用户的提交（可能用户离开后不会回来了）    
   
### onStop()

    和activity一致
    fragment不可见的， 可能情况：activity被stopped了OR fragment被移除但被
    加入到回退栈中
    一个stopped的fragment仍然是活着的如果长时间不用也会被移除    
   
### onDestroyView()

    Fragment中的布局被移除时调用。
    表示fragemnt销毁相关联的UI布局
    清除所有跟视图相关的资源

    以前以为这里没什么用处其实 大有文章可做，
    相信大家都用过ViewPager+Fragment，由于ViewPager的缓存机制，每次都会加载3
    页。
    例如：有四个 fragment 当滑动到第四页的时候 第一页执行onDestroyView(),但没有
    执行onDestroy。他依然和activity关联。当在滑动到第一页的时候又执行了 
    onCreateView()。 生命周期可以自己试一下。
    那么问题来了。会出现重复加载view的局面，所以这么做(下面是先人的代码)    
    
	@Override
    public void onDestroyView() {
        Log.i("onDestroyView_Fragment");
        if(view!=null){
                        ((ViewGroup)view.getParent()).removeView(view);
        }
        super.onDestroyView();
    }   
    
### onDestroy()

    销毁fragment对象
    跟activity类似了。
### onDetach()

    Fragment和Activity解除关联的时候调用。
    脱离activity    
   
### 下面贴一下 activity和fragment同时运行时候的 生命周期   
   
![](https://i.imgur.com/8SpRSnF.jpg)   
   
可以看出 当现实fragment的时候都先执行activity方法，当销毁的时候都是现执行 fragment的方法，这样更好理解fragment是嵌套在activity中。