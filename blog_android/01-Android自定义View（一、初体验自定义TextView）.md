> 版权声明：本文为openXu原创文章[【openXu的博客】](http://blog.csdn.net/xmxkf)，未经博主允许不得以任何形式转载

> 由于CSDN-Markdown改版私自替换改动博主文章内容格式，导致博主CSDN全部博客排版混乱，外加CSDN工作人员傲慢的姿态，本人决定择时将博客全部移到GitHub作为备份，后期新增文章也会在此发布，CSDN博客放弃支持，谢谢大家

[TOC]

对于一个Android攻城狮来说，自定义控件是一项必须掌握的重要技能点，然而对于大部分人而言，感觉自定义控件并不是那么容易。在工作过程中难免遇到一些特效需要自己定义控件实现，如果你不会，内心会有强烈的挫败感，这对一个程序员来说是决不能容忍的，接下来我将写一系列博客，和大家一起学习自定义控件，让她赤裸裸的站在我们的面前，让我们为所欲为... :joy:
言归正传，接触到一个类，你不太了解他，如果贸然翻阅源码只会让你失去方向，不知从哪里下手；所以我们应该从文档着手，看看它是个什么东西，里面有哪些属性和方法，都是用来干嘛的。下面我们看看官方文档对View的介绍：

> View这个类代表用户界面组件的基本构建块。View在屏幕上占据一个矩形区域，并负责绘制和事件处理。View是用于创建交互式用户界面组件（按钮、文本等）的基础类。它的子类ViewGroup是所有布局的父类，它是一个可以包含其他view或者viewGroup并定义它们的布局属性的看不见的容器。
实现一个自定义View，你通常会覆盖一些framework层在所有view上调用的标准方法。你不需要重写所有这些方法。事实上，你可以只是重写onDraw(android.graphics.Canvas)。


| 分类    | 方法  | 说明 |
| :------------:|:---------------:|:-----|
| 构造     | Constructors |有一种形式的构造方法是使用代码创建的时候调用的，另一种形式是View被布局文件填充时被调用的。第二种形式应该解析和使用一些属性定义在布局文件中|
|   |   `onFinishInflate()`    |   当View和他的所有子控件被XML布局文件填充完成时被调用。（这个方法里面可以完成一些初始化，比如初始化子控件）|
| 布局 | `onMeasure(int, int)` |当决定view和他的孩子的尺寸需求时被调用（也就是测量控件大小时调用） |
|      | `onLayout(boolean, int, int, int, int)`       |   当View给他的孩子分配大小和位置的时候调用（摆放子控件）|
|  |`onSizeChanged(int, int, int, int)` |  当view大小发生变化时调用|
| 绘制 |`onDraw(android.graphics.Canvas)`|  当视图应该呈现其内容时调用（绘制） |
|事件处理 | `onKeyDown(int, KeyEvent)` | 按键时被调用 |
| |`onKeyUp(int, KeyEvent)`|   按键被抬起时调用|
|  |`onTrackballEvent(MotionEvent)` |  Called when a trackball motion event occurs. |
|  |`onTouchEvent(MotionEvent)`| 触摸屏幕时调用|
|焦点 | `onFocusChanged(boolean, int, android.graphics.Rect)`|  获取到或者失去焦点是调用|
| |`onWindowFocusChanged(boolean)` |窗口获取或者失去焦点是调用|
|Attaching |`onAttachedToWindow()`| 当视图被连接到一个窗口时调用|
| |`onDetachedFromWindow()`|当视图从窗口分离时调用|
| |`onWindowVisibilityChanged(int)`| 当View的窗口的可见性发生改变时调用|

从上面官方文档介绍我们可以知道，View是所有控件（包括ViewGroup）的父类，它里面有一些常见的方法（上表），如果我们要自定义View，最简单的只需要重写onDraw(android.graphics.Canvas)即可，听起来是不是很简单？那我们就动手自定义一个属于自己的TextView吧。

## 1. 继承View，重写onDraw方法

创建一个类MyTextView继承View，发现报错，因为要覆盖他的构造方法（因为View中没有参数为空的构造方法），View有四种形式的构造方法，其中四个参数的构造方法是API 21才出现，所以一般我们只需要重写其他三个构造方法即可。它们的参数不一样分别对应不同的创建方式，比如只有一个Context参数的构造方法通常是通过代码初始化控件时使用；而两个参数的构造方法通常对应布局文件中控件被映射成对象时调用（需要解析属性）；通常我们让这两个构造方法最终调用三个参数的构造方法，然后在第三个构造方法中进行一些初始化操作。

```Objective-c
public class MyView extends View {
    /**
     * 需要绘制的文字
     */
    private String mText;
    /**
     * 文本的颜色
     */
    private int mTextColor;
    /**
     * 文本的大小
     */
    private int mTextSize;
    /**
     * 绘制时控制文本绘制的范围
     */
    private Rect mBound;
    private Paint mPaint;

    public MyTextView(Context context) {
        this(context, null);
    }
    public MyTextView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }
    public MyTextView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
 //初始化
        mText = "Udf32fA";
        mTextColor = Color.BLACK;
        mTextSize = 100;

        mPaint = new Paint();
        mPaint.setTextSize(mTextSize);
        mPaint.setColor(mTextColor);
        //获得绘制文本的宽和高
        mBound = new Rect();
        mPaint.getTextBounds(mText, 0, mText.length(), mBound);
    }
    //API21
//    public MyTextView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
//        super(context, attrs, defStyleAttr, defStyleRes);
//        init();
//    }

    @Override
    protected void onDraw(Canvas canvas) {
        //绘制文字
        canvas.drawText(mText, getWidth() / 2 - mBound.width() / 2, getHeight() / 2 + mBound.height() / 2, mPaint);
    }
}
```

布局文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

        <view.openxu.com.mytextview.MyTextView
            android:layout_width="200dip"
            android:layout_height="100dip"
            android:textSize="20sp"
            android:text="按钮"
            android:background="#ff0000"/>

</LinearLayout>
```

运行结果：

![这里写图片描述](https://github.com/openXu/Blog/raw/master/blog_android/pic01/pic1.png)

上面我只是重写了一个onDraw方法，文本已经绘制出来，说明到此为止这个自定义控件已经算成功了。可是发现了一个问题，如果我要绘制另外的文本呢？比如写i love you，那是不是又得重新定义一个自定义控件？跟上面一样，只是需要修改mText就可以了；行，再写一遍，那如果我现在又想改变文字颜色为蓝色呢？在写一遍？这时候就用到了新的知识点：自定义属性

## 2. 自定义属性

在res/values/下创建一个名为attrs.xml的文件，然后定义如下属性：
format的意思是该属性的取值是什么类型（支持的类型有string,color,demension,integer,enum,reference,float,boolean,fraction,flag）

```Objective-c
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <attr name="mText" format="string" />
    <attr name="mTextColor" format="color" />
    <attr name="mTextSize" format="dimension" />
    <declare-styleable name="MyTextView">
        <attr name="mText"/>
        <attr name="mTextColor"/>
        <attr name="mTextSize"/>
    </declare-styleable>
</resources>
```

然后在布局文件中使用自定义属性，记住一定要引入我们的命名空间 `xmlns:openxu="http://schemas.android.com/apk/res-auto"`

```Objective-c
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:openxu="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

        <view.openxu.com.mytextview.MyTextView
            android:layout_width="200dip"
            android:layout_height="100dip"
            openxu:mTextSize="25sp"
            openxu:mText="i love you"
            openxu:mTextColor ="#0000ff"
            android:background="#ff0000"/>

</LinearLayout>
```

在构造方法中获取自定义属性的值：

```Objective-c
public MyTextView(Context context, AttributeSet attrs, int defStyleAttr) {
    super(context, attrs, defStyleAttr);
    //获取自定义属性的值
    TypedArray a = context.getTheme().obtainStyledAttributes(attrs, R.styleable.MyTextView, defStyleAttr, 0);
    mText = a.getString(R.styleable.MyTextView_mText);
    mTextColor = a.getColor(R.styleable.MyTextView_mTextColor, Color.BLACK);
    mTextSize = a.getDimension(R.styleable.MyTextView_mTextSize, 100);
   a.recycle();  //注意回收
    mPaint = new Paint();
    mPaint.setTextSize(mTextSize);
    mPaint.setColor(mTextColor);
    //获得绘制文本的宽和高
    mBound = new Rect();
    mPaint.getTextBounds(mText, 0, mText.length(), mBound);
}
```

运行结果：

![这里写图片描述](https://github.com/openXu/Blog/raw/master/blog_android/pic01/pic2.png)


通过运行结果，我们已经成功为MyTextView定义了属性，并获取到值，至于自定义属性的详细知识点到后面会专门写一篇博客去介绍。
到此为止，发现自定义控件还是比较简单的嘛。看看结果，跟原生的TextView还有什么差别？接下来做一点小变化：
让绘制的文本长一点`openxu:mText="i love you i love you i love you" `，运行结果：

![这里写图片描述](https://github.com/openXu/Blog/raw/master/blog_android/pic01/pic3.png)

有没有发现不和谐的现象？文本超度超出了控件边界，控件太小，不足以显示辣么长的文本，我们将宽高改为`wrap_content`试试：

![这里写图片描述](https://github.com/openXu/Blog/raw/master/blog_android/pic01/pic4.png)

什么鬼？不是包裹内容吗？怎么填充整个屏幕了？根据顶部官方文档的说明，我们猜想肯定是控件的测量onMeasure方法出了问题，接下来我们学习onMeasure方法。


## 3. onMeasure方法

### ①. MeasureSpec

在学习onMasure方法之前，我们要先了解他的参数中的一个类MeasureSpec，知己知彼才能百战百胜 。
跟踪一下源码，发现它是View中的一个静态内部类，是由尺寸和模式组合而成的一个值，用来描述父控件对子控件尺寸的约束，看看他的部分源码，一共有三种模式，然后提供了合成和分解的方法：

```Objective-c
/**
 * measurespec封装了父控件对他的孩子的布局要求。
 * 一个measurespec由大小和模式。有三种可能的模式：
 */
public static class MeasureSpec {
    private static final int MODE_SHIFT = 30;
    private static final int MODE_MASK  = 0x3 << MODE_SHIFT;

    //父控件不强加任何约束给子控件，它可以是它想要任何大小。
    public static final int UNSPECIFIED = 0 << MODE_SHIFT;  //0

    //父控件决定给孩子一个精确的尺寸
    public static final int EXACTLY     = 1 << MODE_SHIFT;  //1073741824

    //父控件会给子控件尽可能大的尺寸
    public static final int AT_MOST     = 2 << MODE_SHIFT;   //-2147483648
    /**
     * 根据给定的尺寸和模式创建一个约束规范
     */
    public static int makeMeasureSpec(int size, int mode) {
        if (sUseZeroUnspecifiedMeasureSpec && mode == UNSPECIFIED) {
            return 0;
        }
        return makeMeasureSpec(size, mode);
    }
    /**
     * 从约束规范中获取模式
     */
    public static int getMode(int measureSpec) {
        return (measureSpec & MODE_MASK);
    }
    /**
     * 从约束规范中获取尺寸
     */
    public static int getSize(int measureSpec) {
        return (measureSpec & ~MODE_MASK);
    }
}
```

这样说起来还是有点抽象，举一个小栗子大家就知道这三种约束到底是什么意思。我们自定义一个View，为了方便起见，让它继承Button，布局文件中设置不同的宽高条件，然后在onMeasure方法中打印一下他的参数（int widthMeasureSpec, int heightMeasureSpec）到底是个什么鬼

```Objective-c
/**
 * Created by openXu on 16/5/19.
 */
public class MyView extends Button {
    public MyView(Context context) {
        this(context, null);
    }
    public MyView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }
    public MyView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int widthMode = MeasureSpec.getMode(widthMeasureSpec);   //获取宽的模式
        int heightMode = MeasureSpec.getMode(heightMeasureSpec); //获取高的模式
        int widthSize = MeasureSpec.getSize(widthMeasureSpec);   //获取宽的尺寸
        int heightSize = MeasureSpec.getSize(heightMeasureSpec); //获取高的尺寸
        Log.v("openxu", "宽的模式:"+widthMode);
        Log.v("openxu", "高的模式:"+heightMode);
        Log.v("openxu", "宽的尺寸:"+widthSize);
        Log.v("openxu", "高的尺寸:"+heightSize);
    }
}
```

***情形1，让按钮包裹内容:***

```Objective-c
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
        <com.example.openxu.myview.MyView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="20sp"
            android:text="按钮"
            android:background="#ff0000"/>
