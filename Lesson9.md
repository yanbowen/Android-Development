# Android 开发艺术探索  
---
  
## Bitmap的加载和Cache   
   
### 一、Bitmap的基本加载   
   
Bitmap的加载离不开BitmapFactory类，关于Bitmap官方介绍Creates Bitmap objects from various sources, including files, streams, and byte-arrays.查看api，发现和描述的一样，BitmapFactory类提供了四类方法用来加载Bitmap：

1. decodeFile 从文件系统加载   
	a. 通过Intent打开本地图片或照片   
	b. 在onActivityResult中获取图片uri   
	c. 根据uri获取图片的路径   
	d. 根据路径解析bitmap:Bitmap bm = BitmapFactory.decodeFile(sd_path)   
2. decodeResource 以R.drawable.xxx的形式从本地资源中加载  
	Bitmap bm = BitmapFactory.decodeResource(getResources(), R.drawable.aaa);
3. decodeStream 从输入流加载    
	a.开启异步线程去获取网络图片  
	b.网络返回InputStream   
	c.解析：Bitmap bm = BitmapFactory.decodeStream(stream),这是一个耗时操作，要在子线程中执行   
4. decodeByteArray 从字节数组中加载
	a.开启异步线程去获取网络图片    
	b.网络返回InputStream    
	c. 把InputStream转换成byte[]   
	d. 解析：Bitmap bm = BitmapFactory.decodeByteArray(myByte,0,myByte.length);   

### 二、高效的加载Bitmap    
  
我们在使用bitmap时，经常会遇到内存溢出等情况，这是因为图片太大或者android系统对单个应用施加的内存限制等原因造成的，比如上述方法1加载一张照片时就会报:06-28 10:43:30.777 26007-26036/com.peak.app W/OpenGLRenderer: Bitmap too large to be uploaded into a texture (3120x4160, max=4096x4096)，而方法2加载一个3+G的照片时会报Caused by: java.lang.OutOfMemoryError: Failed to allocate a 144764940 byte allocation with 16765264 free bytes and 109MB until OOM所以，高效的使用bitmap就显得尤为重要，对他效率的优化也是如此。    
    
高效加载Bitmap的思想也很简单，就是使用系统提供给我们Options类来处理Bitmap。翻看Bitmap的源码，发现上述四个加载bitmap的方法都是支持Options参数的。   
   
通过BitmapFactory.Options按一定的采样率来加载缩小后的图片，然后在ImageView中使用缩小的图片这样就会降低内存占用避免【OOM】，提高了Bitamp加载时的性能。   
   
这其实就是我们常说的图片尺寸压缩。尺寸压缩是压缩图片的像素，一张图片所占内存的大小 图片类型＊宽＊高，通过改变三个值减小图片所占的内存，防止OOM，当然这种方式可能会使图片失真 。   
   
android 色彩模式说明：    

* ALPHA_8：每个像素占用1byte内存。
* ARGB_4444:每个像素占用2byte内存
* ARGB_8888:每个像素占用4byte内存
* RGB_565:每个像素占用2byte内存
    
Android默认的色彩模式为ARGB_8888，这个色彩模式色彩最细腻，显示质量最高。但同样的，占用的内存也最大。   
   
BitmapFactory.Options的inPreferredConfig参数可以 指定decode到内存中，手机中所采用的编码，可选值定义在Bitmap.Config中。缺省值是ARGB_8888。    
   
假设一张1024\*1024，模式为ARGB_8888的图片，那么它占有的内存就是：1024\*1024\*4 = 4MB    
   
#### 1、采样率inSampleSize
* inSampleSize的值必须大于1时才会有效果，且采样率同时作用于宽和高；   
* 当inSampleSize=1时，采样后的图片为图片的原始大小   
* 当inSampleSize=2时，采样后的图片的宽高均为原始图片宽高的1/2，这时像素为原始图片的1/(22),占用内存也为原始图片的1/(22);   
* inSampleSize的取值应该总为2的整数倍，否则会向下取整，取一个最接近2的整数倍，比如inSampleSize=3时，系统会取inSampleSize=2   

假设一张1024\*1024,模式为ARGB_8888的图片,inSampleSize=2，原始占用内存大小是4MB，采样后的图片占用内存大小就是(1024/2) * (1024/2 )* 4 = 1MB    
   
