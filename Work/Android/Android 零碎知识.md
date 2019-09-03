# Android 零碎知识

#### 1，adb 隐藏系统栏 
immersive.full：全屏      
immersive.status：隐藏状态栏   
immersive.navigation：隐藏导航栏
* 隐藏 App 导航栏
   ```
   adb shell settings put global policy_control immersive.navigation=package1, package2
   ```
* 撤销 App 操作
   ```
   adb shell settings put global policy_control immersive.off=package1, package2
   ```
* 隐藏所有 App 导航栏，除了 package1
  ```
  adb shell settings put global policy_control immersive.navigation=apps, -package1
  ```
* 撤销全部操作
  ```
  adb shell settings put global policy_control null
  ```

#### 2，FileProvider 

对于面向 Android 7.0 的应用，Android 框架执行的 [StrictMode](https://link.jianshu.com?t=https://developer.android.com/reference/android/os/StrictMode.html)API 政策禁止在您的应用外部公开 file:// URI。如果一项包含文件 URI 的 intent 离开您的应用，则应用出现故障，并出现 FileUriExposedException 异常 。
* 最简单粗暴的就是将 targetSdkVersion 改成24以下（RE管理器就是23）
* 在Android 7.0+ 上使用 `FileProvider.getUriForFile 将 file://Uri 转换成 content://Uri 。
* 利用反射忽略 Android 7.0+ 的 content://uri 检查：
   ```
   val method: Method = 
    StrictMode::class.java.getDeclaredMethod("disableDeathOnFileUriExposure")
   method.invoke("")
   ```
>Android开发者官网：
>[FileProvider](https://developer.android.com/reference/android/support/v4/content/FileProvider.html)


#### 3，同一 Activity 的多个实例以任务的形式显示在概览屏幕中

* 仅仅设置Intent.[FLAG_ACTIVITY_NEW_DOCUMENT](https://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_NEW_DOCUMENT)，系统将 Activity 视为新任务显示在概览屏幕中；当主 Activity 启动新 Activity 时，系统会搜遍现有任务，看看是否有任务的 Intent 与 Activity 的 Intent 组件名称和 Intent 数据（`intent.setData()`）相匹配。 如果未找到任务，则会以该 Activity 作为其根创建新任务。如果找到的话，则会将该任务转到前台并将新 Intent 传递给 onNewIntent() 。
* 当一起设置 Intent.[FLAG_ACTIVITY_NEW_DOCUMENT](https://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_NEW_DOCUMENT)
  和 [FLAG_ACTIVITY_MULTIPLE_TASK](https://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_MULTIPLE_TASK) 标志时，系统始终会以目标 Activity 作为根创建新任务（类似 `launchMode="standard"` 模式）。
* 使用 AppTask 类移除任务：在于概览屏幕创建新任务的 Activity 中，您可以通过调用 [finishAndRemoveTask()](https://developer.android.com/reference/android/app/ActivityManager.AppTask.html#finishAndRemoveTask()) 方法将任务从概览屏幕中删除。

**实现类似微信小程序单独后台卡片显示：**
```java
val intent = Intent(activity, MDViewActivity::class.java)
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_DOCUMENT)  // 以后台任务列表的形式打开 
intent.data = Uri.fromFile(File(file))  // 设置不同 Uri 数据实现不同文档以单独任务卡片展示。
activity.startActivity(intent)
// 默认动画像在两个App之间跳转，所以加上自定义跳转动画实现在同一个应用跳转的效果。
activity.overridePendingTransition(R.anim.slide_right_in, R.anim.slide_left_out)
```
>Android开发者官网：
>[概览屏幕](https://developer.android.com/guide/components/recents.html)


#### 4，更改最近任务列表的 Title、Icon 和 Color

```java
TaskDescription td = new TaskDescription(mTitle, mIcon, mColor);
activity.setTaskDescription(td);
```


#### 5，系统状态栏导航栏相关

```java
// Android 5.0+ 全屏显示 
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
   // <item name="windowActionBar">false</item> & <item name="windowNoTitle">true</item>     
   // android:fitsSystemWindows="false"
   supportActionBar?.hide()
   decorView.systemUiVisibility = View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN  or View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION   
   window.statusBarColor = Color.TRANSPARENT    
   window.navigationBarColor = Color.TRANSPARENT
}
// Android 6.0+ 灰色状态栏图标
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
   var suv = decorView.systemUiVisibility 
   suv = suv or View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR
   decorView.systemUiVisibility =  suv
}
// Android 8.0+ 亮色导航栏
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
   var suv = decorView.systemUiVisibility
   suv = suv or View.SYSTEM_UI_FLAG_LIGHT_NAVIGATION_BAR
   decorView.systemUiVisibility = suv
   window.navigationBarColor = getColor(android.R.color.transparent)
}
//诺基亚Rom
getWindow().setNavigationBarColor(ContextCompat.getColor(this, android.R.color.white));
```

```java
1, statusBarColor (API level 21)
   setStatusBarColor (API level 21)