</LinearLayout>
```
log打印：

```
05-19 06:02:55.112: V/openxu(15599): 宽的模式:-2147483648
05-19 06:02:55.112: V/openxu(15599): 高的模式:-2147483648
05-19 06:02:55.112: V/openxu(15599): 宽的尺寸:1080
05-19 06:02:55.112: V/openxu(15599): 高的尺寸:1860
```

***情形2，让按钮填充父窗体:***

```Objective-c
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
        <com.example.openxu.myview.MyView
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:textSize="20sp"
            android:text="按钮"
            android:background="#ff0000"/>
</LinearLayout>
```

log打印：

```
05-19 06:05:37.302: V/openxu(15960): 宽的模式:1073741824
05-19 06:05:37.302: V/openxu(15960): 高的模式:1073741824
05-19 06:05:37.302: V/openxu(15960): 宽的尺寸:1080
05-19 06:05:37.302: V/openxu(15960): 高的尺寸:1860
```

***情形3，给按钮的宽设置为具体的值:***

```Objective-c
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
        <com.example.openxu.myview.MyView
            android:layout_width="100dip"
            android:layout_height="wrap_content"
            android:textSize="20sp"
            android:text="按钮"
            android:background="#ff0000"/>
</LinearLayout>
```

log打印：

```
05-19 06:07:48.932: V/openxu(16105): 宽的模式:1073741824
05-19 06:07:48.932: V/openxu(16105): 高的模式:-2147483648
05-19 06:07:48.932: V/openxu(16105): 宽的尺寸:300
05-19 06:07:48.932: V/openxu(16105): 高的尺寸:1860
```

根据上面的测试，我们发现，约束中分离出来的尺寸就是父控件剩余的宽高大小（除了设置具体的宽高值外）；而几种约束中的模式不就是对应我们在布局文件中设置给按钮的几种情况吗？如下：

|约束|布局参数|输出值|说明|
|:---|:--:|:--:|:---|
|`UNSPECIFIED`(未指定)| |0|父控件没有对子控件施加任何约束，子控件可以得到任意想要的大小。但是布局文件好像必须设置宽高，目前还没找到与之对应的布局参数，使用较少|
|`EXACTLY`(完全)|`match_parent`/具体宽高值|1073741824|父控件给子控件决定了确切大小，子控件将被限定在给定的边界里而忽略它本身大小。特别说明如果是填充父窗体，说明父控件已经明确知道子控件想要多大的尺寸了（就是剩余的空间都要了)|
|`AT_MOST`(至多)|`wrap_content`|-2147483648|子控件至多达到指定大小的值。包裹内容就是父窗体并不知道子控件到底需要多大尺寸（具体值），需要子控件自己测量之后再让父控件给他一个尽可能大的尺寸以便让内容全部显示但不能超过包裹内容的大小|



### ②. 分析为什么我们自定义的MyTextView设置了wrap_content却填充屏幕

通过上面对MeasureSpec的了解，我们现在就有能看懂View的onMeasure方法默认是怎样为控件测量大小的了
看View中onMeasure的源码：

```Objective-c
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
          
