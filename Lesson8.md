## RecycleBin缓存机制的工作原理：   
   
ListView每当一项子view滑出界面时，RecycleBin会调用addScrapView()方法将这个废弃的子view进行缓存。每当子view滑入界面时，RecycleBin会调用getScrapView()方法获取一个废弃已缓存的view。所以我们再看回Adapter的getView()方法：  
   
	@Override  
	public View getView(int position, View convertView, ViewGroup parent) {   
    	View view;  
    	if (convertView == null) {  
    	    view = LayoutInflater.from(context).inflate(resourceId, null);  
    	    ······
    	} else {  
    	    view = convertView;  
    	}   
    	······
    	return view;  
	} 
   
convertView就是RecycleBin缓存机制调用getScrapView()方法获取废弃已缓存的view。     
   
### ListView的工作原理    
  
ListView终究还是一个View。而每一个View的工作流程都是分为三步，onMeasure()测量，接着onLayout()布局，最后onDraw()绘制。

#### 第一次onLayout() ： 
ListView在加载子项视图的时候，先判断是否有子元素、RecycleBin缓存机制中是否已经有缓存视图了。由于此时ListView是第一次加载，没有任何视图，RecycleBin中也没有任何的缓存记录，所以ListView就直接进行计算，绘制子view等等一系列操作。   

#### 第二次onLayout() ： 
到了第二次onLayout()的时候，要注意，因为在有了第一次onLayout()的过程，ListView现在已经加载好了子项视图了。所以当ListView再次判断子元素是否为空时，现在子元素不再等于0了。所以这次会进行下面这些操作：    

1. ListView首先调用RecycleBin缓存机制的fillActiveViews()方法，将第一次onLayout()已经加载好的视图全部缓存到mActiveViews中，然后再detach掉第一次所有加载好的视图。这样就解决了第二次onLayout()再次加载视图的时候，出现数据重复的问题。    
2. 巧妙的是，在接下来加载子项视图的时候，也是先判断RecycleBin缓存机制中的mActiveViews是否为空，但是因为刚才ListView已经把第一次加载好的子视图全部缓存到了mActiveViews中了，所以此时mActiveViwe并不空，接下来就只要把mActiveViews里面的视图全部attach到ListView上，这样ListView中所有子视图又全部显示出来了。   

---   
   
## Android开发总结之动画(帧动画+补间动画)      
   