2, windowLightStatusBar (API level 23)
   SYSTEM_UI_FLAG_LIGHT_STATUS_BAR (API level 23)

3, navigationBarColor (API level 21)
   setNavigationBarColor (API level 21)

4, windowLightNavigationBar (API level 27)
   SYSTEM_UI_FLAG_LIGHT_NAVIGATION_BAR (API level 26)

5, navigationBarDividerColor (API level 27)
   setNavigationBarDividerColor (API level P)      
```     
```java    
/**
 * Created by xiaoyanger on 2017/3/1.
 * 沉浸式、版本兼容的Toolbar，状态栏透明.
 */
public class CompatToolbar extends Toolbar {

    public CompatToolbar(Context context) {
        this(context, null);
    }

    public CompatToolbar(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CompatToolbar(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        setup();
    }

    public void setup() {
        int compatPadingTop = 0;
        // android 4.4以上将Toolbar添加状态栏高度的上边距，沉浸到状态栏下方
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            compatPadingTop = getStatusBarHeight();
        }
        this.setPadding(getPaddingLeft(), getPaddingTop() + compatPadingTop, getPaddingRight(), getPaddingBottom());
    }
    
    public int getStatusBarHeight() {
        int statusBarHeight = 0;
        int resourceId = getResources().getIdentifier("status_bar_height", "dimen", "android");
        if (resourceId > 0) {
            statusBarHeight = getResources().getDimensionPixelSize(resourceId);
        }
        Log.d("CompatToolbar", "状态栏高度：" + px2dp(statusBarHeight) + "dp");
        return statusBarHeight;
    }

    public float px2dp(float pxVal) {
        final float scale = getContext().getResources().getDisplayMetrics().density;
        return (pxVal / scale);
    }
}       

```

#### 6，16进制颜色透明度动态
```Java
/** Adds alpha to a hex color
  * @param originalColor color, without alpha
  * @param alpha from 0.0 to 1.0
  * @return the original color with alpha
  * */
fun getColorWithAlpha(originalColor: String, alpha: Double): String {
   val alphaFixed = Math.round(alpha * 255)
   var alphaHex = java.lang.Long.toHexString(alphaFixed)
   if (alphaHex.length == 1) {
      alphaHex = "0" + alphaHex
   }
   return originalColor.replace("#", "#" + alphaHex)
}
```


#### 7，RecyclerView恢复浏览位置

```java
// 要恢复的界面的第一条 item 位置
val position = layoutManager.findFirstVisibleItemPosition()
val view = layoutManager.findViewByPosition(position)
if (view != null) {
   // 第一条 item 位置距顶部距离
   val top = view.top 
}

// 恢复，立即恢复无动画无感觉
layoutManager.scrollToPositionWithOffset(position, top)
```
>参考：
>[RecyclerView恢复浏览位置一些事](https://www.jianshu.com/p/f6c63a59d511)


#### 8，java.lang.ExceptionInInitializerError异常

此异常是 **静态初始化程序中发生意外异常的信号**
```java
public class A{
    private static A  a = new A();
    private static HashMap<String, String>() b = new HashMap<String, String>();
    private A(){
        b.put("key", "value");
    }
    public static A getInstance(){
        return a;
    }
}
```
**原因：**当此类单例调用 `getInstance()` 时，类A开始加载，而静态代码块和静态变量是在类的加载时进行初始化，是所有类对象所共享的，静态变量的加载顺序是按它们在源文件中声明的顺序来进行，所以类A加载，就开始加载静态变量a，接着调用a的构造，直接调用 `b.put("key", "value");` ，但是静态变量b还没有开始初始化，所以就出现了此错误。
**解决：**解决此错误只需要将上面静态变量定义的位置对换一下就可以了。
**注意：**因为 ExceptionInInitializerError 导致了类无法加载，由于类加载失败了，因此JVM会抛出 NoClassDefFoundError ，在分析 NoClassDefFoundError 的原因，最好看下日志文件中有没有 ExceptionInInitializerError ，然后再考虑要不要检查 classpath。

#### 9，从demens.xml文件获取长度值

getDimension()、getDimensionPixelSize()和getDimenPixelOffset()的结果值都是实际像素值px；
getDimension()：返回实际数值；
getDimensionPixelSize()：返回的是实际数值的四舍五入；
getDimensionPixelOffset()：返回的是实际数值去掉后面的小数点。

#### 10，View随ScrollView滑动透明度变化

```java
public class TranslucentScrollView extends ScrollView {