public static int getDefaultSize(int size, int measureSpec) {
    int result = size;
    int specMode = MeasureSpec.getMode(measureSpec);
    int specSize = MeasureSpec.getSize(measureSpec);

    switch (specMode) {
    case MeasureSpec.UNSPECIFIED:
        result = size;
        break;
    case MeasureSpec.AT_MOST:
    case MeasureSpec.EXACTLY:
        result = specSize;
        break;
    }
    return result;
}

```

onMeasure方法调用了setMeasuredDimension(int measuredWidth, int measuredHeight)方法，而传入的参数已经是测量过的默认宽和高的值了；我们看看getDefaultSize 方法是怎么计算测量宽高的。根据父控件给予的约束，发现AT_MOST （相当于wrap_content ）和EXACTLY （相当于match_parent ）两种情况返回的测量宽高都是specSize，而这个specSize正是我们上面说的父控件剩余的宽高，所以默认onMeasure方法中wrap_content 和match_parent 的效果是一样的，都是填充剩余的空间。




### ③. 重写onMeasure方法

我们先忽略掉UNSPECIFIED 的情况（使用极少），只考虑AT_MOST 和EXACTLY ，现在的问题是设置wrap_content 时，控件却使用了match_parent  的效果，看下面怎么重写onMeasure（注释比较详细，不做过多讲解）：



```Objective-c
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    int widthMode = MeasureSpec.getMode(widthMeasureSpec);   //获取宽的模式
    int heightMode = MeasureSpec.getMode(heightMeasureSpec); //获取高的模式
    int widthSize = MeasureSpec.getSize(widthMeasureSpec);   //获取宽的尺寸
    int heightSize = MeasureSpec.getSize(heightMeasureSpec); //获取高的尺寸
    Log.v("openxu", "宽的模式:"+widthMode);
    Log.v("openxu", "高的模式:"+heightMode);
    Log.v("openxu", "宽的尺寸:"+widthSize);
    Log.v("openxu", "高的尺寸:"+heightSize);
    int width;
    int height ;
    if (widthMode == MeasureSpec.EXACTLY) {
        //如果match_parent或者具体的值，直接赋值
        width = widthSize;
    } else {
        //如果是wrap_content，我们要得到控件需要多大的尺寸
        float textWidth = mBound.width();   //文本的宽度
        //控件的宽度就是文本的宽度加上两边的内边距。内边距就是padding值，在构造方法执行完就被赋值
        width = (int) (getPaddingLeft() + textWidth + getPaddingRight());
        Log.v("openxu", "文本的宽度:"+textWidth + "控件的宽度："+width);
    }
    //高度跟宽度处理方式一样
    if (heightMode == MeasureSpec.EXACTLY) {
        height = heightSize;
    } else {
        float textHeight = mBound.height();
        height = (int) (getPaddingTop() + textHeight + getPaddingBottom());
        Log.v("openxu", "文本的高度:"+textHeight + "控件的高度："+height);
    }
    //保存测量宽度和测量高度
    setMeasuredDimension(width, height);
}