#### 2、获取采样率遵循以下步骤
* 将BitmapFacpry.Options的inJustDecodeBounds参数设为true并加载图片当inJustDecodeBounds为true时，执行decodeXXX方法时，BitmapFactory只会解析图片的原始宽高信息，并不会真正的加载图片
* 从BitmapFacpry.Options取出图片的原始宽高(outWidth,outHeight)信息
* 选取合适的采样率
* 将BitmapFacpry.Options的inSampleSize参数设为false并重新加载图片
   
经过上面过程加载出来的图片就是采样后的图片，代码如下：   
   
	public void decodeResource(View view) {
    	Bitmap bm = decodeBitmapFromResource();
    	imageview.setImageBitmap(bm);
	}

	private Bitmap decodeBitmapFromResource(){
    	BitmapFactory.Options options = new BitmapFactory.Options();
    	options.inJustDecodeBounds = true;
    	BitmapFactory.decodeResource(getResources(), R.drawable.bbbb, options);
    	options.inSampleSize = calculateSampleSize(options,300,300);
    	options.inJustDecodeBounds =false;
    	return  BitmapFactory.decodeResource(getResources(),R.drawable.bbbb,options);
	}

	// 计算合适的采样率(当然这里还可以自己定义计算规则)，reqWidth为期望的图片大小，单位是px
	private int calculateSampleSize(BitmapFactory.Options options,int reqWidth,int reqHeight){
    	Log.i("========","calculateSampleSize reqWidth:"+reqWidth+",reqHeight:"+reqHeight);
    	int width = options.outWidth;
    	int height =options.outHeight;
    	Log.i("========","calculateSampleSize width:"+width+",height:"+height);
    	int inSampleSize = 1;
    	int halfWidth = width/2;
    	int halfHeight = height/2;
    	while((halfWidth/inSampleSize)>=reqWidth&& (halfHeight/inSampleSize)>=reqHeight){
    	    inSampleSize*=2;
    	    Log.i("========","calculateSampleSize inSampleSize:"+inSampleSize);
    	}
    	return inSampleSize;
	}

   
### 三、Android中的缓存策略   
  
在Android中当加载大量图片时首先需要考虑的一个问题是如何避免OOM。为了保证内存的使用始终维持在一个合理的范围，通常会把移出屏幕的图片进行回收处理，此时垃圾回收器会认为你不再持有这些图片的引用，从而对这些图片进行GC。然而当某些图片被回收之后用户又将它重新滑入屏幕时，这时又会去重新加载一遍刚刚加载过的图片。这样频繁地处理图片的加载和回收不利于操作的流畅性，而内存和硬盘的Cache就会帮助解决这个问题，实现快速加载已加载过的图片。   
   
在缓存上，主要有两种级别的Cache：LruCache和DiskLruCache。 前者是基于内存的，后者是基于硬盘的。   
    
#### LruCache（内存缓存技术）
   
内存缓存技术对那些大量占用应用程序宝贵内存的图片提供了快速访问的方法。以前常用的内存缓存是通过SoftReference或WeakReference来实现的，但现在不推荐使用这种方式了，从Android2.3（API 9）开始，垃圾回收器会更倾向于回收持有软引用或弱引用的对象，这让软引用和弱引用变得不再可靠。   
   
现在非常流行的内存缓存技术是LruCache（LRU是Least Recently Used 近期最少使用算法），在android-support-v4包中提供。这个类非常适合用来缓存图片，它的主要算法原理是把最近使用的对象用强引用存储在 LinkedHashMap 中，并且把最近最少使用的对象在缓存值达到预设定值之前从内存中移除。 
下面是一个使用 LruCache 来缓存图片的例子：   
   
	public class BitmapCache {
    	private LruCache<String, Bitmap> mMemoryCache;

	    public BitmapCache() {
	     // 获取到可用内存的最大值，使用内存超出这个值会引起OutOfMemory异常
	       int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);  
	       // 使用最大可用内存值的1/8作为缓存的大小。  
	       int cacheSize = maxMemory / 8;  
	       mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {  
	           @Override  
	           protected int sizeOf(String key, Bitmap bitmap) {  
	               // 重写此方法来衡量每张图片的大小，默认返回图片数量。  
	               return bitmap.getByteCount() / 1024;  
	           }  
	       };
	    }
	
	    public void addBitmapToMemoryCache(String key, Bitmap bitmap) {  
	       if (getBitmapFromMemCache(key) == null) {  
	          mMemoryCache.put(key, bitmap);  
	      }  
	  }  

	   public Bitmap getBitmapFromMemCache(String key) {  
	       return mMemoryCache.get(key);  
	   } 
	}     
    