    //渐变的视图
    private View mAlphaView;
    //渐变视图高度
    private int alphaViewHeight = 0; 
    /**
     * @param alphaView       透明变化的视图
     * @param alphaViewHeight 透明变化视图高度
     */
    public void setTransView(View alphaView, int alphaViewHeight) {
        mAlphaView = alphaView;
        //初始视图-透明
        mAlphaView.setAlpha(0);
        mAlphaView.setEnabled(false);
        this.alphaViewHeight = alphaViewHeight;
    }
    /**
     * 获取透明度比例
     * @return
     */
    private float getTransAlpha() {
        float scrollY = getScrollY();
        float offset = 1 - Math.max((alphaViewHeight - scrollY) / alphaViewHeight, 0f);
        return Math.abs(offset);
    }
    @Override
    protected void onScrollChanged(int l, int t, int oldl, int oldt) {
        super.onScrollChanged(l, t, oldl, oldt);
        if (mAlphaView != null) {
            float alpha = getTransAlpha();
            mAlphaView.setAlpha(alpha);
            mAlphaView.setEnabled(alpha > 0.1);
        }
    }
}
```
也可以使用CoordinatorLayout配合layout_behavior实现

#### 11，ScrollView嵌套RecyclerView滑动冲突

简单方法一：
```java
LinearLayoutManager manger = new LinearLayoutManager(getActivity()){
   @Override
   public boolean canScrollVertically() {
       return false;
   }
};
```


#### 12，RecyclerView使用LinearLayoutManager时，item的match_parent不起作用

解决方法：
`View itemView = inflater.inflate(R.layout.item_swipe_layout, parent, false);`
原因：
```java
if (root != null) {
   if (DEBUG) {
       System.out.println("Creating params from root: " +
               root);
   }
   // Create layout params that match root, if supplied
   params = root.generateLayoutParams(attrs);
   if (!attachToRoot) {
       // Set the layout params for temp if we are not
       // attaching. (If we are, we use addView, below)
       temp.setLayoutParams(params);
   }
}
//看源码得知当你传入的root不为null时才会setLayoutParams，所以item的match_parent不生效
```


#### 13，自绘形状的水波纹

```xml
//btn_ripple.xml
<?xml version="1.0" encoding="utf-8"?>
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
        android:color="?android:attr/colorControlHighlight">
    <item android:id="@android:id/mask">
        <shape android:shape="rectangle">
            <solid android:color="@color/black"/>
            <corners android:radius="4dp"/>
        </shape>
    </item>
    <item android:drawable="@drawable/btn_border"/>
</ripple>

//btn_border.xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
       android:shape="rectangle">
    <corners android:radius="4dp"/>
    <stroke
        android:width="@dimen/border_width"
        android:color="@color/accent"/>
</shape>
```


#### 14，SpannableStringBuilder单TextView多样式

```Java
//添加文字删除线
public static SpannableStringBuilder getStrikethroughString(String str) {
    SpannableStringBuilder strikethroushStr = new SpannableStringBuilder(str);
    strikethroushStr.setSpan(
        new StrikethroughSpan(),
        0,
        str.length(),
        Spanned.SPAN_INCLUSIVE_INCLUSIVE);
    return strikethroushStr;
}
textView.setText(getStrikethroughString("测试文字"));
```
> 参考：
>
> [【Android】强大的SpannableStringBuilder](https://www.jianshu.com/p/f004300c6920)

#### 15，不同语言单复数 getQuantityString()

```java
textView.setText(getResources().getQuantityString(
                    R.plurals.choose_day, days, days));

//string.xml
<plurals name="choose_day">
        <item quantity="one"><xliff:g id="count_Single">"%1$d day "</xliff:g></item>
        <item quantity="other"><xliff:g id="count_Plural">"%1$d days "</xliff:g></item>
