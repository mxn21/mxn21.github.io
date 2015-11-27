---
layout: post
author: mxn
titile: ViewConfiguration使用
category: 技术博文
tag: android
---

### ViewConfiguration类

ViewConfiguration是系统中关于视图的各种特性的常量记录对象。其中包含各种基础数据,在编写自定义控件时会用经常用到。

下面介绍几个常用的方法：

1.在可滑动的控件中用于区别单击子控件和滑动操作的一个伐值。

    {% highlight java %}
mTouchSlop = configuration.getScaledTouchSlop();
   {% endhighlight %}

2.用于设置最小加速率和最大速率.

    {% highlight java %}
mMinimumVelocity = configuration.getScaledMinimumFlingVelocity();
mMaximumVelocity = configuration.getScaledMaximumFlingVelocity();
   {% endhighlight %}

3.滚动距离

    {% highlight java %}
mOverscrollDistance = configuration.getScaledOverscrollDistance();
   {% endhighlight %}
   
4.fling距离

    {% highlight java %}
mOverflingDistance = configuration.getScaledOverflingDistance();
   {% endhighlight %}
 
5.摩擦力，用来计算减速度

    {% highlight java %}
mDeceleration = computeDeceleration(ViewConfiguration.getScrollFriction());
   {% endhighlight %}
   

ViewConfiguration的常量总结：

    {% highlight java %}
/**    
  * 包含了方法和标准的常量用来设置UI的超时、大小和距离    
  */
 public class ViewConfiguration {     
     // 设定水平滚动条的宽度和垂直滚动条的高度，单位是像素px     
     private static final int SCROLL_BAR_SIZE = 10;     
           
     //定义滚动条逐渐消失的时间，单位是毫秒     
     private static final int SCROLL_BAR_FADE_DURATION = 250;     
           
     // 默认的滚动条多少秒之后消失，单位是毫秒     
     private static final int SCROLL_BAR_DEFAULT_DELAY = 300;     
           
     // 定义边缘地方褪色的长度     
     private static final int FADING_EDGE_LENGTH = 12;     
           
     //定义子控件按下状态的持续事件     
     private static final int PRESSED_STATE_DURATION = 125;     
               
     //定义一个按下状态转变成长按状态的转变时间     
     private static final int LONG_PRESS_TIMEOUT = 500;     
               
     //定义用户在按住适当按钮，弹出全局的对话框的持续时间     
     private static final int GLOBAL_ACTIONS_KEY_TIMEOUT = 500;     
               
     //定义一个touch事件中是点击事件还是一个滑动事件所需的时间，如果用户在这个时间之内滑动，那么就认为是一个点击事件     
     private static final int TAP_TIMEOUT = 115;     
               
     /**    
      * Defines the duration in milliseconds we will wait to see if a touch event     
      * is a jump tap. If the user does not complete the jump tap within this interval, it is    
      * considered to be a tap.     
      */
     //定义一个touch事件时候是一个点击事件。如果用户在这个时间内没有完成这个点击，那么就认为是一个点击事件     
     private static final int JUMP_TAP_TIMEOUT = 500;     
           
     //定义双击事件的间隔时间     
     private static final int DOUBLE_TAP_TIMEOUT = 300;     
               
     //定义一个缩放控制反馈到用户界面的时间     
     private static final int ZOOM_CONTROLS_TIMEOUT = 3000;     
           
     /**    
      * Inset in pixels to look for touchable content when the user touches the edge of the screen    
      */
     private static final int EDGE_SLOP = 12;     
               
     /**    
      * Distance a touch can wander before we think the user is scrolling in pixels    
      */
     private static final int TOUCH_SLOP = 16;     
               
     /**    
      * Distance a touch can wander before we think the user is attempting a paged scroll    
      * (in dips)    
      */
     private static final int PAGING_TOUCH_SLOP = TOUCH_SLOP * 2;     
               
     /**    
      * Distance between the first touch and second touch to still be considered a double tap    
      */
     private static final int DOUBLE_TAP_SLOP = 100;     
               
     /**    
      * Distance a touch needs to be outside of a window's bounds for it to    
      * count as outside for purposes of dismissing the window.    
      */
     private static final int WINDOW_TOUCH_SLOP = 16;     
           
    //用来初始化fling的最小速度，单位是每秒多少像素     
     private static final int MINIMUM_FLING_VELOCITY = 50;     
               
     //用来初始化fling的最大速度，单位是每秒多少像素     
     private static final int MAXIMUM_FLING_VELOCITY = 4000;     
           
     //视图绘图缓存的最大尺寸，以字节表示。在ARGB888格式下，这个尺寸应至少等于屏幕的大小     
     @Deprecated     
     private static final int MAXIMUM_DRAWING_CACHE_SIZE = 320 * 480 * 4; // HVGA screen, ARGB8888     
           
     //flings和scrolls摩擦力度大小的系数     
     private static float SCROLL_FRICTION = 0.015f;     
           
     /**    
      * Max distance to over scroll for edge effects    
      */
     private static final int OVERSCROLL_DISTANCE = 0;     
           
     /**    
      * Max distance to over fling for edge effects    
      */
     private static final int OVERFLING_DISTANCE = 4;     
           
 }
      {% endhighlight %}
   