这里使用了系统分配给应用程序的八分之一内存来作为缓存大小。那么缓存大小有什么限制吗？其实并没有一个指定大小的缓存可以满足所有的应用程序，这是由你决定的。你应该去分析程序内存的使用情况，然后制定出一个合适的解决方案。一个太小的缓存空间，有可能造成图片频繁地被释放和重新加载，这并没有好处。而一个太大的缓存空间，则有可能还是会引起 Java.lang.OutOfMemory 的异常。     
    
#### DiskLruCache（硬盘缓存技术）    
   
#### 使用
由于DiskLruCache并不是由Google官方编写的，所以这个类并没有被包含在Android API当中，我们需要将这个类从网上下载下来，然后手动添加到项目当中。DiskLruCache源码见后面Demo。下载好源码之后，只需要在项目中新建一个libcore.io包，然后将DiskLruCache.Java文件复制到这个包中即可。    
     
#### 打开缓存
DiskLruCache是不能new出实例的，如果我们要创建一个DiskLruCache的实例，则需要调用它的open()方法，接口如下所示：    

	public static DiskLruCache open(File directory, int appVersion, int valueCount, long maxSize)    
   
open()方法接收四个参数，第一个参数指定的是数据的缓存地址，第二个参数指定当前应用程序的版本号，第三个参数指定同一个key可以对应多少个缓存文件，基本都是传1，第四个参数指定最多可以缓存多少字节的数据。    
DiskLruCache并没有限制数据的缓存位置，可以自由地进行设定，但是通常情况下多数应用程序都会将缓存的位置选择为 /sdcard/Android/data/Application package/cache这个路径。选择在这个位置有两点好处：第一，这是存储在SD卡上的，因此即使缓存再多的数据也不会对手机的内置存储空间有任何影响，只要SD卡空间足够就行。第二，这个路径被Android系统认定为应用程序的缓存路径，当程序被卸载的时候，这里的数据也会一起被清除掉，这样就不会出现删除程序之后手机上还有很多残留数据的问题。但同时我们又需要考虑如果这个手机没有SD卡，或者SD正好被移除了的情况，因此比较优秀的程序都会专门写一个方法来获取缓存地址，如下所示：    
   
	public File getDiskCacheDir(Context context, String uniqueName) {  
    	String cachePath;  
    	if (Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState()) || !Environment.isExternalStorageRemovable()) {  
        cachePath = context.getExternalCacheDir().getPath();  
    	} else {  
        	cachePath = context.getCacheDir().getPath();  
    	}  
    	return new File(cachePath + File.separator + uniqueName);  
	}   
    
可以看到，当SD卡存在或者SD卡不可被移除的时候，就调用getExternalCacheDir()方法来获取缓存路径，否则就调用getCacheDir()方法来获取缓存路径。前者获取到的就是 /sdcard/Android/data/application package/cache 这个路径，而后者获取到的是 /data/data/application package/cache 这个路径。      
   
接着将获取到的路径和一个uniqueName进行拼接，作为最终的缓存路径返回。那么这个uniqueName又是什么呢？其实这就是为了对不同类型的数据进行区分而设定的一个唯一值，比如说bitmap、object等文件夹。     
接着是应用程序版本号，我们可以使用如下代码简单地获取到当前应用程序的版本号：    
    
	public int getAppVersion(Context context) {  
    	try {  
        	PackageInfo info = context.getPackageManager().getPackageInfo(context.getPackageName(), 0);  
        	return info.versionCode;  
    	} catch (NameNotFoundException e) {  
        	e.printStackTrace();  
    	}  
    	return 1;  
	}     
    
需要注意的是，每当版本号改变，缓存路径下存储的所有数据都会被清除掉，因为DiskLruCache认为当应用程序有版本更新的时候，所有的数据都应该从网上重新获取。    
后面两个参数就没什么需要解释的了，第三个参数传1，第四个参数通常传入10M的大小就够了，这个可以根据自身的情况进行调节。    
   
因此，一个非常标准的open()方法就可以这样写：   

	DiskLruCache mDiskLruCache = null;  
	try {  
    	File cacheDir = getDiskCacheDir(context, "bitmap");  
    	if (!cacheDir.exists()) {  
    	    cacheDir.mkdirs();  
    	}  
    	mDiskLruCache = DiskLruCache.open(cacheDir, getAppVersion(context), 1, 10 * 1024 * 1024);  
	} catch (IOException e) {  
    	e.printStackTrace();  
	}    
   