```

布局文件：

```Objective-c
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:openxu="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
        <view.openxu.com.mytextview.MyTextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:padding="20dip"
            openxu:mTextSize="25sp"
            openxu:mText="i love you i love you i love you"
            openxu:mTextColor ="#0000ff"
            android:background="#ff0000"/>
</LinearLayout>

```
下面是输出的log：

```
05-19 01:29:12.662 27380-27380/view.openxu.com.mytextview V/openxu: 宽的模式:-2147483648
05-19 01:29:12.662 27380-27380/view.openxu.com.mytextview V/openxu: 高的模式:-2147483648
05-19 01:29:12.662 27380-27380/view.openxu.com.mytextview V/openxu: 宽的尺寸:720
05-19 01:29:12.666 27380-27380/view.openxu.com.mytextview V/openxu: 高的尺寸:1230
05-19 01:29:12.678 27380-27380/view.openxu.com.mytextview V/openxu: 文本的宽度:652.0控件的宽度：732
05-19 01:29:12.690 27380-27380/view.openxu.com.mytextview V/openxu: 文本的高度:49.0控件的高度：129
```

我的模拟器是720x1280的，根据log显示，文本的宽度是652，加上两边的内边距，控件的宽度为732，确实实现了包裹内容的效果，运行程序结果如下：

![这里写图片描述](https://github.com/openXu/Blog/raw/master/blog_android/pic01/pic5.png)

但是发现宽度已经超出了屏幕，还不能像TextView一样换行；下面我们简单的模拟一下换行的功能，做的不够好，但有这个效果，不是重点，不需要重点掌握

## 4. 自动换行

只需要在测量的时候，根据文字的总长度和控件的宽度，就可以知道需要绘制几行，然后将文本分割成小段放入集合中，在onDraw方法中分别绘制；
需要注意的是，onMeasure方法不只调用一次，所以在分段文本是需要判断，不要重复分段，否则会报错。代码如下（仅供参考）：

```Objective-c
import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.Rect;
import android.text.TextUtils;
import android.util.AttributeSet;
import android.util.Log;
import android.view.View;

