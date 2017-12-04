# Android 开发艺术探索  
---
## 第二章  
### Android IPC简介  
* IPC是Inter-Process Communication的缩写，含义：进程间通信，指两个进程之间进行数据交换的过程。  
* 线程是CPU调度的最小单元，同时线程是一种有限资源系统资源。  
* 进程一般指一个执行单元，在PC和移动设备上指一个程序或者一个应用。  
<center>__一个进程可以包含多个线程，也可以只有一个线程，即主线程，在Android里面主线程也叫UI线程。__</center>  
* __ANR__:Application Not Responding,即应用无响应。  
  
在Android中使用多进程只有一种方法，那就是给四大组件（Activity、Service、Receiver、ContentProvider）中指定android:process属性。另一种不常用的是通过JNI在native层去fork一个新的进程。  
进程名以“：”开头的进程属于当前应用的私有进程，其他应用的组件不可以和它跑在同一个进程中，而进程名不以“：”开始的属于全局进程，其他应用通过ShareUID方式可以和它跑在同一个进程中。  
##### 一般来说，多进程会造成如下几个方面的问题  
* 静态成员和单例模式完全失效  
* 线程同步机制完全失效  
* SharePreferences的可靠性下降。  
* Application会多次创建  
因为不是同一块内存，所以不管是锁对象还是锁全局类都无法保证线程同步，SharePreferences不支持两个进程同时去执行写操作，会导致一定几率数据丢失，这是因为SharePreferences底层是通过读/写XML文件来实现的，并发写显然是会出问题的。  
##### 跨进程通信方式：   
* Intent传递消息  
* 共享文件  
* SharePreferences  
* 基于Binder的Messenger  
* AIDL及Socket  
##### Serializable接口  
Serializable是Java所提供的一个序列化接口，它是一个空接口，为对象提供标准的序列化和反序列化操作。操作方法只需该类实现Serializable接口并声明一个SerialVersionUID,如下则是自动序列化过程：  

    private static final long serialVersionUID = 6587943156456215786L  
##### 序列化：将对象的状态信息转换为可以存储或传输的形式的过程。在序列化期间，对象将其当前状态写入到临时或持久性存储区。以后可以通过从存储区中读取或反序列化对象的状态，重新创建该对象。  
_对于任何可能包含重要的安全性数据的对象，如果可能，应该使该对象不可序列化。如果它必须为可序列化的，请尝试生成特定字段来保存不可序列化的重要数据。如果无法实现这一点，则应注意该数据会被公开给任何拥有序列化权限的代码，并确保不让任何恶意代码获得该权限。_  
  
##### Parcelable接口  
Parcelable是Android中的序列化方式，更适合在Android平台上。效率高，但过程麻烦。  

由于不同的进程有着不同的内存区域，并且它们只能访问自己的那一块内存区域，所以我们不能像平时那样，传一个句柄过去就完事了——句柄指向的是一个内存区域，现在目标进程根本不能访问源进程的内存，那把它传过去又有什么用呢？所以我们必须将要传输的数据转化为能够在内存之间流通的形式。这个转化的过程就叫做序列化与反序列化。  

>实现 Parcelable 接口  

* 创建一个类，正常的书写其成员变量，建立getter和setter，然后 implements Parcelable；默认生成的类里面只有 writeToParcel() 方法，该方法的对象只支持为 in 的定向 tag  。  

*那么这个 readFromParcel() 方法应当怎么写呢？这样写：*  
<pre><code>@Override
public void writeToParcel(Parcel dest, int flags) {
    dest.writeString(name);
    dest.writeInt(price);
}

/**
 * 参数是一个Parcel,用它来存储与传输数据
 * @param dest
 */
public void readFromParcel(Parcel dest) {
    //注意，此处的读值顺序应当是和writeToParcel()方法中一致的
    name = dest.readString();
    price = dest.readInt();
}</code></pre>