API:14以后提供了方法判断是否有物理按键：ViewConfiguration.hasPermanentMenuKey().
一般来说物理按键和虚拟按键只会有一种。可以用这个方法来判断是否有导航栏NavigationBar和计算NavigationBar的高度。

判断是否有导航栏NavigationBar：

    {% highlight java %}
  public static boolean checkDeviceHasNavigationBar(Context activity) {  
        //通过判断设备是否有返回键、菜单键(不是虚拟键,是手机屏幕外的按键)来确定是否有navigation bar  
        boolean hasMenuKey = ViewConfiguration.get(activity)  
                .hasPermanentMenuKey();  
        boolean hasBackKey = KeyCharacterMap  
                .deviceHasKey(KeyEvent.KEYCODE_BACK);  
  
        if (!hasMenuKey && !hasBackKey) {  
            // 做任何你需要做的,这个设备有一个导航栏  
            return true;  
        }  
        return false;  
    }  
         {% endhighlight %}
            
计算NavigationBar的高度：

    {% highlight java %}
            public int getNavigationBarHeight(Context c) {
                    int result = 0;
                    boolean hasMenuKey = ViewConfiguration.get(c).hasPermanentMenuKey();
                    boolean hasBackKey = KeyCharacterMap.deviceHasKey(KeyEvent.KEYCODE_BACK);
            
                    if (!hasMenuKey && !hasBackKey) {
                        //The device has a navigation bar
                        Resources resources = c.getResources();
                        int orientation = getResources().getConfiguration().orientation;
                        int resourceId;
                        if (isTablet(c)) {
                            resourceId = resources.getIdentifier(orientation == Configuration.ORIENTATION_PORTRAIT ? "navigation_bar_height" : "navigation_bar_height_landscape", "dimen", "android");
                        } else {
                            resourceId = resources.getIdentifier(orientation == Configuration.ORIENTATION_PORTRAIT ? "navigation_bar_height" : "navigation_bar_width", "dimen", "android");
                        }
                        if (resourceId > 0) {
                            return getResources().getDimensionPixelSize(resourceId);
                        }
                    }
                    return result;
                }
    {% endhighlight %}
      
### Configuration类

Configuration类虽然和ViewConfiguration类名字之差一个“view”,但是二者并无继承关系，也没有共同的父类。
Configuration类是专门用来描述手机设备上的配置信息。这些配置信息包括用户特定的配置项，也包括系统的动态设备配置。

程序中可调用Activity的如下方法来获取Configuration对象

    {% highlight java %}
//获取系统的Configuration对象
Configuration cfg = getResources().getConfiguration();
      {% endhighlight %}
      
其中以下的参数代表的配置信息：

* fontScale:获取当前用户设置的字体的缩放因子。

* keyboard：获取当前设备所关联的键盘类型。该属性的返回值：KEYBOARD_12KEY（只有12个键的小键盘）、KEYBOARD_NOKEYS、KEYBOARD_QWERTY（普通键盘）

* keyboardHidden:该属性返回一个boolean值用于标识当前键盘是否可用。该属性不仅会判断系统的硬件键盘，也会判断系统的软键盘（位于屏幕）。