import java.util.ArrayList;

/**
 * Created by openXu on 16/5/19.
 */
public class MyTextView extends View {

    /**
     * 需要绘制的文字
     */
    private String mText;
    private ArrayList<String> mTextList;
    /**
     * 文本的颜色
     */
    private int mTextColor;
    /**
     * 文本的大小
     */
    private float mTextSize;

    /**
     * 绘制时控制文本绘制的范围
     */
    private Rect mBound;
    private Paint mPaint;

    public MyTextView(Context context) {
        this(context, null);
    }
    public MyTextView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }
    public MyTextView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mTextList = new ArrayList<String>();
        //获取自定义属性的值
        TypedArray a = context.getTheme().obtainStyledAttributes(attrs, R.styleable.MyTextView, defStyleAttr, 0);
        mText = a.getString(R.styleable.MyTextView_mText);
        mTextColor = a.getColor(R.styleable.MyTextView_mTextColor, Color.BLACK);
        mTextSize = a.getDimension(R.styleable.MyTextView_mTextSize, 100);
        a.recycle();  //注意回收
        Log.v("openxu", "文本总长度:"+mText);
        mPaint = new Paint();
        mPaint.setTextSize(mTextSize);
        mPaint.setColor(mTextColor);
        //获得绘制文本的宽和高
        mBound = new Rect();
        mPaint.getTextBounds(mText, 0, mText.length(), mBound);

    }
    //API21