</plurals>
```


#### 16，ScrollView 嵌套 RecyclerView 时，自动显示到 RecyclerView 的位置

原理：
1，ScrollView 会滑到获取焦点的子 view 的位置；
2，RecyclerView 的 focusableOnTouchMode 属性默认是 true；
解决：
1，ScrollView 第一个子 View 设置 `android:focusableInTouchMode="true"`；
2，ScrollView 的直接子 ViewGroup 设置`android:descendantFocusability=”blocksDescendants”`属性；
3，自定义 ScrollView ，页面隐藏时记录滑动位置，显示时滑动到上一次保存的位置；
其它：
1，`android:focusableInTouchMode="true"`：可以通过触摸获取焦点；
2，`android:descendantFocusability=”blocksDescendants”`：Viewgroup 会覆盖子类控件而直接获得焦点；
>参考：
>[android:focusableInTouchMode为什么能解决ScrollView自动滚动的原理分析](https://segmentfault.com/a/1190000011509975)


#### 17，Log信息过长打印不全

```java
/**
 * 信息太长,分段打印
 *
 * @param tag
 * @param msg
 */
 public static void e(String tag, String msg) {
    //因为String的length是字符数量不是字节数量所以为了防止中文字符过多，
    //  把4*1024的MAX字节打印长度改为2001字符数
    int max_str_length = 2001 - tag.length();
    //大于4000时
    while (msg.length() > max_str_length) {
       Log.e(tag, msg.substring(0, max_str_length));
       msg = msg.substring(max_str_length);
    }
    //剩余部分
    Log.e(tag, msg);
}
```


#### 18，打开联系人选择列表

```java
@OnClick(R.id.iv_contact)
public void onViewClicked() {
    Intent intent = new Intent(Intent.ACTION_PICK, ContactsContract.CommonDataKinds.Phone.CONTENT_URI);
    startActivityForResult(intent, PICK_CONTACT_REQUEST);
}
@Override
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    // If the request went well (OK) and the request was PICK_CONTACT_REQUEST
    if (resultCode == Activity.RESULT_OK && requestCode == PICK_CONTACT_REQUEST) {
        if (data != null) {
            Uri uri = data.getData();
            if (uri != null) {
                Cursor cursor = getActivity().getContentResolver().query(uri, null, null, null, null);
                String num = "", name = "";
                if (cursor != null && cursor.moveToNext()) {
                    name = cursor.getString(cursor.getColumnIndex(ContactsContract.Contacts.DISPLAY_NAME));
                    num = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
                   //替换掉不合法的字符
                   num = num.replace("-", "");
                   num = num.replace(" ", "");
                   cursor.close();
                 }
                 Log.e(">>>>>>>>", "onActivityResult:" + name + "-" + num);
             }
         }
     }
}
```
打开的这个联系人选择界面是系统默认会提供的，通常这个界面被称为 Android Contact Picker 。并且该界面会列出所有有号码的联系人，包括一个联系人下有多个号码。

#### 19，隐藏与显示软键盘

```java
/**
 * 隐藏软键盘
 */
public static void hideSoftInput(View view) {
    InputMethodManager imm = (InputMethodManager) view.getContext().getSystemService(Context.INPUT_METHOD_SERVICE);
    if (imm != null) {
       imm.hideSoftInputFromWindow(view.getWindowToken(), 0);
    }
}
/**
 * 显示软键盘
 */
