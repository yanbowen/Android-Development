# Android 开发艺术探索  
---
## 第一章  
### Activity 的生命周期  
* 当启动一个新的Activity时，旧Activity的onPause方法会先执行，然后才会启动新的Activity的onCreate，onStart，onResume；  
![](http://i.imgur.com/KGtVEmo.png)   
* Activity在异常状态下终止，系统会调用onSaveInstanceState方法来保存当前Activity的状态，这个方法的调用时机是在onStop之前，它和onPause没有既定的时序关系。
* 每个View都有OnSaveInstanceState和onRestoreInstanceState这个两个方法，Activity重建后系统会调用onRestoreInstanceState，获取到保存的Bundle对象。  
![](http://i.imgur.com/5keSn4K.png)   
* 当系统配置发生变化后，Activity会被重启，若不想重启，可设置Activity的ConfigChanges属性。   
![](http://i.imgur.com/vKiXWn8.png)  
### Activity的启动模式  
#### 给Activity指定启动模式有两种方法：   
##### 第一种通过AndroidMenifest  
![](http://i.imgur.com/pGNfrCL.png)  
##### 第二种通过Intent中设置标志位  
![](http://i.imgur.com/521Ao98.png)   

* 第二种方法的优先级要高于第一种  
#### Activity启动4种模式：  
* 1. standard：标准模式
* 2. singleTop:栈顶复用模式
* 3. singleTask:栈内复用模式---singleTask模式的Activity切换到栈顶会导致它之上的栈内的Activity出栈。
* 4. singleInstance：单实例模式   

通过执行`adb shell dumpsys activity`命令：
![](http://i.imgur.com/AkTeGgv.png)  

* 可以看出去掉singleTask后，连续点击3次，出现了2个任务栈，前台任务栈是第一个红框，里面有4个Activity，后台任务栈是第二个红框，是Android桌面。  
#### 常见Activity的Flags：  
* FLAG\_ACTIVITY\_NEW\_TASK : 这个标记作用为Activity指定singleTask启动模式   
* FLAG\_ACTIVITY\_SINGLE\_TOP : 这个标记作用为Activity指定singleTop启动模式   
* FLAG\_ACTIVITY\_CLEAR_\TOP : 同一个任务栈中所有位于它上面的Activity都要出栈，这个标记一般会和singleTask启动模式一起出现，在这样的情况下，被启动Activity实例如果存在，那么系统会调用它的onNewIntent。如果被启动的Activity采用standard模式启动，那么它连同它之上的Activity都要出栈，系统会创建新的Activity实例并放入栈顶。  
* FLAG\_ACTIVITY\_EXCLUDE\_FROM\_RECENTS : 具有这个标记的Activity不会出现在历史Activity列表中，当某些情况下我们不希望用户通过历史列表回到我们的Activity的时候这个标记比较有用，等同于XML中指定Activity的属性`android:excludeFromRecents="true"`  

#### IntentFilter的匹配规则： 
隐式启动需要Intent能够匹配目标组件的IntentFilter中所设置的过滤信息，需要同时匹配过滤列表中的action、category、data信息，否则匹配失败。另外一个Activity中可以有多个intent-filter,一个Intent只要能匹配任何一组intent-filter即可成功启动Activity  

* action是一个字符串，Intent中的action能够和过滤规则中的任何一个action相同即可匹配成功。
* Intent可以没有category，但如果一旦有，不管有几个，每个都要能够和过滤规则中的任何一个category相同
* data由两部分组成，mimeType和URI。mimeType指媒体类型，比如image、JPEG、video等  
1.显示启动：用于启动同一个应用中的Activity，效率高  
2.隐式启动：用于启动不同应用中的Activity