* locale：获取用户当前的Locale.

* mcc:获取移动信号的国家码

* mnc:获取移动信号的网络码

* navigation：判断系统上方向导航设备的类型。该属性的返回值：NAVIGATION_NONAV（无导航）、NAVIGATION_DPAD(DPAD导航）
NAVIGATION_TRACKBALL（轨迹球导航）、NAVIGATION_WHEEL（滚轮导航）

* orientation：获取系统屏幕的方向。该属性的返回值：ORIENTATION_LANDSCAPE（横向屏幕）、ORIENTATION_PORTRAIT（竖向屏幕）

* touchscreen：获取系统触摸屏的触摸方式。该属性的返回值：TOUCHSCREEN_NOTOUCH（无触摸屏）、TOUCHSCREEN_STYLUS（触摸笔式触摸屏）、

* TOUCHSCREEN_FINGER（接收手指的触摸屏）

下面写一个监听屏幕旋转的demo：

MainActivity如下:

    {% highlight java %}
    public class MainActivity extends Activity {
        private Button mButton;
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            System.out.println("---> onCreate()");
            init();
        }
    
        private void init(){
            mButton=(Button) findViewById(R.id.button);
            mButton.setOnClickListener(new ClickListenerImpl());
        }
    
        private class ClickListenerImpl implements View.OnClickListener {
            @Override
            public void onClick(View v) {
                getConfigurationInfo();
            }
    
        }
    
        private void getConfigurationInfo(){
            Configuration configuration=getResources().getConfiguration();
            //获取屏幕方向
            int l=configuration.ORIENTATION_LANDSCAPE;
            int p=configuration.ORIENTATION_PORTRAIT;
            if (configuration.orientation==l) {
                System.out.println("现在是横屏");
            }
            if (configuration.orientation==p) {
                System.out.println("现在是竖屏");
            }
    
            //获取国家码和网络码
            int countryNum=configuration.mcc;
            int netNum=configuration.mnc;
            System.out.println("国家码="+countryNum+",网络码="+netNum);
        }
    
        @Override
        public void onConfigurationChanged(Configuration newConfig) {
            super.onConfigurationChanged(newConfig);
            System.out.println("---> onConfigurationChanged()");
        }
    
        @Override
        protected void onSaveInstanceState(Bundle outState) {
            super.onSaveInstanceState(outState);
            outState.putString("name", "mxn");
            outState.putInt("id", 21);
            System.out.println("---> onSaveInstanceState()");
        }
    
        @Override
        protected void onRestoreInstanceState(Bundle savedInstanceState) {
            super.onRestoreInstanceState(savedInstanceState);
            String name=savedInstanceState.getString("name");
            int id=savedInstanceState.getInt("id");
            System.out.println("---> onRestoreInstanceState()");
            System.out.println("名字="+name+",编号="+id);
        }
    }
    {% endhighlight %}

main.xml如下:

    {% highlight xml %}
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >

    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="获取Configuration信息"
        android:textSize="25sp"
        android:layout_marginTop="80dip"
        android:layout_centerHorizontal="true"
        />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="测试ConfigurationChange"
        android:textSize="25sp"
        android:layout_centerInParent="true"
        />

</RelativeLayout>  

    {% endhighlight %}
    
