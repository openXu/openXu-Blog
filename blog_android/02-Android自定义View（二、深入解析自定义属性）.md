> 版权声明：本文为openXu原创文章[【openXu的博客】](http://blog.csdn.net/xmxkf)，未经博主允许不得以任何形式转载

> 由于CSDN-Markdown改版私自替换改动博主文章内容格式，导致博主CSDN全部博客排版混乱，外加CSDN工作人员傲慢的姿态，本人决定择时将博客全部移到GitHub作为备份，后期新增文章也会在此发布，CSDN博客放弃支持，谢谢大家

在上一篇博客[《Android自定义View（一、初体验）》](http://blog.csdn.net/xmxkf/article/details/51454685)中我们体验了自定义控件的基本流程：
> 1. 继承View，覆盖构造方法
> 2. 自定义属性
> 3. 重写onMeasure方法测量宽高
> 4. 重写onDraw方法绘制控件

接下来几篇博客分别深入学习每一个步骤的知识点，第一步就不用多讲了，这篇博客我们看一看自定义属性到底是怎么一个流程，为什么要这样做。

# 1. 为什么要自定义属性

要使用属性，首先这个属性应该存在，所以如果我们要使用自己的属性，必须要先把他定义出来才能使用。但我们平时在写布局文件的时候好像没有自己定义属性，但我们照样可以用很多属性，这是为什么？我想大家应该都知道：系统定义好的属性我们就可以拿来用呗，但是你们知道系统定义了哪些属性吗？哪些属性是我们自定义控件可以直接使用的，哪些不能使用？什么样的属性我们能使用？这些问题我想大家不一定都弄得清除，下面我们去一一解开这些谜团。
系统定义的所有属性我们可以在**\sdk\platforms\android-xx\data\res\values**目录下找到**attrs.xml**这个文件，这就是系统自带的所有属性，打开看看一些比较熟悉的：

```xml
<declare-styleable name="View">
    <attr name="id" format="reference" />
    <attr name="background" format="reference|color" />
    <attr name="padding" format="dimension" />
     ...
    <attr name="focusable" format="boolean" />
     ...
</declare-styleable>

<declare-styleable name="TextView">
    <attr name="text" format="string" localization="suggested" />
    <attr name="hint" format="string" />
    <attr name="textColor" />
    <attr name="textColorHighlight" />
    <attr name="textColorHint" />
     ...
</declare-styleable>

<declare-styleable name="ViewGroup_Layout">
    <attr name="layout_width" format="dimension">
        <enum name="fill_parent" value="-1" />
        <enum name="match_parent" value="-1" />
        <enum name="wrap_content" value="-2" />
    </attr>
    <attr name="layout_height" format="dimension">
        <enum name="fill_parent" value="-1" />
        <enum name="match_parent" value="-1" />
        <enum name="wrap_content" value="-2" />
    </attr>
</declare-styleable>

<declare-styleable name="LinearLayout_Layout">
    <attr name="layout_width" />
    <attr name="layout_height" />
    <attr name="layout_weight" format="float" />
    <attr name="layout_gravity" />
</declare-styleable>

<declare-styleable name="RelativeLayout_Layout">
    <attr name="layout_centerInParent" format="boolean" />
    <attr name="layout_centerHorizontal" format="boolean" />
    <attr name="layout_centerVertical" format="boolean" />
     ...
</declare-styleable>
```

看看上面attrs.xml文件中的属性，发现他们都是有规律的分组的形式组织的。以declare-styleable 为一个组合，后面有一个name属性，属性的值为View 、TextView 等等，有没有想到什么？没错，属性值为View的那一组就是为View定义的属性，属性值为TextView的就是为TextView定义的属性…。

因为所有的控件都是View的子类，所以为View定义的属性所有的控件都能使用，这就是为什么我们的自定义控件没有定义属性就能使用一些系统属性。

但是并不是每个控件都能使用所有属性，比如TextView是View的子类，所以为View定义的所有属性它都能使用，但是子类肯定有自己特有的属性，得单独为它扩展一些属性，而单独扩展的这些属性只有它自己能有，View是不能使用的，比如View中不能使用`android:text=""`。又比如，LinearLayout中能使用layout_weight属性，而RelativeLayout却不能使用，因为layout_weight是为LinearLayout的LayoutParams定义的。

综上所述，自定义控件如果不自定义属性，就只能使用View的属性，但为了给我们的控件扩展一些属性，我们就必须自己去定义。


# 2. 怎样自定义属性

翻阅系统的属性文件，你会发现，有的`<attr name="xxx" />`这中形式，有的是`<attr name=“xxx" format="xxx" />`；这两种的区别就是attr标签后面带不带format属性，如果带format的就是在定义属性，如果不带format的就是在使用已有的属性，name的值就是属性的名字，format是限定当前定义的属性能接受什么值。

打个比方，比如系统已经定义了android:text属性，我们的自定义控件也需要一个文本的属性，可以有两种方式：

**第一种**：我们并不知道系统定义了此名称的属性，我们自己定义一个名为text或者mText的属性（属性名称可以随便起的）

```xml
<resources>
    <declare-styleable name="MyTextView">
        <attr name=“text" format="string" />
    </declare-styleable>
</resources>
```

**第二种**：我们知道系统已经定义过名称为text的属性，我们不用自己定义，只需要在自定义属性中申明，我要使用这个text属性（注意加上android命名空间，这样才知道使用的是系统的text属性）

```xml
<resources>
    <declare-styleable name="MyTextView">
        <attr name=“android:text"/>
    </declare-styleable>
</resources>
```

为什么系统定义了此属性，我们在使用的时候还要声明？因为，系统定义的text属性是给TextView使用的，如果我们不申明，就不能使用text属性。


# 3. 属性值的类型format

**format支持的类型一共有11种：**

***(1). reference：参考某一资源ID***

* 属性定义：
```
<declare-styleable name = "名称">
     <attr name = "background" format = "reference" />
</declare-styleable>
```
* 属性使用：
```xml
<ImageView android:background = "@drawable/图片ID"/>
```

***(2). color：颜色值***

* 属性定义：
```xml
<attr name = "textColor" format = "color" />
```
* 属性使用：
```xml
<TextView android:textColor = "#00FF00" />
```

***(3). boolean：布尔值***

* 属性定义：
```xml
<attr name = "focusable" format = "boolean" />
```
* 属性使用：
```xml
<Button android:focusable = "true"/>
```
***(4).  dimension：尺寸值***

* 属性定义：
```xml
<attr name = "layout_width" format = "dimension" />
```
* 属性使用：
```xml
<Button android:layout_width = "42dip"/>
```
***(5).  float：浮点值***

* 属性定义：
```xml
<attr name = "fromAlpha" format = "float" />
```
* 属性使用：
```xml
<alpha android:fromAlpha = "1.0"/>
```
***(6). integer：整型值***

* 属性定义：
```xml
<attr name = "framesCount" format="integer" />
```
* 属性使用：
```xml
<animated-rotate android:framesCount = "12"/>
```
***(7). string：字符串***

* 属性定义：
```xml
<attr name = "text" format = "string" />
```
* 属性使用：
```xml
<TextView android:text = "我是文本"/>
```
***(8). fraction：百分数***

* 属性定义：
```xml
<attr name = "pivotX" format = "fraction" />
```
* 属性使用：
```xml
<rotate android:pivotX = "200%"/>
```
***(9). enum：枚举值***

* 属性定义：
```xml
<declare-styleable name="名称">
    <attr name="orientation">
        <enum name="horizontal" value="0" />
        <enum name="vertical" value="1" />
    </attr>
</declare-styleable>
```
* 属性使用：
```xml
<LinearLayout  
    android:orientation = "vertical">
</LinearLayout>
```
> **注意：枚举类型的属性在使用的过程中只能同时使用其中一个，不能 `android:orientation = “horizontal｜vertical"`**

**(10). flag：位或运算**

* 属性定义：
```xml
<declare-styleable name="名称">
    <attr name="gravity">
            <flag name="top" value="0x30" />
            <flag name="bottom" value="0x50" />
            <flag name="left" value="0x03" />
            <flag name="right" value="0x05" />
            <flag name="center_vertical" value="0x10" />
            ...
    </attr>
</declare-styleable>
```
* 属性使用：
```xml
<TextView android:gravity="bottom|left"/>
```
> 注意：位运算类型的属性在使用的过程中可以使用多个值

**(11). 混合类型：属性定义时可以指定多种类型值**

* 属性定义：
```xml
<declare-styleable name = "名称">
     <attr name = "background" format = "reference|color" />
</declare-styleable>
```
* 属性使用：
```xml
<ImageView
android:background = "@drawable/图片ID" />
或者：
<ImageView
android:background = "#00FF00" />
```

通过上面的学习我们已经知道怎么定义各种类型的属性，以及怎么使用它们，但是我们写好布局文件之后，要在控件中使用这些属性还需要将它解析出来。


# 4. 类中获取属性值

在这之前，顺带讲一下命名空间，我们在布局文件中使用属性的时候（`android:layout_width="match_parent"`）发现前面都带有一个android：，这个android就是上面引入的命名空间`xmlns:android="http://schemas.android.com/apk/res/android”`，表示到android系统中查找该属性来源。只有引入了命名空间，XML文件才知道下面使用的属性应该去哪里找（哪里定义的，不能凭空出现，要有根据）。

如果我们自定义属性，这个属性应该去我们的应用程序包中找，所以要引入我们应用包的命名空间`xmlns:openxu="http://schemas.android.com/apk/res-auto”`，res-auto表示自动查找，还有一种写法`xmlns:openxu="http://schemas.android.com/apk/com.example.openxu.myview"`，`com.example.openxu.myview`为我们的应用程序包名。

按照上面学习的知识，我们先定义一些属性，并写好布局文件。
先在res\values目录下创建attrs.xml，定义自己的属性：
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="MyTextView">
        <!--声明MyTextView需要使用系统定义过的text属性,注意前面需要加上android命名-->
        <attr name="android:text" />
        <attr name="mTextColor" format="color" />
        <attr name="mTextSize" format="dimension" />
    </declare-styleable>
</resources>
```

在布局文件中，使用属性（注意引入我们应用程序的命名空间，这样在能找到我们包中的attrs）：
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:openxu="http://schemas.android.com/apk/res-auto"
    android:orientation="horizontal"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
        <com.example.openxu.myview.MyTextView
            android:layout_width="200dip"
            android:layout_height="100dip"
            openxu:mTextSize="25sp"
            android:text="我是文字"
            openxu:mTextColor ="#0000ff"
            android:background="#ff0000"/>
</LinearLayout>
```

在构造方法中获取属性值：
```Objective-c
public MyTextView(Context context, AttributeSet attrs, int defStyleAttr) {
    super(context, attrs, defStyleAttr);
    TypedArray ta = context.obtainStyledAttributes(attrs, R.styleable.MyTextView);
    String text = ta.getString(R.styleable.MyTextView_android_text);
    int mTextColor = ta.getColor(R.styleable.MyTextView_mTextColor, Color.BLACK);
    int mTextSize = ta.getDimensionPixelSize(R.styleable.MyTextView_mTextSize, 100);
    ta.recycle();  //注意回收
    Log.v("openxu", “text属性值:"+mText);
    Log.v("openxu", "mTextColor属性值:"+mTextColor);
    Log.v("openxu", "mTextSize属性值:"+mTextSize);
}
```
log输出：
```
05-21 00:14:07.192: V/openxu(25652): mText属性值:我是文字
05-21 00:14:07.192: V/openxu(25652): mTextColor属性值:-16776961
05-21 00:14:07.192: V/openxu(25652): mTextSize属性值:75
```

到此为止，是不是发现自定义属性是如此简单？

属性的定义我们应该学的差不多了，但有没有发现构造方法中获取属性值的时候有两个比较陌生的类`AttributeSet` 和`TypedArray`，这两个类是怎么把属性值从布局文件中解析出来的？


# 5. Attributeset和TypedArray以及declare-styleable
Attributeset看名字就知道是一个属性的集合，实际上，它内部就是一个XML解析器，帮我们将布局文件中该控件的所有属性解析出来，并以key-value的兼职对形式维护起来。其实我们完全可以只用他通过下面的代码来获取我们的属性就行。

```Objective-c
public MyTextView(Context context, AttributeSet attrs, int defStyleAttr) {
    super(context, attrs, defStyleAttr);
    int count = attrs.getAttributeCount();
    for (int i = 0; i < count; i++) {
        String attrName = attrs.getAttributeName(i);
        String attrVal = attrs.getAttributeValue(i);
        Log.e("openxu", "attrName = " + attrName + " , attrVal = " + attrVal);
    }
}
```

log输出：

```
05-21 02:18:09.052: E/openxu(14704): attrName = background , attrVal = @2131427347
05-21 02:18:09.052: E/openxu(14704): attrName = layout_width , attrVal = 200.0dip
05-21 02:18:09.052: E/openxu(14704): attrName = layout_height , attrVal = 100.0dip
05-21 02:18:09.052: E/openxu(14704): attrName = text , attrVal = 我是文字
05-21 02:18:09.052: E/openxu(14704): attrName = mTextSize , attrVal = 25sp
05-21 02:18:09.052: E/openxu(14704): attrName = mTextColor , attrVal = #0000ff
```

发现通过Attributeset获取属性的值时，它将我们布局文件中的值原原本本的获取出来的，比如宽度200.0dip，其实这并不是我们想要的，如果我们接下来要使用宽度值，我们还需要将dip去掉，然后转换成整形，这多麻烦。其实这都不算什么，更恶心的是，backgroud我应用了一个color资源ID，它直接给我拿到了这个ID值，前面还加了个@，接下来我要自己获取资源，并通过这个ID值获取到真正的颜色。
我们再换***TypedArray***试试。
在这里，穿插一个知识点，定义属性的时候有一个***declare-styleable***，他是用来干嘛的，如果不要它可不可以？答案是可以的，我们自定义属性完全可以写成下面的形式：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
        <attr name="mTextColor" format="color" />
        <attr name="mTextSize" format="dimension" />
</resources>
```

之前的形式是这样的：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="MyTextView">
        <attr name="android:text" />
        <attr name="android:layout_width" />
        <attr name="android:layout_height" />
        <attr name="android:background" />
        <attr name="mTextColor" format="color" />
        <attr name="mTextSize" format="dimension" />
    </declare-styleable>
</resources>

或者：

<?xml version="1.0" encoding="utf-8"?>
<resources>
          <!--定义属性-->
       <attr name="mTextColor" format="color" />
        <attr name="mTextSize" format="dimension" />
    <declare-styleable name="MyTextView">
        <!--生成索引-->
        <attr name="android:text" />
        <attr name="android:layout_width" />
        <attr name="android:layout_height" />
        <attr name="android:background" />
        <attr name=“mTextColor" />
        <attr name="mTextSize"  />
    </declare-styleable>
</resources>
```

我们都知道所有的资源文件在R中都会对应一个整型常亮，我们可以通过这个ID值找到资源文件。
属性在R中对应的类是`public static final class attr`，如果我们写了***declare-styleable***，在R文件中就会生成***styleable***类，这个类其实就是将每个控件的属性分组，然后记录属性的索引值，而TypedArray正好需要通过此索引值获取属性。

```Objective-c
public static final class styleable

          public static final int[] MyTextView = {
            0x0101014f, 0x7f010038, 0x7f010039
        };
        public static final int MyTextView_android_text = 0;
        public static final int MyTextView_mTextColor = 1;
        public static final int MyTextView_mTextSize = 2;
｝
```

使用TypedArray获取属性值：

```Objective-c
public MyTextView(Context context, AttributeSet attrs, int defStyleAttr) {
    super(context, attrs, defStyleAttr);
    TypedArray ta = context.obtainStyledAttributes(attrs, R.styleable.MyTextView);
    String mText = ta.getString(R.styleable.MyTextView_android_text);
    int mTextColor = ta.getColor(R.styleable.MyTextView_mTextColor, Color.BLACK);
    int mTextSize = ta.getDimensionPixelSize(R.styleable.MyTextView_mTextSize, 100);
    float width = ta.getDimension(R.styleable.MyTextView_android_layout_width, 0.0f);
    float hight = ta.getDimension(R.styleable.MyTextView_android_layout_height,0.0f);
    int backgroud = ta.getColor(R.styleable.MyTextView_android_background, Color.BLACK);
    ta.recycle();  //注意回收
    Log.v("openxu", "width:"+width);
    Log.v("openxu", "hight:"+hight);
    Log.v("openxu", "backgroud:"+backgroud);
    Log.v("openxu", "mText:"+mText);
    Log.v("openxu", "mTextColor:"+mTextColor);
    Log.v("openxu", "mTextSize:"+mTextSize);ext, 0, mText.length(), mBound);
}
```

log输出：

```
05-21 02:56:49.162: V/openxu(22630): width:600.0
05-21 02:56:49.162: V/openxu(22630): hight:300.0
05-21 02:56:49.162: V/openxu(22630): backgroud:-12627531
05-21 02:56:49.162: V/openxu(22630): mText:我是文字
05-21 02:56:49.162: V/openxu(22630): mTextColor:-16777216
05-21 02:56:49.162: V/openxu(22630): mTextSize:100
```

看看多么舒服的结果，我们得到了想要的宽高（float型），背景颜色（color的十进制）等，TypedArray提供了一系列获取不同类型属性的方法，这样就可以直接得到我们想要的数据类型，而不用像Attributeset获取属性后还要一个个处理才能得到具体的数据，实际上TypedArray是为我们获取属性值提供了方便，注意一点，TypedArray使用完毕后记得调用 `ta.recycle(); `回收 。


好了，今天的任务算是完成了，我们已经完全掌握自定义属性，总结一下这篇博客的重点内容：
> 1. 为什么要自定义属性
> 2. 自定义属性的方法
> 3. 声明使用系统自带的属性
> 4. 属性值的类型format
> 5. 构造方法中获取属性值
> 6. Attributeset、TypedArray以及declare-styleable的用法

不知不绝天快亮了，各位学习的童鞋，鼓励鼓励，评论评论顺便点个赞呗～谢过了喔