//    public MyTextView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
//        super(context, attrs, defStyleAttr, defStyleRes);
//        init();
//    }

    @Override
    protected void onDraw(Canvas canvas) {
        //绘制文字
        for(int i = 0; i<mTextList.size(); i++){
            mPaint.getTextBounds(mTextList.get(i), 0, mTextList.get(i).length(), mBound);
            Log.v("openxu", "mBound.h:"+mBound.height());
            Log.v("openxu", "在X:" + (getWidth() / 2 - mBound.width() / 2)+"  Y:"+(getPaddingTop() + (mBound.height() *i))+"  绘制："+mTextList.get(i));
            canvas.drawText(mTextList.get(i), (getWidth() / 2 - mBound.width() / 2), (getPaddingTop() + (mBound.height() *i)), mPaint);
        }
    }

    boolean isOneLines = true;
    float lineNum;
    float spLineNum;
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int widthMode = MeasureSpec.getMode(widthMeasureSpec);   //获取宽的模式
        int heightMode = MeasureSpec.getMode(heightMeasureSpec); //获取高的模式
        int widthSize = MeasureSpec.getSize(widthMeasureSpec);   //获取宽的尺寸
        int heightSize = MeasureSpec.getSize(heightMeasureSpec); //获取高的尺寸
        Log.v("openxu", "宽的模式:"+widthMode);
        Log.v("openxu", "高的模式:"+heightMode);
        Log.v("openxu", "宽的尺寸:"+widthSize);
        Log.v("openxu", "高的尺寸:"+heightSize);

        float textWidth = mBound.width();   //文本的宽度
        if(mTextList.size()==0){
            //将文本分段
            int padding = getPaddingLeft() + getPaddingRight();
            int specWidth = widthSize - padding; //能够显示文本的最大宽度
            if(textWidth<specWidth){
                //说明一行足矣显示
                lineNum = 1;
                mTextList.add(mText);
            }else{
                //超过一行
                isOneLines = false;
                spLineNum = textWidth/specWidth;
                if((spLineNum+"").contains(".")){
                    lineNum = Integer.parseInt((spLineNum+"").substring(0,(spLineNum+"").indexOf(".") ))+1;
                }else{
                    lineNum = spLineNum;
                }
                int lineLength = (int)(mText.length()/spLineNum);
                Log.v("openxu", "文本总长度:"+mText);
                Log.v("openxu", "文本总长度:"+mText.length());
                Log.v("openxu", "能绘制文本的宽度:"+lineLength);
                Log.v("openxu", "需要绘制:"+lineNum+"行");
                Log.v("openxu", "lineLength:"+lineLength);
                for(int i = 0; i<lineNum; i++){
                    String lineStr;
                    if(mText.length()<lineLength){
                        lineStr = mText.substring(0, mText.length());
                    }else{
                        lineStr = mText.substring(0, lineLength);
                    }
                    Log.v("openxu", "lineStr:"+lineStr);
                    mTextList.add(lineStr);
                    if(!TextUtils.isEmpty(mText)) {
                        if(mText.length()<lineLength){
                            mText = mText.substring(0, mText.length());
                        }else{
                            mText = mText.substring(lineLength, mText.length());
                        }
                    }else{
                        break;
                    }
                }
            }
        }
        int width;
        int height ;
        if (widthMode == MeasureSpec.EXACTLY) {
            //如果match_parent或者具体的值，直接赋值
            width = widthSize;
        } else {
            //如果是wrap_content，我们要得到控件需要多大的尺寸
            if(isOneLines){
                //控件的宽度就是文本的宽度加上两边的内边距。内边距就是padding值，在构造方法执行完就被赋值
                width = (int) (getPaddingLeft() + textWidth + getPaddingRight());
            }else{
                //如果是多行，说明控件宽度应该填充父窗体
                width = widthSize;
            }
            Log.v("openxu", "文本的宽度:"+textWidth + "控件的宽度："+width);
        }
        //高度跟宽度处理方式一样
        if (heightMode == MeasureSpec.EXACTLY) {
            height = heightSize;
        } else {
            float textHeight = mBound.height();
            if(isOneLines){
                height = (int) (getPaddingTop() + textHeight + getPaddingBottom());
            }else{
                //如果是多行
                height = (int) (getPaddingTop() + textHeight*lineNum + getPaddingBottom());;
            }

            Log.v("openxu", "文本的高度:"+textHeight + "控件的高度："+height);
        }
        //保存测量宽度和测量高度
        setMeasuredDimension(width, height);
    }
}

```

布局文件：

```Objective-c
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:openxu="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
        <view.openxu.com.mytextview.MyTextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:padding="20dip"
            openxu:mTextSize="25sp"
            openxu:mText="门前大桥下，游过一群鸭，快来快来数一数，二四六七八。帽儿破，鞋儿破，身上滴袈裟破，你笑我"
            openxu:mTextColor ="#0000ff"
            android:background="#ff0000"/>
</LinearLayout>

```

运行效果：

![这里写图片描述](https://github.com/openXu/Blog/raw/master/blog_android/pic01/pic6.png)

到此为止，我们已经了解到自定义控件的基本步骤:

1. 继承View，重写构造方法
2. 自定义属性，在构造方法中初始化属性
3. 重写onMeasure方法测量宽高
4. 重写onDraw方法绘制控件


各位看官，看到这里辛苦了，顺便留个言、点个赞呗~ ~谢过了


## 源码下载：

[https://github.com/openXu/MyTextView](https://github.com/openXu/MyTextView)