##### Binder类  
* Binder是Android中的一个类，它实现了IBinder接口。
* 从IPC角度来说，Binder是Android中的一种跨进程通信方式  
* 从AndroidFramework角度来说，Binder是ServiceManager连接各种Manager(ActivityManager、windowManager等等)和相应ManagerService的桥梁  
* 从Android应用层来说，Binder是客户端和服务端进行通信的媒介。  
  
##### AIDL
AIDL是一个缩写，全称是Android Interface Definition Language，也就是android接口定义语言。   
设计这门语言的目的是为了实现进程间通信，尤其是在涉及多进程并发情况下的进程间通信。  
AIDL这么语言和Java语法基本一样，不通点主要有：  

* 文件类型：用AIDL书写的文件的后缀是 .aidl，而不是 .java  
* 数据类型：AIDL默认支持一些数据类型，在使用这些数据类型的时候是不需要导包的，但是除了这些类型之外的数据类型，在使用之前必须导包，就算目标文件与当前正在编写的 .aidl 文件在同一个包下——在 Java 中，这种情况是不需要导包的。比如，现在我们编写了两个文件，一个叫做 Book.java ，另一个叫做 IBookManager.aidl，它们都在 com.ipc.aidl 包下 ，现在我们需要在 .aidl 文件里使用 Book 对象，那么我们就必须在 .aidl 文件里面写上 import com.ipc.aidl.Book; 哪怕 .java 文件和 .aidl 文件就在一个包下。    
<center>![](http://i.imgur.com/JJJSdXk.png)</center>
* 默认支持的数据类型包括：  
	> Java中的八种基本数据类型，包括 byte，short，int，long，float，double，boolean，char  
	> String 类型   
	> CharSequence类型  
	> List类型：List中的所有元素必须是AIDL支持的类型之一，或者是一个其他AIDL生成的接口，或者是定义的parcelable,List可以使用泛型。  
	> Map类型：Map中的所有元素必须是AIDL支持的类型之一，或者是一个其他AIDL生成的接口，或者是定义的parcelable,Map是不支持泛型的。  
* 定向tag：AIDL中的定向 tag 表示了在跨进程通信中数据的流向，其中 in 表示数据只能由客户端流向服务端， out 表示数据只能由服务端流向客户端，而 inout 则表示数据可在服务端与客户端之间双向流通。其中，数据流向是针对在客户端中的那个传入方法的对象而言的。in 为定向 tag 的话表现为服务端将会接收到一个那个对象的完整数据，但是客户端的那个对象不会因为服务端对传参的修改而发生变动；out 的话表现为服务端将会接收到那个对象的的空对象，但是在服务端对接收到的空对象有任何修改之后客户端将会同步变动；inout 为定向 tag 的情况下，服务端将会接收到客户端传来对象的完整信息，并且客户端将会同步服务端对该对象的任何变动。  
* 两种AIDL文件：在我的理解里，所有的AIDL文件大致可以分为两类。一类是用来定义parcelable对象，以供其他AIDL文件使用AIDL中非默认支持的数据类型的。一类是用来定义方法接口，以供系统使用来完成跨进程通信的。可以看到，两类文件都是在“定义”些什么，而不涉及具体的实现，这就是为什么它叫做“Android接口定义语言”。  

<pre><code>// Book.aidl
//第一类AIDL文件的例子
//这个文件的作用是引入了一个序列化对象 Book 供其他的AIDL文件使用
//注意：Book.aidl与Book.java的包名应当是一样的
package com.ipc.aidl;

//注意parcelable是小写
parcelable Book;
</code></pre>  
<pre><code>// IBookManager.aidl
//第二类AIDL文件的例子
package com.ipc.aidl;
//导入所需要使用的非默认支持数据类型的包
import com.ipc.aidl.Book;

interface IBookManager {

    //所有的返回值前都不需要加任何东西，不管是什么数据类型
    List<Book> getBookList();
    Book getBook();
    int getBookCount();

    //传参时除了Java基本类型以及String，CharSequence之外的类型
    //都需要在前面加上定向tag，具体加什么量需而定
    void addBook(in Book book);
}
</code></pre>  
