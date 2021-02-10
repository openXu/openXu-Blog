> 版权声明：本文为openXu原创文章[【openXu的博客】](http://blog.csdn.net/xmxkf)，未经博主允许不得以任何形式转载

目录：

@[toc]

&emsp;&emsp;通过前面几篇博客，我们能够自定义出一些比较简单的自定义控件，但是这在实际应用中是远远不够的，为了实现一些比较牛X的效果，比如侧滑菜单、滑动卡片等等，我们还需要了解自定义`ViewGroup`。官方文档中对ViewGroup这样描述的：

> ViewGroup是一种可以包含其他视图的特殊视图，他是各种布局和所有容器的基类，这些类也定义了ViewGroup.LayoutParams类作为类的布局参数。

&emsp;&emsp;之前，我们只是学习过自定义`View`，其实自定义`ViewGroup`和自定义`View`的步骤差不了多少，他们的的区别主要来自各自的作用不同，ViewGroup是容器，用来包含其他控件，而View是真正意义上看得见摸得着的，它需要将自己画出来。`ViewGroup`需要重写`onMeasure()`方法测量子控件的宽高和自己的宽高，然后实现`onLayout()`方法摆放子控件。而 `View`则是需要重写`onMeasure()`根据测量模式和父控件给出的建议的宽高值计算自己的宽高，然后再父控件为其指定的区域绘制自己的图形。
　　
根据以往经验我们初步将自定义ViewGroup的步骤定为下面几步：

1. 继承`ViewGroup`，覆盖构造方法
2. 重写`onMeasure()`方法测量子控件和自身宽高
3. 实现`onLayout()`方法摆放子控件

# 1. 简单实现水平排列效果

我们先自定义一个`ViewGroup`作为布局容器，实现一个从左往右水平排列（排满换行）的效果：

```java
/**
 * 自定义布局管理器的示例。
 */
public class CustomLayout extends ViewGroup {
      private static final String TAG = "CustomLayout";

    public CustomLayout(Context context) {
        super(context);
    }

    public CustomLayout(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CustomLayout(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }

    /**
     * 要求所有的孩子测量自己的大小，然后根据这些孩子的大小完成自己的尺寸测量
     */
    @SuppressLint("NewApi") @Override
    protected void onMeasure( int widthMeasureSpec, int heightMeasureSpec) {
        // 计算出所有的childView的宽和高 
        measureChildren(widthMeasureSpec, heightMeasureSpec); 
        //测量并保存layout的宽高(使用getDefaultSize时，wrap_content和match_perent都是填充屏幕)
        //稍后会重新写这个方法，能达到wrap_content的效果
        setMeasuredDimension( getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }

    /**
     * 为所有的子控件摆放位置.
     */
    @Override
    protected void onLayout( boolean changed, int left, int top, int right, int bottom) {
        final int count = getChildCount();
        int childMeasureWidth = 0;
        int childMeasureHeight = 0;
        int layoutWidth = 0;    // 容器已经占据的宽度
        int layoutHeight = 0;   // 容器已经占据的宽度
        int maxChildHeight = 0; //一行中子控件最高的高度，用于决定下一行高度应该在目前基础上累加多少
        for(int i = 0; i<count; i++){
            View child = getChildAt(i);
             //注意此处不能使用getWidth和getHeight，这两个方法必须在onLayout执行完，才能正确获取宽高
            childMeasureWidth = child.getMeasuredWidth(); 
            childMeasureHeight = child.getMeasuredHeight(); 
            if(layoutWidth<getWidth()){
                   //如果一行没有排满，继续往右排列
                  left = layoutWidth;
                  right = left+childMeasureWidth;
                  top = layoutHeight;
                  bottom = top+childMeasureHeight;
            } else{
                   //排满后换行
                  layoutWidth = 0;
                  layoutHeight += maxChildHeight;
                  maxChildHeight = 0;
                  
                  left = layoutWidth;
                  right = left+childMeasureWidth;
                  top = layoutHeight;
                  bottom = top+childMeasureHeight;
            }

            layoutWidth += childMeasureWidth;  //宽度累加
             if(childMeasureHeight>maxChildHeight){
                  maxChildHeight = childMeasureHeight;
            }
                  
             //确定子控件的位置，四个参数分别代表（左上右下）点的坐标值
            child.layout(left, top, right, bottom);
        }
    }
}

```

**布局文件：**

```xml
<?xml version="1.0" encoding= "utf-8"?>
<com.openxu.costomlayout.CustomLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width= "wrap_content"
    android:layout_height= "wrap_content" >
    <Button
        android:layout_width= "wrap_content"
        android:layout_height= "wrap_content"
        android:background= "#FF8247"
        android:textColor= "#ffffff"
        android:textSize="20dip"
        android:padding= "20dip"
        android:text="按钮1" />
    <Button
        android:layout_width= "wrap_content"
        android:layout_height= "wrap_content"
        android:background= "#8B0A50"
        android:textColor= "#ffffff"
        android:textSize="20dip"
        android:padding= "10dip"
        android:text="按钮2222222222222" />
    <Button
        android:layout_width= "wrap_content"
        android:layout_height= "wrap_content"
        android:background= "#7CFC00"
        android:textColor= "#ffffff"
        android:textSize="20dip"
        android:padding= "15dip"
        android:text="按钮333333" />
    <Button
        android:layout_width= "wrap_content"
        android:layout_height= "wrap_content"
        android:background= "#1E90FF"
        android:textColor= "#ffffff"
        android:textSize="10dip"
        android:padding= "10dip"
        android:text="按钮4" />
    <Button
        android:layout_width= "wrap_content"
        android:layout_height= "wrap_content"
        android:background= "#191970"
        android:textColor= "#ffffff"
        android:textSize="20dip"
        android:padding= "15dip"
        android:text="按钮5" />
    <Button
        android:layout_width= "wrap_content"
        android:layout_height= "wrap_content"
        android:background= "#7A67EE"
        android:textColor= "#ffffff"
        android:textSize="20dip"
        android:padding= "20dip"
        android:text="按钮6" />

</com.openxu.costomlayout.CustomLayout>
```

</br>

**运行效果：**

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwNTI1MTcwMzQ1NDky)

