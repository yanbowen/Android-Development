## Android中的IPC方式  
  
### 一、使用Bundle   
  
Activity、 Service、 Receiver 都是支持在Intent中传递Bundle数据的，由于Bundle实现了Parcelable接口，所以它可以在不同的进程间传输。**（传输的数据必须能够被序列化）**  
  
![](https://i.imgur.com/ID34SL2.jpg)   
  
### 二、使用文件共享   
  
两个进程通过读/写一个文件来交换数据。  
由于Android系统基于Linux，使得其并发读/写文件可以没有限制地进行，甚至两个线程同时对同一个文件进行写操作都是允许的，尽管这可能会出现问题。  
   
### 三、使用Messenger    
  
Messenger翻译为信使，可以在不同进程中传递Message对象，在Message中放入我们需要传递的数据。Messenger是一种轻量级的IPC方案，它的低层实现是AIDL。同时由于它一次处理一个请求，因此在服务端我们不用考虑线程同步的问题，这是因为服务端中不存在并发执行的情形。   
  
实现一个Messenger分为服务端和客户端。  
  
1. 服务端进程  
	* 首先我们需要在服务端创建一个Service来处理客户端的连接请求，同时创建一个Handler并通过它创建一个Messenger对象，然后在Service的onBind中返回这个Messenger对象底层的Binder即可。  

2. 客户端  
	* 首先需要绑定服务端Service，绑定成功后用服务端返回的IBinder对象创建一个Messenger，通过这个Messenger就可以向服务端发送消息了，发送消息的类型是Message对象。如果需要服务端回复客户端，客户端就和服务端一样，创建一个Handler并创建一个Messenger对象，使用在发送给服务端的Message.replyTo指定为客户端的Messenger对象。  

![](https://i.imgur.com/277U6o8.jpg)	  
    
![](https://i.imgur.com/uJdMS0q.jpg)
  
![](https://i.imgur.com/RVDmShJ.jpg)  