AndroidManifest.xml如下:
   
    {% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.mxn.soul.demo" >

    <application
        android:allowBackup="true"
        android:icon="@mipmap/icon"
        android:label="@string/app_name"
        android:name=".BaseApplication"
        android:theme="@style/AppTheme" >
        <activity
            android:name=".MainActivity"
            android:configChanges="keyboardHidden|orientation|screenSize"
            android:label="@string/app_name" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
    {% endhighlight %}
    
1.点击Button利用Configuration查看手机上的配置信息 

    {% highlight c %}
11-27 11:56:41.763 4093-4093/com.mxn.soul.demo I/System.out: 现在是横屏
11-27 11:56:41.763 4093-4093/com.mxn.soul.demo I/System.out: 国家码=0,网络码=0
    {% endhighlight %}
    
2.在manifest为Activity配置configChanges时，旋转屏幕，输出如下：

    {% highlight c %}
11-27 11:59:26.827 8684-8684/com.mxn.soul.demo I/System.out: ---> onCreate()
11-27 11:59:31.110 8684-8684/com.mxn.soul.demo I/System.out: ---> onConfigurationChanged()
    {% endhighlight %}
    
由于设备和系统版本的差异,少数情况下设置android:configChanges="orientation" 无效,
建议设置为：android:configChanges="keyboardHidden|orientation|screenSize" ，表示当前Activity可以对屏幕是否旋转进行监听
(当然也可对其他系统信息进行监听) 配置后屏幕旋转时会调用onConfigurationChanged()方法. 

注意：当onConfigurationChanged发生之后，并不会对activiyt的生命周期有影响的，也就是说当横竖修改之后，android
并不会有任何生命周期的变化。同理还有（键盘显示或者隐藏、用户的语言设置、用户字体修改、对键盘类型修改、键盘导航修改）
这些都可以触发相关的activity的事件产生。

android:configChanges配置说明：

Android通过终止、重启应用程序来重新加载资源文件，以做到对语言、区域和硬件实时变化的支持。它的默认的行为不是总是方便和令人满意的，
尤其当配置变化（如屏幕方向和键盘可视）、用户旋转设备或划出键盘等。你可以通过监测和响应定制你的应用程序来对这些变化作出响应。
为了能让Activity能监听实时的配置变化，需要在manifest节点里添加“android:configChanges”特性，指定你要处理的配置变化事件。
接下来的列表给出了你可以指定的配置变化的事件值：

*orientation 屏幕在纵向和横向间旋转。
*keyboardHidden 键盘显示或隐藏。
*fontScale 用户变更了首选的字体大小。
*locale 用户选择了不同的语言设定。
*keyboard 键盘类型变更，例如手机从12键盘切换到全键盘
*ouchscreen或navigation 键盘或导航方式变化，一般不会发生这样的事件。

你可以选择捕获多个事件，通过在各事件值间使用“|”。
override onConfigurationChanged这个方法去捕获配置变化，通过传入的Configuration 对象的值进行合适的处理。
需要提醒的是需要调用父类的方法super.onConfigurationChanged(newConfig);并且要重新加载Activity使用的资源，以防有变更。

当onConfigurationChanged 被调用时，Activity的资源变量都已经用新的值进行了更新，所以它们是安全的。任何你没有显式指明的配置变化事件，
都将由应用程序捕获，而且仍然会引起应用程序的重启，而不会调用onConfigurationChanged 方法。

3.若不配置android:configChanges,那么每次屏幕旋转的时候都会调用Activity的onCreate()方法 而不会调用onConfigurationChanged()。取消在2中的设置
.此时每次旋转屏幕都会调用onCreate(),并在屏幕旋转前调用onSaveInstanceState()保存现场状态,在选中后调用onRestoreInstanceState()
恢复现场. 所以此时调用顺序为: onSaveInstanceState->onCreate()->onRestoreInstanceState()。除此以外:
当系统内存紧张时可暂时杀死该Activity,内存允许时重启该Activity.在这样情况下也是该调用顺序,原理亦类似. 

输出如下：

    {% highlight c %}
11-27 12:04:07.508 13914-13914/com.mxn.soul.demo I/System.out: ---> onCreate()
11-27 12:04:11.400 13914-13914/com.mxn.soul.demo I/System.out: ---> onSaveInstanceState()
11-27 12:04:11.464 13914-13914/com.mxn.soul.demo I/System.out: ---> onCreate()
11-27 12:04:11.465 13914-13914/com.mxn.soul.demo I/System.out: ---> onRestoreInstanceState()
11-27 12:04:11.465 13914-13914/com.mxn.soul.demo I/System.out: 名字=mxn,编号=21
    {% endhighlight %}
    
另外还有一个判断是否是平板的方法如下（官方用法）：

    {% highlight java %}
  public static boolean isTablet(Context context) {
        return (context.getResources().getConfiguration().screenLayout & Configuration.SCREENLAYOUT_SIZE_MASK) >= Configuration.SCREENLAYOUT_SIZE_LARGE;
    }
    {% endhighlight %}
    