运行成功，是不是略有成就感？这个布局就是简单版的`LinearLayout`设置`android:orientation ="horizontal"`的效果，比他还牛X一点，还能自动换行（哈哈）。接下来我们要实现一个功能，只需要在布局文件中指定布局属性，就能控制子控件在什么位置（类似相对布局`RelativeLayout`）。

</br>

# 2. 自定义LayoutParams

回想一下我们平时使用`RelativeLayout`的时候，在布局文件中使用`android:layout_alignParentRight="true"`、`android:layout_centerInParent="true"`等各种属性，就能控制子控件显示在父控件的上下左右、居中等效果。 在上一篇[讲onMeasure的博客](http://blog.csdn.net/xmxkf/article/details/51490283)中，我们有了解过`ViewGroup.LayoutParams`类，ViewGroup中有两个内部类`ViewGroup.LayoutParams`和`ViewGroup.MarginLayoutParams`，`MarginLayoutParams`继承自`LayoutParams`，这两个内部类就是`ViewGroup`的布局参数类，比如我们在`LinearLayout`等布局中使用的`layout_width\layout_hight`等以“layout_ ”开头的属性都是布局属性。

在View中有一个`mLayoutParams`的变量用来保存这个View的所有布局属性。`ViewGroup.LayoutParams`有两个属性`layout_width`和`layout_height`，因为所有的容器都需要设置子控件的宽高，所以这个`LayoutParams`是所有布局参数的基类，如果需要扩展其他属性，都应该继承自它。比如`RelativeLayout`中就提供了它自己的布局参数类`RelativeLayout.LayoutParams`，并扩展了很多布局参数，我们平时在`RelativeLayout`中使用的布局属性都来自它 ：

```xml
<declare-styleable name= "RelativeLayout_Layout">
        <attr name ="layout_toLeftOf" format= "reference" />
        <attr name ="layout_toRightOf" format= "reference" />
        <attr name ="layout_above" format="reference" />
        <attr name ="layout_below" format="reference" />
        <attr name ="layout_alignBaseline" format= "reference" />
        <attr name ="layout_alignLeft" format= "reference" />
        <attr name ="layout_alignTop" format= "reference" />
        <attr name ="layout_alignRight" format= "reference" />
        <attr name ="layout_alignBottom" format= "reference" />
        <attr name ="layout_alignParentLeft" format= "boolean" />
        <attr name ="layout_alignParentTop" format= "boolean" />
        <attr name ="layout_alignParentRight" format= "boolean" />
        <attr name ="layout_alignParentBottom" format= "boolean" />
        <attr name ="layout_centerInParent" format= "boolean" />
        <attr name ="layout_centerVertical" format= "boolean" />
        <attr name ="layout_alignWithParentIfMissing" format= "boolean" />
        <attr name ="layout_toStartOf" format= "reference" />
        <attr name ="layout_toEndOf" format="reference" />
        <attr name ="layout_alignStart" format= "reference" />
        <attr name ="layout_alignEnd" format= "reference" />
        <attr name ="layout_alignParentStart" format= "boolean" />
        <attr name ="layout_alignParentEnd" format= "boolean" />
    </declare-styleable >

```

看了上面的介绍，我们大概知道怎么为我们的布局容器定义自己的布局属性了吧，就不绕弯子了，按照下面的步骤做：

</br>

## ①. 大致明确布局容器的需求，初步定义布局属性

在定义属性之前要弄清楚，我们自定义的布局容器需要满足那些需求，需要哪些属性，比如，我们现在要实现像相对布局一样，为子控件设置一个位置属性layout_position=""，来控制子控件在布局中显示的位置。暂定位置有五种：左上、左下、右上、右下、居中。有了需求，我们就在attr.xml定义自己的布局属性（和之前讲的自定义属性一样的操作，不太了解的可以翻阅[《Android自定义View（二、深入解析自定义属性）》](https://blog.csdn.net/xmxkf/article/details/51468648)。

```xml
<?xml version="1.0" encoding= "utf-8"?>
<resources> 
    <declare-styleable name ="CustomLayout">
    <attr name ="layout_position">
        <enum name ="center" value="0" />
        <enum name ="left" value="1" />
        <enum name ="right" value="2" />
        <enum name ="bottom" value="3" />
        <enum name ="rightAndBottom" value="4" />
    </attr >
    </declare-styleable>
</resources>
```

left就代表是左上（按常理默认就是左上方开始，就不用写leftTop了，简洁一点），bottom左下，right 右上，rightAndBottom右下，center居中。属性类型是枚举，同时只能设置一个值。

</br>
 
## ②. 继承LayoutParams，定义布局参数类

我们可以选择继承`ViewGroup.LayoutParams`，这样的话我们的布局只是简单的支持`layout_width`和`layout_height`；也可以继承`MarginLayoutParams`，就能使用layout_marginxxx属性了。因为后面我们还要用到margin属性，所以这里方便起见就直接继承`MarginLayoutParams`了。

覆盖构造方法，然后在有AttributeSet参数的构造方法中初始化参数值，这个构造方法才是布局文件被映射为对象的时候被调用的。

```java
public static class CustomLayoutParams extends MarginLayoutParams {
       public static final int POSITION_MIDDLE = 0; // 中间
       public static final int POSITION_LEFT = 1; // 左上方
       public static final int POSITION_RIGHT = 2; // 右上方
       public static final int POSITION_BOTTOM = 3; // 左下角
       public static final int POSITION_RIGHTANDBOTTOM = 4; // 右下角

       public int position = POSITION_LEFT;  // 默认我们的位置就是左上角

       public CustomLayoutParams(Context c, AttributeSet attrs) {
             super(c, attrs);
            TypedArray a = c.obtainStyledAttributes(attrs,R.styleable.CustomLayout );
             //获取设置在子控件上的位置属性
             position = a.getInt(R.styleable.CustomLayout_layout_position ,position );

            a.recycle();
      }

       public CustomLayoutParams( int width, int height) {
             super(width, height);
      }

       public CustomLayoutParams(ViewGroup.LayoutParams source) {
             super(source);
      }

}
```
</br>

## ③. 重写generateLayoutParams()

在`ViewGroup`中有下面几个关于`LayoutParams`的方法，`generateLayoutParams (AttributeSet attrs) `是在布局文件被填充为对象的时候调用的，这个方法是下面几个方法中最重要的，如果不重写它，我么布局文件中设置的布局参数都不能拿到。后面我也会专门写一篇博客来介绍布局文件被添加到activity窗口的过程，里面会讲到这个方法被调用的来龙去脉。其他几个方法我们最好也能重写一下，将里面的`LayoutParams`换成我们自定义的` CustomLayoutParams `类，避免以后会遇到布局参数类型转换异常。

```java
@Override
public LayoutParams generateLayoutParams(AttributeSet attrs) {
    return new CustomLayoutParams(getContext(), attrs);
}
@Override
protected ViewGroup.LayoutParams generateLayoutParams(ViewGroup.LayoutParams p) {
    return new CustomLayoutParams (p);
}
@Override
protected LayoutParams generateDefaultLayoutParams() {
    return new CustomLayoutParams (LayoutParams.MATCH_PARENT , LayoutParams.MATCH_PARENT);
}
@Override
protected boolean checkLayoutParams(ViewGroup.LayoutParams p) {
    return p instanceof CustomLayoutParams ;
}
```
</br>

## ④. 在布局文件中使用布局属性

注意引入命名空间`xmlns:openxu= "http://schemas.android.com/apk/res/包名"`

```xml
<?xml version="1.0" encoding= "utf-8"?>
<com.openxu.costomlayout.CustomLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:openxu= "http://schemas.android.com/apk/res/com.openxu.costomlayout"
    android:background="#33000000"
    android:layout_width= "match_parent "
    android:layout_height= "match_parent" >

    <Button
        android:layout_width= "wrap_content"
        android:layout_height= "wrap_content"
        openxu:layout_position= "left"
        android:background= "#FF8247"
        android:textColor= "#ffffff"
         android:textSize="20dip"
        android:padding= "20dip"
        android:text= "按钮1" />

    <Button
        android:layout_width= "wrap_content"
        android:layout_height= "wrap_content"
        openxu:layout_position= "right"
        android:background= "#8B0A50"
        android:textColor= "#ffffff"
        android:textSize= "18dip"
        android:padding= "10dip"
        android:text= "按钮2222222222222" />

    <Button
        android:layout_width= "wrap_content"
        android:layout_height= "wrap_content"
        openxu:layout_position= "bottom"
        android:background= "#7CFC00"
        android:textColor= "#ffffff"
        android:textSize= "20dip"
        android:padding= "15dip"
        android:text= "按钮333333" />

    <Button
        android:layout_width= "wrap_content"
        android:layout_height= "wrap_content"
        openxu:layout_position= "rightAndBottom"
        android:background= "#1E90FF"
        android:textColor= "#ffffff"
        android:textSize= "15dip"
        android:padding= "10dip"
        android:text= "按钮4" />

    <Button
        android:layout_width= "wrap_content"
        android:layout_height= "wrap_content"
        openxu:layout_position= "center"
        android:background= "#191970"
        android:textColor= "#ffffff"
        android:textSize= "20dip"
        android:padding= "15dip"
        android:text= "按钮5" />

</com.openxu.costomlayout.CustomLayout>
```
</br>

## ⑤. 在onMeasure()和onLayout()中使用布局参数

经过上面几步之后，我们运行程序，就能获取子控件的布局参数了，在`onMeasure()`方法和`onLayout()`方法中，我们按照自定义布局容器的特殊需求，对宽度和位置坐特殊处理。这里我们需要注意一下，如果布局容器被设置为包裹类容，我们只需要保证能将最大的子控件包裹住就ok，代码注释比较详细，就不多说了。

```java
 @Override
protected void onMeasure( int widthMeasureSpec, int heightMeasureSpec) { 
  //获得此ViewGroup上级容器为其推荐的宽和高，以及计算模式  
 int widthMode = MeasureSpec. getMode(widthMeasureSpec); 
 int heightMode = MeasureSpec. getMode(heightMeasureSpec); 
 int sizeWidth = MeasureSpec. getSize(widthMeasureSpec); 
 int sizeHeight = MeasureSpec. getSize(heightMeasureSpec); 
 int layoutWidth = 0;
 int layoutHeight = 0;
      // 计算出所有的childView的宽和高
     measureChildren(widthMeasureSpec, heightMeasureSpec);
     
      int cWidth = 0;
      int cHeight = 0;
      int count = getChildCount(); 
     
      if(widthMode == MeasureSpec. EXACTLY){
            //如果布局容器的宽度模式是确定的（具体的size或者match_parent），直接使用父窗体建议的宽度
           layoutWidth = sizeWidth;
     } else{
            //如果是未指定或者wrap_content，我们都按照包裹内容做，宽度方向上只需要拿到所有子控件中宽度做大的作为布局宽度
            for ( int i = 0; i < count; i++)  { 
                  View child = getChildAt(i); 
              cWidth = child.getMeasuredWidth(); 
              //获取子控件最大宽度
              layoutWidth = cWidth > layoutWidth ? cWidth : layoutWidth;
           }
     }
      //高度很宽度处理思想一样
      if(heightMode == MeasureSpec. EXACTLY){
           layoutHeight = sizeHeight;
     } else{
            for ( int i = 0; i < count; i++)  { 
                  View child = getChildAt(i); 
                  cHeight = child.getMeasuredHeight();
                  layoutHeight = cHeight > layoutHeight ? cHeight : layoutHeight;
           }
     }
     
      // 测量并保存layout的宽高
     setMeasuredDimension(layoutWidth, layoutHeight);
}

@Override
protected void onLayout( boolean changed, int left, int top, int right,
            int bottom) {
      final int count = getChildCount();
      int childMeasureWidth = 0;
      int childMeasureHeight = 0;
     CustomLayoutParams params = null;
      for ( int i = 0; i < count; i++) {
           View child = getChildAt(i);
            // 注意此处不能使用getWidth和getHeight，这两个方法必须在onLayout执行完，才能正确获取宽高
           childMeasureWidth = child.getMeasuredWidth();
           childMeasureHeight = child.getMeasuredHeight();

           params = (CustomLayoutParams) child.getLayoutParams(); 
     switch (params. position) {
            case CustomLayoutParams. POSITION_MIDDLE:    // 中间
                 left = (getWidth()-childMeasureWidth)/2;
                 top = (getHeight()-childMeasureHeight)/2;
                  break;
            case CustomLayoutParams. POSITION_LEFT:      // 左上方
                 left = 0;
                 top = 0;
                  break;
            case CustomLayoutParams. POSITION_RIGHT:     // 右上方
                 left = getWidth()-childMeasureWidth;
                 top = 0;
                  break;
            case CustomLayoutParams. POSITION_BOTTOM:    // 左下角
                 left = 0;
                 top = getHeight()-childMeasureHeight;
                  break;
            case CustomLayoutParams. POSITION_RIGHTANDBOTTOM:// 右下角
                 left = getWidth()-childMeasureWidth;
                 top = getHeight()-childMeasureHeight;
                  break;
            default:
                  break;
           }
    
            // 确定子控件的位置，四个参数分别代表（左上右下）点的坐标值
           child.layout(left, top, left+childMeasureWidth, top+childMeasureHeight);
     }
}
```

</br>

**运行效果：**

下面几个效果分别对应布局容器宽高设置不同的属性的情况（设置match_parent 、设置200dip、设置）：

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwNTI1MTcyMTI0ODYz)　![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwNTI1MTcyMTM4MjE3)　![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwNTI1MTcyMTQ2NDM2)

从运行结果看，我们自定义的布局容器在各种宽高设置下都能很好的测量大小和摆放子控件。现在我们让他支持`margin`属性

</br>

# 3. 支持layout_margin属性

如果我们自定义的布局参数类继承自`MarginLayoutParams`，就自动支持了`layout_margin`属性了，我们需要做的就是直接在布局文件中使用layout_margin属性，然后再`onMeasure()`和`onLayout()`中使用`margin`属性值测量和摆放子控件。需要注意的是我们测量子控件的时候应该调用`measureChildWithMargin()`方法。

**布局文件：**

```xml
<?xml version="1.0" encoding= "utf-8"?>
<com.openxu.costomlayout.CustomLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:openxu= "http://schemas.android.com/apk/res/com.openxu.costomlayout"
    android:background="#33000000"
    android:layout_width= "match_parent"
    android:layout_height= "match_parent" >
    <Button
        android:layout_width= "wrap_content"
        android:layout_height= "wrap_content"
        openxu:layout_position= "left"
        android:layout_marginLeft = "20dip"
        android:background= "#FF8247"
        android:textColor= "#ffffff"
        android:textSize="20dip"
        android:padding= "20dip"
        android:text="按钮1" />
    <Button
        android:layout_width= "wrap_content"
        android:layout_height= "wrap_content"
        android:layout_marginTop = "30dip"
        openxu:layout_position= "right"
        android:background= "#8B0A50"
        android:textColor= "#ffffff"
        android:textSize="18dip"
        android:padding= "10dip"
        android:text="按钮2222222222222" />
    <Button
        android:layout_width= "wrap_content"
        android:layout_height= "wrap_content"
        android:layout_marginLeft = "30dip"
        android:layout_marginBottom = "10dip"
        openxu:layout_position= "bottom"
        android:background= "#7CFC00"
        android:textColor= "#ffffff"
        android:textSize="20dip"
        android:padding= "15dip"
        android:text="按钮333333" />
    <Button
        android:layout_width= "wrap_content"
        android:layout_height= "wrap_content"
        openxu:layout_position= "rightAndBottom"
        android:layout_marginBottom = "30dip"
        android:background= "#1E90FF"
        android:textColor= "#ffffff"
        android:textSize="15dip"
        android:padding= "10dip"
        android:text="按钮4" />
    <Button
        android:layout_width= "wrap_content"
        android:layout_height= "wrap_content"
        openxu:layout_position= "center"
        android:layout_marginBottom = "30dip"
        android:layout_marginRight = "30dip"
        android:background= "#191970"
        android:textColor= "#ffffff"
        android:textSize="20dip"
        android:padding= "15dip"
        android:text="按钮5" />

</com.openxu.costomlayout.CustomLayout>
```

**onMeasure()和onLayout()：**

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) { 
   // 获得此ViewGroup上级容器为其推荐的宽和高，以及计算模式   
  int widthMode = MeasureSpec. getMode(widthMeasureSpec); 
  int heightMode = MeasureSpec. getMode(heightMeasureSpec); 
  int sizeWidth = MeasureSpec. getSize(widthMeasureSpec); 
  int sizeHeight = MeasureSpec. getSize(heightMeasureSpec); 
  int layoutWidth = 0;
  int layoutHeight = 0;
       int cWidth = 0;
       int cHeight = 0;
       int count = getChildCount(); 

       // 计算出所有的childView的宽和高
       for( int i = 0; i < count; i++){
            View child = getChildAt(i); 
            measureChildWithMargins(child, widthMeasureSpec, 0, heightMeasureSpec, 0);
      }
      CustomLayoutParams params = null;
       if(widthMode == MeasureSpec. EXACTLY){
             //如果布局容器的宽度模式时确定的（具体的size或者match_parent）
            layoutWidth = sizeWidth;
      } else{
             //如果是未指定或者wrap_content，我们都按照包裹内容做，宽度方向上只需要拿到所有子控件中宽度做大的作为布局宽度
             for ( int i = 0; i < count; i++)  { 
                   View child = getChildAt(i); 
               cWidth = child.getMeasuredWidth(); 
               params = (CustomLayoutParams) child.getLayoutParams(); 
               //获取子控件宽度和左右边距之和，作为这个控件需要占据的宽度
               int marginWidth = cWidth+params.leftMargin+params.rightMargin ;
               layoutWidth = marginWidth > layoutWidth ? marginWidth : layoutWidth;
            }
      }
       //高度很宽度处理思想一样
       if(heightMode == MeasureSpec. EXACTLY){
            layoutHeight = sizeHeight;
      } else{
             for ( int i = 0; i < count; i++)  { 
                   View child = getChildAt(i); 
                   cHeight = child.getMeasuredHeight();
                   params = (CustomLayoutParams) child.getLayoutParams(); 
                   int marginHeight = cHeight+params.topMargin+params.bottomMargin ;
                   layoutHeight = marginHeight > layoutHeight ? marginHeight : layoutHeight;
            }
      }
      
       // 测量并保存layout的宽高
      setMeasuredDimension(layoutWidth, layoutHeight);
}

