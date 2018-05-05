# Android 开发艺术探索  
---
  
## Android的线程和线程池   
  
![](https://i.imgur.com/HzbRjel.jpg)   

![](https://i.imgur.com/HolUBeL.jpg)  
   
### AsyncTask  
   
AsyncTask是一种轻量级的异步任务类，它可以在线程池中执行后台任务，然后把执行的进度和最终结果传递给主线程并在主线程中更新UI。从实现上来说，AsyncTask封装了Thread和Handler，通过AsyncTask并不适合执行特别耗时的后台任务，对于特别耗时的任务来说，建议使用线程池。
   
![](https://i.imgur.com/gFM0oIP.jpg)   
   
### HandlerThread  
  
![](https://i.imgur.com/790ww9p.jpg)   
   
![](https://i.imgur.com/eXC3xhK.jpg)   
   
### IntentService  
  
![](https://i.imgur.com/x42lxTT.jpg)    
    
### Android线程池  
   
![](https://i.imgur.com/5eaqzhy.png)    
   
Java中线程通常有五种状态： 创建、 就绪、 运行、 阻塞、 死亡；   
     
#### ThreadPoolExecutor  
![](https://i.imgur.com/7qJOvZu.jpg)   
   
![](https://i.imgur.com/E5CgxDK.jpg)
     
![](https://i.imgur.com/P7SacTj.jpg)  
  
#### 线程池分类   
   
![](https://i.imgur.com/ggPAixQ.jpg)   
   
![](https://i.imgur.com/V4Iao6Y.jpg)   
   
![](https://i.imgur.com/T7oYsSC.jpg)   
   
