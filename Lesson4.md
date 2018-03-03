# Android 开发艺术探索  
---
## android的消息机制  

本质上说，Handler并不是专门用于更新UI的，它只是常被开发者用来更新UI。
  
* MessageQueue虽然叫消息队列，但是它内部存储结构并不是真正的队列，而是采用单链表的数据结构来存储消息列表。  
* Looper会以无限循环的形式去查找是否有新消息，如果有的话就处理消息，否则就一直等待着。  
* Looper中还有一个特殊的概念，那就是ThreadLocal，ThreadLocal并不是线程，它的作用是可以在每个线程中存储数据。我们知道，Handler创建的时候会采用当前线程的Looper来构造消息循环系统，那么Handler内部如何获取到当前的线程的Looper呢，这就要使用到ThreadLocal，ThreadLocal可以在不同的线程中互不干扰地存储并提供数据，通过ThreadLocal可以轻松获取每个线程的Looper。需要注意的是，线程默认是没有Looper的，如果需要使用Handler就必须为线程创建Looper。主线程ActivityThread，创建的时候会去初始化Looper，所以主线程默认可以使用Handler。  

* Android为什么不允许在子线程中访问UI呢？   
这是因为Android的UI控件不是线程安全的，如果在多线程中并发访问可能会导致UI控件处于不可预期的状态。   

* 为什么系统不对UI控件的访问加上锁机制呢？   
首先加上锁机制会让UI访问的逻辑变得复杂，其次锁机制会降低UI访问的效率，因为锁机制会阻塞某些线程的执行。  
  
Handler创建时会采用当前线程的Looper来构造内部的消息循环系统，如果当前线程没有Looper，那么就会报错。 
  
Handler创建完毕后，这个时候其内部的Looper以及MessageQueue就可以和Handler一起协同工作了，然后通过Handler的post方法将一个Runnable投递到Handler内部的Looper中去处理，也可以通过Handler的send方法发送一个消息，这个消息同样会在Looper中去处理。其实post方法最终也是通过send方法来完成的。  
  
* Send方法的工作过程:   
当Handler的send方法被调用时，它会调用MessageQueue的enqueueMessage方法将这个消息放入消息队列中，然后Looper发现有新消息到来时，就会处理这个消息，最终消息中的Runnable或者Handler的handleMessage方法就会被调用。注意Looper是运行在创建Handler所在的线程中的，这样一来Handler中的业务逻辑就被切换到创建Handler所在的线程中去了。  
  
### ThreadLocal的工作原理  
  
ThreadLocal是一个线程内部的数据存储类，通过它可以在指定的线程中存储数据，存储以后，只能在该线程中可以获取到存储的数据，对于其他线程来说无法获取。  
   
**使用场景**  

* 某些数据以线程为作用域并且在不同线程有不同的数据时。比如每个线程要使用Handler需要创建Looper，而且每个线程的Looper是不同的，所以使用ThreadLocal可以轻松的实现Looper在线程中的存取。  
* 复杂逻辑下的对象传递。比如线程中的一个监听器，它需要贯穿线程的整个生命周期，这时就可以使用ThreadLocal让监听器作为线程中的全局变量而存在。  

**原理**  
之所以可以在指定的线程中存储数据，而其他线程无法获取的原因是：不同线程访问同一个ThreadLocal的get方法，ThreadLocal内部会从各自的线程中取出一个数组，然后再从数组中根据当前ThreadLocal的索引去查询出对应的value值。  
  
从ThreadLocal的set和get方法可以看出，他们所操作的对象都是当前线程的localValues对象的table数组，因此在不同线程中访问同一个ThreadLocal的set和get方法，它们对ThreadLocal所做的读\写操作仅限于各自线程的内部，这就是ThreadLocal可以在多个线程中互不干扰地存储和修改数据。  
  
消息队列MessageQueue主要包含两个操作：插入和读取。对应方法分别为恩queueMessage和next；  
可以发现next方法是一个无线循环的方法，如果消息队列中没有消息，那么next方法会一直阻塞在这里。当有新消息到来时，next方法会返回这条消息并将其从单链表中移出。  
  
![](https://i.imgur.com/l274r68.jpg)  
  
![](https://i.imgur.com/XzBvIr7.jpg)  
  
![](https://i.imgur.com/ESLVMti.jpg)  
  
![](https://i.imgur.com/jhwPkGJ.jpg)  
  
![](https://i.imgur.com/HcacAxQ.jpg)  
  
![](https://i.imgur.com/WQJPltE.jpg)  
  