[https://blog.csdn.net/songtangxue/article/details/79094413](https://blog.csdn.net/songtangxue/article/details/79094413 "Android动画总结")
  
### 一、概述  
动画的概念   

* 动画的概念不同于一般意义上的动画片，动画是一种综合艺术，它是集合了绘画、漫画、电影、数字媒体、摄影、音乐、文学等众多艺术门类于一身的艺术表现形式。   
* 动画的英文有很多表述，如animation、cartoon、animated cartoon、cameracature。其中较正式的 “Animation” 一词源自于拉丁文字根anima，意思为“灵魂”，动词animate是“赋予生命”的意思，引申为使某物活起来的意思。所以动画可以定义为使用绘画的手法，创造生命运动的艺术。 
* 动画技术较规范的定义是采用逐帧拍摄对象并连续播放而形成运动的影像技术。不论拍摄对象是什么，只要它的拍摄方式是采用的逐格方式，观看时连续播放形成了活动影像，它就是动画。

Android系统中的动画   

* 在Android系统中，动画可分为三类，分别为帧动画(Frame Animation)，补间动画(Tweened Animation)，属性动画。本章主要讲述帧动画和补间动画。   

### 二、动画的实现   
#### 1. 帧动画    

* 帧动画是一种常见的动画形式（Frame By Frame），其原理是在“连续的关键帧”中分解动画动作，也就是在时间轴的每帧上逐帧绘制不同的内容，使其连续播放而成动画。其表现出来的样式类似于我们常见到的GIF图。而帧动画所呈现的结果也依赖于每一“帧”，大家先看效果图：    

![](https://i.imgur.com/BW22yCC.gif)   
   
这个奔跑的京东小人以及跳跃的鱼，其实是一张张不同的图片很快的切换而形成的动画效果。看我的资源文件：   
   
![](https://i.imgur.com/FUkMtSj.png)   
   
我们来看看帧动画的实现，首先上代码：  
  
	<?xml version="1.0" encoding="utf-8"?>
	<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    	android:oneshot="false">
    	<item
    	    android:drawable="@mipmap/a_0"
    	    android:duration="100" />
    	<item
    	    android:drawable="@mipmap/a_1"
    	    android:duration="100" />
    	<item
    	    android:drawable="@mipmap/a_2"
    	    android:duration="100" />
	</animation-list>   
   
这是在res/drawable目录下建立一个的animation-list标签的动画文件，每个item里面有一张图，多张图组合起来就是我们的动画了。而android:duration则是表示每张图呈现的时间长短。还有一个android:oneshot属性，代表是否只展示一遍，true的话该动画只会执行一遍，false则会循环播放动画。   

1.2 Activity里面的代码：   
   
	public class FrameActivity extends AppCompatActivity {
    	private SimpleDraweeView mImgvOne;
    	private ImageView mImgvTwo;
    	private AnimationDrawable animationDrawableOne;
    	private AnimationDrawable animationDrawableTwo;
    	@Override
    	protected void onCreate(Bundle savedInstanceState) {
    	    super.onCreate(savedInstanceState);
    	    setContentView(R.layout.activity_frame);
    	    mImgvOne = (SimpleDraweeView) findViewById(R.id.imgv_one);
    	    mImgvTwo = (ImageView) findViewById(R.id.imgv_two);
    	    setAnimation();

    	}

    	private void setAnimation() {
    	    //Fresco的动画 和ImageView用法一样
    	    mImgvOne.setImageResource(R.drawable.fram_one);
    	    animationDrawableOne = (AnimationDrawable) mImgvOne.getDrawable();
    	    animationDrawableOne.start();
    	    //ImageView的动画
    	    mImgvTwo.setImageResource(R.drawable.fram_two);
    	    animationDrawableTwo = (AnimationDrawable) mImgvTwo.getDrawable();
    	    animationDrawableTwo.start();
    	}
	}    
   
java代码很简单，导入动画开始动画，三行代码就可。大家可以看到我的mImgvOne是一个SimpleDraweeView，因为现在很多人使用Fresco，我只是想证明在使用动画时SimpleDraweeView的用法和ImageView一样，大家不必担心用了Fresco就没法玩动画了。而且基于Fresco的强大基因，想要实现帧动画？人家Fresco是支持gif图的，直接往里面放一个gif图，比我们设置帧动画简单多了。    
  
下面我们再看看AnimationDrawable这个类，我们在执行帧动画时要依靠AnimationDrawable的start方法，先上源码分析一下：   
   
	public class AnimationDrawable extends DrawableContainer implements Runnable, Animatable {
    	public AnimationDrawable() {
    	    throw new RuntimeException("Stub!");
    	}

    	public boolean setVisible(boolean visible, boolean restart) {
    	    throw new RuntimeException("Stub!");
    	}

    	public void start() {
    	    throw new RuntimeException("Stub!");
    	}

    	public void stop() {
    	    throw new RuntimeException("Stub!");
    	}

    	public boolean isRunning() {
    	    throw new RuntimeException("Stub!");
    	}

    	public void run() {
    	    throw new RuntimeException("Stub!");
    	}

    	public void unscheduleSelf(Runnable what) {
    	    throw new RuntimeException("Stub!");
    	}

    	public int getNumberOfFrames() {
    	    throw new RuntimeException("Stub!");
    	}

    	public Drawable getFrame(int index) {
    	    throw new RuntimeException("Stub!");
    	}

    	public int getDuration(int i) {
    	    throw new RuntimeException("Stub!");
    	}

    	public boolean isOneShot() {
    	    throw new RuntimeException("Stub!");
    	}

    	public void setOneShot(boolean oneShot) {
    	    throw new RuntimeException("Stub!");
    	}

    	public void addFrame(Drawable frame, int duration) {
    	    throw new RuntimeException("Stub!");
    	}

    	public void inflate(Resources r, XmlPullParser parser, AttributeSet attrs, Theme theme) throws XmlPullParserException, IOException {
        throw new RuntimeException("Stub!");
    	}

    	public Drawable mutate() {
    	    throw new RuntimeException("Stub!");
    	}

    	protected void setConstantState(DrawableContainerState state) {
    	    throw new RuntimeException("Stub!");
    	}
	}   
   
* 可以看到AnimationDrawable继承了DrawableContainer，而DrawableContainer其实就是Drawable的一个子类，这也是我们animationDrawableTwo = (AnimationDrawable) mImgvTwo.getDrawable();把Drawable强转成AnimationDrawable的原因了。所以AnimationDrawable也是个Drawable罢了。再看它所实现的接口Runnable, Animatable，表明它是一个可执行命令，并且自己支持动画。源码不难，下面简略介绍一下它的相关方法：   

![](https://i.imgur.com/fcVoAsJ.jpg)   
   
源码解析完了，我们也看到了AnimationDrawable的一些方法。下面我们再看看不通过xml文件，直接在代码中执行帧动画的方法。先上代码：  
   
        animationDrawableOne = new AnimationDrawable();
        animationDrawableOne.addFrame(getResources().getDrawable(R.mipmap.a_0),100);
        animationDrawableOne.addFrame(getResources().getDrawable(R.mipmap.a_1),100);
        animationDrawableOne.addFrame(getResources().getDrawable(R.mipmap.a_2),100);
        animationDrawableOne.setOneShot(false);
        mImgvOne.setBackground(animationDrawableOne);
        animationDrawableOne.start();   
   
 同样，使用代码依旧可以做出帧动画的效果，所以大家是使用xml还是代码创建动画，看大家喜好咯。    

#### 2. 补间动画（Tween）   

* 补间动画指的是做flash动画时，在两个关键帧中间需要做“补间动画”，才能实现图画的运动；插入补间动画后两个关键帧之间的插补帧是由计算机自动运算而得到的。也就是说在使用补间动画时，我们开发者指定了动画开始、结束的关键帧，中间的变化是计算机自动帮助我们补齐的。    

* 补间动画有四种基本形式，分别是Alpha（透明度），Translate（位移），Scale（缩放），Rotate（旋转）。当然还可以是多种动画效果的组合，例如alpha+translate、alpha+rotate，甚至四种形式组合在一起的动画。同样，和帧动画一样，补间动画的可以以xml的方式实现，也可以以代码的方式实现。    