#### 关闭缓存   
   
这个方法用于将DiskLruCache关闭掉，是和open()方法对应的一个方法。关闭掉了之后就不能再调用DiskLruCache中任何操作缓存数据的方法，通常只应该在Activity的onDestroy()方法中去调用close()方法。   

	mDiskLruCache.close();   
   
#### 写入缓存  
   
来看写入，比如说现在有一张图片，地址是http://img.my.csdn.net/uploads/201308/31/1377949454_6367.jpg，那么为了将这张图片下载下来，可以这样写：      
   
	private boolean downloadUrlToStream(String urlString, OutputStream outputStream) {  
    	HttpURLConnection urlConnection = null;  
    	BufferedOutputStream out = null;  
    	BufferedInputStream in = null;  
    	try {  
    	    final URL url = new URL(urlString);  
    	    urlConnection = (HttpURLConnection) url.openConnection();  
    	    in = new BufferedInputStream(urlConnection.getInputStream());  
    	    out = new BufferedOutputStream(outputStream);  
    	    int b;  
    	    while ((b = in.read()) != -1) {  
    	        out.write(b);  
    	    }  
    	    return true;  
    	} catch (final IOException e) {  
    	    e.printStackTrace();  
    	} finally {  
    	    if (urlConnection != null) {  
    	        urlConnection.disconnect();  
    	    }  
    	    try {  
    	        if (out != null) {  
    	            out.close();  
    	        }  
    	        if (in != null) {  
    	            in.close();  
    	        }  
    	    } catch (final IOException e) {  
    	        e.printStackTrace();  
    	    }  
    	}  
    	return false;  
	}    
    
图片的传统硬盘缓存就可以通过outputStream写入到本地文件。有了这个方法之后，下面我们就可以使用DiskLruCache来进行写入了，写入的操作是借助DiskLruCache.Editor这个类完成的。类似地，这个类也是不能new的，需要调用DiskLruCache的edit()方法来获取实例，接口如下所示：   
   
	public Editor edit(String key) throws IOException    
   
edit()方法接收一个参数key，这个key将会成为缓存文件的文件名，并且必须要和图片的URL是一一对应的。那么怎样才能让key和图片的URL能够一一对应呢？直接使用URL来作为key？不太合适，因为图片URL中可能包含一些特殊字符，这些字符有可能在命名文件时是不合法的。其实最简单的做法就是将图片的URL进行MD5编码，编码后的字符串肯定是唯一的，并且只会包含0-F这样的字符，完全符合文件的命名规则。    
    
我们可以写一个字符串MD5编码的工具类    
   
	public String getMD5String(String key) {  
    	String cacheKey;  
    	try {  
        	final MessageDigest mDigest = MessageDigest.getInstance("MD5");  
        	mDigest.update(key.getBytes());  
        	cacheKey = bytesToHexString(mDigest.digest());  
    	} catch (NoSuchAlgorithmException e) {  
        	cacheKey = String.valueOf(key.hashCode());  
    	}  
    	return cacheKey;  
	}  

	private String bytesToHexString(byte[] bytes) {  
    	StringBuilder sb = new StringBuilder();  
    	for (int i = 0; i < bytes.length; i++) {  
    	    String hex = Integer.toHexString(0xFF & bytes[i]);  
    	    if (hex.length() == 1) {  
    	        sb.append('0');  
    	    }  
    	    sb.append(hex);  
    	}  
    	return sb.toString();  
	}    
    
现在就可以这样写来得到一个DiskLruCache.Editor的实例：    
   
	String imageUrl = "http://img.my.csdn.net/uploads/201308/31/1377949454_6367.jpg";  
	String key = getMD5String(imageUrl);  
	DiskLruCache.Editor editor = mDiskLruCache.edit(key);    
   
有了DiskLruCache.Editor的实例之后，我们可以调用它的newOutputStream()方法来创建一个输出流，然后把它传入到downloadUrlToStream()中就能实现下载并写入缓存的功能了。注意newOutputStream()方法接收一个index参数，由于前面在设置valueCount的时候指定的是1，所以这里index传0就可以了。在写入操作执行完之后，我们还需要调用一下commit()方法进行提交才能使写入生效，调用abort()方法的话则表示放弃此次写入。     
   