@Override
protected void onLayout( boolean changed, int left, int top, int right,
             int bottom) {
       final int count = getChildCount();
       int childMeasureWidth = 0;
       int childMeasureHeight = 0;
      CustomLayoutParams params = null;
       for ( int i = 0; i < count; i++) {
            View child = getChildAt(i);
             // 注意此处不能使用getWidth和getHeight，这两个方法必须在onLayout执行完，才能正确获取宽高
            childMeasureWidth = child.getMeasuredWidth();
            childMeasureHeight = child.getMeasuredHeight();
            params = (CustomLayoutParams) child.getLayoutParams(); 
      switch (params. position) {
             case CustomLayoutParams. POSITION_MIDDLE:    // 中间
                  left = (getWidth()-childMeasureWidth)/2 - params.rightMargin + params.leftMargin ;
                  top = (getHeight()-childMeasureHeight)/2 + params.topMargin - params.bottomMargin ;
                   break;
             case CustomLayoutParams. POSITION_LEFT:      // 左上方
                  left = 0 + params. leftMargin;
                  top = 0 + params. topMargin;
                   break;
             case CustomLayoutParams. POSITION_RIGHT:     // 右上方
                  left = getWidth()-childMeasureWidth - params.rightMargin;
                  top = 0 + params. topMargin;
                   break;
             case CustomLayoutParams. POSITION_BOTTOM:    // 左下角
                  left = 0 + params. leftMargin;
                  top = getHeight()-childMeasureHeight-params.bottomMargin ;
                   break;
             case CustomLayoutParams. POSITION_RIGHTANDBOTTOM:// 右下角
                  left = getWidth()-childMeasureWidth - params.rightMargin;
                  top = getHeight()-childMeasureHeight-params.bottomMargin ;
                   break;
             default:
                   break;
            }
     
             // 确定子控件的位置，四个参数分别代表（左上右下）点的坐标值
            child.layout(left, top, left+childMeasureWidth, top+childMeasureHeight);
      }
      
}
```

</br>

**运行效果：**

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwNTI1MTcyNjIwODc1)

</br>

好了，就写到这里，如果想尝试设置其他属性，比如above、below等，感兴趣的同学可以尝试一下哦~。其实也没什么难的，无非就是如果布局属性定义的多，那么在onMeasure和onLayout中考虑的问题就更多更复杂，自定义布局容器就是根据自己的需求，让容器满足我们特殊的摆放要求。

总结一下今天学习的内容，这篇博客主要学习了两个知识点：

**自定义ViewGroup的步骤：**
1. 继承`ViewGroup`，覆盖构造方法  
2. 重写`onMeasure()`方法测量子控件和自身宽高  
3. 实现`onLayout()`方法摆放子控件

**为布局容器自定义布局属性：**
1. 大致明确布局容器的需求，初步定义布局属性
2. 继承`LayoutParams`，定义布局参数类
3. 重写获取布局参数的方法
4. 在布局文件中使用布局属性
5. 在`onMeasure()`和`onLayout()`中使用布局参数

</br>

各位童鞋有什么疑问或者建议欢迎留言，有你的支持让我们共同进步，如果觉得写的不错，那就顶我一下吧·

</br>

**源码下载**

[https://github.com/openXu/View-CustomLayout](https://github.com/openXu/View-CustomLayout)