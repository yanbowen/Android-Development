## Android 打包过程   
   
### 1. Android APK是如何来的呢？
  

![](https://i.imgur.com/M6jkahd.png)   
   
* 由android的项目经过编译和打包，形成了:

	1. **.dex 文件**  
	2. **resources.arsc**   
	3. **uncompiled resources**  
	4. **AndroidManifest.xml**   
   
解压了一个普通的apk文件，解压出来的文件如下：   
   
![](https://i.imgur.com/aiIHXE1.png)  
   
classes.dex 是.dex文件。  
resources.arsc是resources resources文件。  
AndroidManifest.xml是AndroidManifest.xml文件。  
res是uncompiled resources。  
META-INF是签名文件夹。  
  
* META-INF其中有三个文件：   

![](https://i.imgur.com/jX2U9KP.png)   
   
MANIFEST.MF文件   
版本号以及每一个文件的哈希值（BASE64）。包括资源文件。这个是对每个文件的整体进行SHA1(hash)。    
   
	Manifest-Version: 1.0
	Built-By: Generated-by-ADT
	Created-By: Android Gradle 2.2.0
	Name: res/drawable-xhdpi-v4/abc_scrubber_control_to_pressed_mtrl_005.png
	SHA1-Digest: I9s6aQ5VyOLrNo4odqSij549Oyo=
	Name: res/drawable-mdpi-v4/abc_textfield_search_default_mtrl_alpha.9.png
	SHA1-Digest: D6dilO+UMcglambujyMOhNbLZuY=
	……
   
CERT.SF
这个是对每个文件的头3行进行SHA1 hash。   
   
	Signature-Version: 1.0
	X-Android-APK-Signed: 2
	SHA1-Digest-Manifest: QxOfCCAuQtZnHh0YRNnoxmiHT80=
	Created-By: 1.0 (Android)
	Name: res/drawable-xhdpi-v4/abc_scrubber_control_to_pressed_mtrl_005.png
	SHA1-Digest: I9s6aQ5VyOLrNo4odqSij549Oyo=
	Name: res/drawable-mdpi-v4/abc_textfield_search_default_mtrl_alpha.9.png
	SHA1-Digest: D6dilO+UMcglambujyMOhNbLZuY=
	……
   
CERT.RSA
这个文件保存了签名和公钥证书。   
    
---

### 2. 具体打包过程   
   
![](https://i.imgur.com/sjmQrXr.png)   
   
#### 2.1 aapt阶段
  
* 使用aapt来打包res资源文件，生成R.java、resources.arsc和res文件（二进制 & 非二进制如res/raw和pic保持原样）   
   
* res目录有9种目录   
**--animator**。这类资源以XML文件保存在res/animator目录下，用来描述属性动画。   
**--anim**。这类资源以XML文件保存在res/anim目录下，用来描述补间动画。   
**--color**。这类资源以XML文件保存在res/color目录下，用描述对象颜色状态选择子。   
**--drawable**。这类资源以XML或者Bitmap文件保存在res/drawable目录下，用来描述可绘制对象。例如，我们可以在里面放置一些图片（.png, .9.png, .jpg, .gif），来作为程序界面视图的背景图。注意，保存在这个目录中的Bitmap文件在打包的过程中，可能会被优化的。例如，一个不需要多于256色的真彩色PNG文件可能会被转换成一个只有8位调色板的PNG面板，这样就可以无损地压缩图片，以减少图片所占用的内存资源。   
**--layout**。这类资源以XML文件保存在res/layout目录下，用来描述应用程序界面布局。   
**--menu**。这类资源以XML文件保存在res/menu目录下，用来描述应用程序菜单。   
**--raw**。这类资源以任意格式的文件保存在res/raw目录下，它们和assets类资源一样，都是原装不动地打包在apk文件中的，不过它们会被赋予资源ID，这样我们就可以在程序中通过ID来访问它们。例如，假设在res/raw目录下有一个名称为filename的文件，并且它在编译的过程，被赋予的资源ID为R.raw.filename，那么就可以使用以下代码来访问它：    
   
		Resources res = getResources();  
		InputStream is = res .openRawResource(R.raw.filename);   
   
**--values**。这类资源以XML文件保存在res/values目录下，用来描述一些简单值，例如，数组、颜色、尺寸、字符串和样式值等，一般来说，这六种不同的值分别保存在名称为arrays.xml、colors.xml、dimens.xml、strings.xml和styles.xml文件中。   
**--xml**。这类资源以XML文件保存在res/xml目录下，一般就是用来描述应用程序的配置信息。   
    
* R.java文件

![](https://i.imgur.com/gI4m7vF.png)   

这就是R.java的源代码，里面拥有很多个静态内部类，比如layout，string等。   
每当有这种资源添加时，就在R.java文件中添加一条静态内部类里的静态常量类成员，且所有成员都是int类型。   
   
![](https://i.imgur.com/GMWsbDu.png)   
   
里面的资源可以有两种方法引用：   
1.在java程序中引用资源按照java的语法来引用即：R.resource_type.resource_name注意：resource_name不需要文件的后缀名   
2.在XML文件中引用资源格式：@[package:]type/name     
   
* resources.arsc文件  
resources.arsc这个文件记录了所有的应用程序资源目录的信息，包括每一个资源名称、类型、值、ID以及所配置的维度信息。我们可以将这个resources.arsc文件想象成是一个资源索引表，这个资源索引表在给定资源ID和设备配置信息的情况下，能够在应用程序的资源目录中快速地找到最匹配的资源。   
   
#### 2.2 aidl阶段
* AIDL （Android Interface Definition Language）， Android接口定义语言，Android提供的IPC （Inter Process Communication，进程间通信）的一种独特实现。
这个阶段处理.aidl文件，生成对应的Java接口文件。

#### 2.3 Java Compiler阶段
* 通过Java Compiler编译R.java、Java接口文件、Java源文件，生成.class文件。   

#### 2.4 dex阶段
* 通过dex命令，将.class文件和第三方库中的.class文件处理生成classes.dex。   

#### 2.5 apkbuilder阶段
* 将classes.dex、resources.arsc、res文件夹(res/raw资源被原装不动地打包进APK之外，其它的资源都会被编译或者处理)、Other Resources(assets文件夹)、AndroidManifest.xml打包成apk文件。   

#### 注意：
res/raw和assets的相同点：   

1. 两者目录下的文件在打包后会原封不动的保存在apk包中，不会被编译成二进制。    
   
res/raw和assets的不同点：    
 
1. res/raw中的文件会被映射到R.java文件中，访问的时候直接使用资源ID即R.id.filename；assets文件夹下的文件不会被映射到R.java中，访问的时候需要AssetManager类。   
2. res/raw不可以有目录结构，而assets则可以有目录结构，也就是assets目录下可以再建立文件夹   

#### 2.6 Jarsigner阶段
对apk进行签名，可以进行Debug和Release 签名。   
   
* release mode 下使用 aipalign进行align，即对签名后的apk进行对齐处理。   

* Zipalign是一个android平台上整理APK文件的工具，它对apk中未压缩的数据进行4字节对齐，对齐后就可以使用mmap函数读取文件，可以像读取内存一样对普通文件进行操作。如果没有4字节对齐，就必须显式的读取，这样比较缓慢并且会耗费额外的内存。    

* 在 Android SDK 中包含一个名为 “zipalign” 的工具，它能够对打包后的 app 进行优化。 其位于 SDK 的 build-tools 目录下, 例如: D:\Develop\Android\sdk\build-tools\23.0.2\zipalign.exe。     
   
---   
   
### Android APK加壳技术方案   
   
一、什么是加壳？

加壳是在二进制的程序中植入一段代码，在运行的时候优先取得程序的控制权，做一些额外的工作。大多数病毒就是基于此原理。PC EXE文件加壳的过程如下：   
   
![](https://i.imgur.com/jAIHruO.png)   

加壳的程序可以有效阻止对程序的反汇编分析
   
二、Android Dex文件加壳原理

在这个过程中，牵扯到三个角色：

    1、加壳程序：加密源程序为解壳数据、组装解壳程序和解壳数据

    2、解壳程序：解密解壳数据，并运行时通过DexClassLoader动态加载

    3、源程序：需要加壳处理的被保护代码   
   
根据解壳数据在解壳程序DEX文件中的不同分布，本文将提出两种Android Dex加壳的实现方案。   
   
（一）解壳数据位于解壳程序文件尾部   
  
         加壳程序工作流程：

                  1、加密源程序APK文件为解壳数据

                  2、把解壳数据写入解壳程序Dex文件末尾，并在文件尾部添加解壳数据的大小。

                  3、修改解壳程序DEX头中checksum、signature 和file_size头信息。

                  4、修改源程序AndroidMainfest.xml文件并覆盖解壳程序AndroidMainfest.xml文件。



          解壳DEX程序工作流程：

                  1、读取DEX文件末尾数据获取借壳数据长度。

                  2、从DEX文件读取解壳数据，解密解壳数据。以文件形式保存解密数据到a.APK文件

                  3、通过DexClassLoader动态加载a.apk。    
   
（二）解壳数据位于解壳程序文件头   
   
          加壳程序工作流程：

                  1、加密源程序APK文件为解壳数据

                  2、计算解壳数据长度，并添加该长度到解壳DEX文件头末尾，并继续解壳数据到文件头末尾。
					（插入数据的位置为0x70处）

                  3、修改解壳程序DEX头中checksum、signature、file_size、header_size、string_ids_off、
					type_ids_off、proto_ids_off、field_ids_off、method_ids_off、class_defs_off和data_off相关项。
					分析map_off 数据，修改相关的数据偏移量。  

                  4、修改源程序AndroidMainfest.xml文件并覆盖解壳程序AndroidMainfest.xml文件。



          解壳DEX程序工作流程：

                  1、从0x70处读取解壳数据长度。

                  2、从DEX文件读取解壳数据，解密解壳数据。以文件形式保存解密数据到a.APK

                  3、通过DexClassLoader动态加载a.APK。