## Android 打包过程   
   
### RecycleBin缓存机制的工作原理：   
   
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