因此，一次完整写入操作的代码如下所示：    
   
	new Thread(new Runnable() {  
    	@Override  
    	public void run() {  
        	try {  
        	    String imageUrl = "http://img.my.csdn.net/uploads/201308/31/1377949454_6367.jpg";  
        	    String key = getMD5String(imageUrl);  
        	    DiskLruCache.Editor editor = mDiskLruCache.edit(key);  
        	    if (editor != null) {  
        	        OutputStream outputStream = editor.newOutputStream(0);
        	        if (downloadUrlToStream(imageUrl, outputStream)) {  
        	            editor.commit();  
        	        } else {
        	            editor.abort();  
        	        }  
        	    }  
        	    mDiskLruCache.flush();  
        	} catch (IOException e) {  
        	    e.printStackTrace();  
        	}  
    	}  
	}).start();    
    
由于这里调用了downloadUrlToStream()方法来从网络上下载图片，所以一定要确保这段代码是在子线程当中执行的。注意在代码的最后我还调用了一下flush()方法，这个方法并不是每次写入都必须要调用的，但在这里却不可缺少，我会在后面说明它的作用。    
    
#### 读取缓存   
读取的方法要比写入简单一些，主要是借助DiskLruCache的get()方法实现的，接口如下所示：   
   
	public synchronized Snapshot get(String key) throws IOException    
    
get()方法要求传入一个key来获取到相应的缓存数据，而这个key毫无疑问就是将图片URL进行MD5编码后的值了，因此读取缓存数据的代码就可以这样写    
   
	String imageUrl = "http://img.my.csdn.net/uploads/201308/31/1377949454_6367.jpg";  
	String key = getMD5String(imageUrl);  
	DiskLruCache.Snapshot snapShot = mDiskLruCache.get(key);    
   
这里获取到的是一个DiskLruCache.Snapshot对象，这个对象我们该怎么利用呢？很简单，只需要调用它的getInputStream()方法就可以得到缓存文件的输入流了。同样地，getInputStream()方法也需要传一个index参数，这里传入0就好。有了文件的输入流之后，想要把缓存图片显示到界面上就轻而易举了。所以，一段完整的读取缓存，并将图片加载到界面上的代码如下所示：    
   
	try {  
    	String imageUrl = "http://img.my.csdn.net/uploads/201308/31/1377949454_6367.jpg";  
    	String key = getMD5String(imageUrl); 
    	DiskLruCache.Snapshot snapShot = mDiskLruCache.get(key);  
    	if (snapShot != null) {  
        	InputStream is = snapShot.getInputStream(0);  
        	Bitmap bitmap = BitmapFactory.decodeStream(is);  
        	mImage.setImageBitmap(bitmap);  
    	}  
	} catch (IOException e) {  
    	e.printStackTrace();  
	}    
    
#### 移除缓存   
  
学习完了写入缓存和读取缓存的方法之后，最难的两个操作你就都已经掌握了，那么接下来要学习的移除缓存对你来说也一定非常轻松了。移除缓存主要是借助DiskLruCache的remove()方法实现的，接口如下所示：    
    
	public synchronized boolean remove(String key) throws IOException    
    
remove()方法中要求传入一个key，然后会删除这个key对应的缓存图片，示例代码如下：   
   
	try {  
    	String imageUrl = "http://img.my.csdn.net/uploads/201308/31/1377949454_6367.jpg";  
    	String key = getMD5String(imageUrl);   
    	mDiskLruCache.remove(key);  
	} catch (IOException e) {  
    	e.printStackTrace();  
	}     
        
用法虽然简单，但是你要知道，这个方法我们并不应该经常去调用它。因为你完全不需要担心缓存的数据过多从而占用SD卡太多空间的问题，DiskLruCache会根据我们在调用open()方法时设定的缓存最大值来自动删除多余的缓存。只有你确定某个key对应的缓存内容已经过期，需要从网络获取最新数据的时候才应该调用remove()方法来移除缓存。   
   
#### 清空缓存   
   
移除缓存只能根据key移除掉某一个缓存，如果我们需要清空DiskLruCache的所有缓存呢？delete()方法用于将所有的缓存数据全部删除，比如说手动清理缓存功能就可以用它来做。    
   
	mDiskLruCache.delete();   

原文地址  
**https://blog.csdn.net/huaxun66/article/details/52434528**