public static void showSoftInput(View view) {
    InputMethodManager imm = (InputMethodManager) view.getContext().getSystemService(Context.INPUT_METHOD_SERVICE);
    if (imm != null) {
       imm.showSoftInput(view, 0);
    }
}
```
>参考
>
>[Android 软键盘的显示和隐藏，这样操作就对了](https://segmentfault.com/a/1190000012279204)


#### 20，禁止App字体大小随系统设置改变而改变

```java           
// BaseActivity
@Override
protected void attachBaseContext(Context newBase) {
    Context targetContext;
    final Resources res = newBase.getResources();
    final Configuration configuration = res.getConfiguration();
    // 防止字体大小修改
    configuration.fontScale = 1;
    // 防止显示大小修改
    DisplayMetrics displayMetrics = res.getDisplayMetrics();
    final int densityStable;
    if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.N) {
        densityStable = DisplayMetrics.DENSITY_DEVICE_STABLE;
    } else {
        Log.w(LOG_TAG, "SDK_INT < N, so can't get DENSITY_DEVICE_STABLE");
        densityStable = 0;
    }
    configuration.densityDpi = densityStable > DisplayMetrics.DENSITY_MEDIUM ?
            densityStable : UIUtil.getMaxDensityDpi(displayMetrics.widthPixels);
    targetContext = newBase.createConfigurationContext(configuration);
    super.attachBaseContext(targetContext);
}
// 弹框影响
public class FixedAlterDialog extends AlertDialog {
    protected FixedAlterDialog(Context context) {
        super(context);
    }
    public static class Builder extends AlertDialog.Builder {
        private Context mContext;
        public Builder(Context context) {
            super(context);
            mContext = context;
        }
        @Override
        public AlertDialog show() {
            AlertDialog dialog = super.show();
            WindowManager windowManager = (WindowManager) mContext.getSystemService(Context.WINDOW_SERVICE);
            if (windowManager != null) {
                Display display = windowManager.getDefaultDisplay();
                Window window = dialog.getWindow();
                if (window != null) {
                    WindowManager.LayoutParams lp = window.getAttributes();
                    lp.width = display.getWidth();
                    dialog.getWindow().setAttributes(lp);
                }
            }
            return dialog;
        }
    }
}
```    
#### 21， clipToPadding 和 clipChildren     
* clipToPadding    
对于 padding 所占的尺寸大小也绘制其他的 item 的 view；    
当我们为 ListView 等设置了 paddingTop 或 paddingBottom 的时候，当滑动到顶部和底部的时候，默认情况下 padding/margin 在滑动中一直存在，view 总是不能滑动到最底部和最顶部，设置 false 时列表滑动可占据 padding 的位置。    
    
* clipChildren    
表示允许子 view 超越父布局的边界；且外，layout_gravity="bottom" ，告知系统要从底部向上绘制该子 view。     

#### 22，透明度对应16进制      
|  透明度  |  16进制 |  透明度  | 16进制  |  透明度 | 16进制  |  透明度  |  16进制 |  透明度 |  16进制 |    
|  -----  | ------ |  -----  | ------ |   ----- | ------ |  -----  | ------ |  -----  | ------ |
|   0%    |   00   |   20%   |   33   |   40%   |   66   |   60%   |   99   |   80%   |   CC   |
|   1%    |   03   |   21%   |   36   |   41%   |   69   |   61%   |   9C   |   81%   |   CF   |
|   2%    |   05   |   22%   |   38   |   42%   |   6B   |   62%   |   9E   |   82%   |   D1   |
|   3%    |   08   |   23%   |   3B   |   43%   |   6E   |   63%   |   A1   |   83%   |   D4   |
|   4%    |   0A   |   24%   |   3D   |   44%   |   70   |   64%   |   A3   |   84%   |   D6   |
|   5%    |   0D   |   25%   |   40   |   45%   |   73   |   65%   |   A6   |   85%   |   D9   |
|   6%    |   0F   |   26%   |   42   |   46%   |   75   |   66%   |   A8   |   86%   |   DB   |
|   7%    |   12   |   27%   |   45   |   47%   |   78   |   67%   |   AB   |   87%   |   DE   |
|   8%    |   14   |   28%   |   47   |   48%   |   7A   |   68%   |   AD   |   88%   |   E0   |
|   9%    |   17   |   29%   |   4A   |   49%   |   7D   |   69%   |   B0   |   89%   |   E3   |
|   10%   |   1A   |   30%   |   4D   |   50%   |   80   |   70%   |   B3   |   90%   |   E6   |
|   11%   |   1C   |   31%   |   4F   |   51%   |   82   |   71%   |   B5   |   91%   |   E8   |
|   12%   |   1F   |   32%   |   52   |   52%   |   85   |   72%   |   B8   |   92%   |   EB   |
|   13%   |   21   |   33%   |   54   |   53%   |   87   |   73%   |   BA   |   93%   |   ED   |
|   14%   |   24   |   34%   |   57   |   54%   |   8A   |   74%   |   BD   |   94%   |   F0   |
|   15%   |   26   |   35%   |   59   |   55%   |   8C   |   75%   |   BF   |   95%   |   F2   |
|   16%   |   29   |   36%   |   5C   |   56%   |   8F   |   76%   |   C2   |   96%   |   F5   |
|   17%   |   2B   |   37%   |   5E   |   57%   |   91   |   77%   |   C4   |   97%   |   F7   |
|   18%   |   2E   |   38%   |   61   |   58%   |   94   |   78%   |   C7   |   98%   |   FA   |
|   19%   |   30   |   39%   |   63   |   59%   |   96   |   79%   |   C9   |   99%   |   FC   |       

* 计算方法：      
```
for (int i = 0; i <= 100; i++) {
   float temp = 255 * i * 1.0f / 100f;
   int alpha = Math.round(temp);
   String hexStr = Integer.toHexString(alpha);
   if (hexStr.length() < 2)
      hexStr = "0" + hexStr;
   System.out.println(i + "%, " + hexStr.toUpperCase());
}     
```    
#### 22，Webview 设置背景颜色     
```   
<WebView
   android:id="@+id/webView"
   android:layout_width="match_parent"
   android:layout_height="match_parent"/>   
    
webView.setBackgroundColor(Color.parseColor("#000000"));    
```


​      

​      


欢迎提 Issues 😊

​      

© 著作权归作者所有，转载请联系作者获得授权。
