> 版权声明：本文为openXu原创文章[【openXu的博客】](http://blog.csdn.net/xmxkf)，未经博主允许不得以任何形式转载

**目录**

@[toc]

&emsp;&emsp;在上一篇博客[《Android自定义View（二、初深入解析自定义属性）》](http://blog.csdn.net/xmxkf/article/details/51468648)中我们比较深入的探讨了自定义属性。首先，我们把自定义View的步骤搬上来，一切按套路干活：

> 1. 继承View，覆盖构造方法
> 2. 自定义属性
> 3. 重写onMeasure方法测量宽高
> 4. 重写onDraw方法绘制控件

&emsp;&emsp;不知大家还记不记得，在我的第一篇博客[《Android自定义View（一、初体验）》](http://blog.csdn.net/xmxkf/article/details/51454685)中我们已经接触过onMeasure方法，其中我们学习过`MeasureSpec`这个类，但不是很全面。很多人认为自定义控件中最不能理解的就是`onMeasure`方法了，只知道它是用来测量控件大小的，至于什么时候测量，怎么测量就不太清楚了。这篇博客我们就深入学习onMeasure相关的知识点，带着问题，我们一步步揭开onMeasure的神秘面纱。


# 1. onMeasure什么时候会被调用

&emsp;&emsp;`onMeasure()`方法的作用时测量空间的大小，什么时候需要测量控件的大小呢？我们举个栗子，做饭的时候我们炒一碗菜，炒菜的过程我们并不要求知道这道菜有多少分量，只有在菜做熟了我们要拿个碗盛放的时候，我们才需要掂量拿多大的碗盛放，这时候我们就要对菜的分量进行估测。

&emsp;&emsp;而我们的控件也正是如此，创建一个`View`(执行构造方法)的时候不需要测量控件的大小，只有将这个view放入一个容器（父控件）中的时候才需要测量，而这个测量方法就是父控件唤起调用的。当控件的父控件要放置该控件的时候，父控件会调用子控件的`onMeasure()`方法询问子控件：“你有多大的尺寸，我要给你多大的地方才能容纳你？”，然后传入两个参数（`widthMeasureSpec`和`heightMeasureSpec`），这两个参数就是父控件告诉子控件可获得的空间以及关于这个空间的约束条件（好比我在思考需要多大的碗盛菜的时候我要看一下碗柜里最大的碗有多大，菜的分量不能超过这个容积，这就是碗对菜的约束），子控件拿着这些条件就能正确的测量自身的宽高了。

</br>

# 2. onMeasure方法执行流程

&emsp;&emsp;上面说到`onMeasure()`方法是由父控件调用的，所有父控件都是`ViewGroup`的子类，`ViewGroup`是一个抽象类，它里面有一个抽象方法onLayout，这个方法的作用就是摆放它所有的子控件（安排位置），因为是抽象类，不能直接new对象，所以我们在布局文件中可以使用`View`但是不能直接使用 `ViewGroup`。

&emsp;&emsp;在给子控件确定位置之前，必须要获取到子控件的大小（只有确定了子控件的大小才能正确的确定上下左右四个点的坐标），而`ViewGroup`并没有重写`View`的`onMeasure`方法，也就是说抽象类`ViewGroup`没有为子控件测量大小的能力，它只能测量自己的大小。但是既然`ViewGroup`是一个能容纳子控件的容器，系统当然也考虑到测量子控件的问题，所以`ViewGroup`提供了三个测量子控件相关的方法(measuireChildren\measuireChild\measureChildWithMargins)，只是在ViewGroup中没有调用它们，所以它本身不具备为子控件测量大小的能力，但是他有这个潜力哦。

&emsp;&emsp;为什么都有测量子控件的方法了而ViewGroup中不直接重写onMeasure方法，然后在onMeasure中调用呢？因为不同的容器摆放子控件的方式不同，比如RelativeLayout，LinearLayout这两个ViewGroup的子类，它们摆放子控件的方式不同，有的是线性摆放，而有的是叠加摆放，这就导致测量子控件的方式会有所差别，所以ViewGroup就干脆不直接测量子控件，他的子类要测量子控件就根据自己的布局特性重写onMeasure方法去测量。这么看来ViewGroup提供的三个测量子控件的方法岂不是没有作用？答案是NO，既然提供了就肯定有作用，这三个方法只是按照一种通用的方式去测量子控件，很多ViewGruop的子类测量子控件的时候就使用了ViewGroup的measureChildxxx系列方法；还有一个作用就是为我们自定义ViewGroup提供方便咯，自定义ViewGroup我会在以后的博客中专门探讨，这里就不大费篇章了。

&emsp;&emsp;测量的时候父控件的onMeasure方法会遍历他所有的子控件，挨个调用子控件的measure方法，measure方法会调用onMeasure，然后会调用setMeasureDimension方法保存测量的大小，一次遍历下来，第一个子控件以及这个子控件中的所有子控件都会完成测量工作；然后开始测量第二个子控件…；最后父控件所有的子控件都完成测量以后会调用setMeasureDimension方法保存自己的测量大小。值得注意的是，这个过程不只执行一次，也就是说有可能重复执行，因为有的时候，一轮测量下来，父控件发现某一个子控件的尺寸不符合要求，就会重新测量一遍。

举个栗子，看下图：

&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwNTI0MTQwNzM0NDAy) ![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwNTI0MTQwNzQzODI1)

下面是测量的时序图：

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwNTI0MTQwOTM5NzAz)

</br>

# 3. MeasureSpec类
# 
&emsp;&emsp;上面说到`MeasureSpec`约束是由父控件传递给子控件的，这个类里面到底封装了什么东西？我们看一看源码：

```java
public static class MeasureSpec {
    private static final int MODE_SHIFT = 30;
    private static final int MODE_MASK = 0x3 << MODE_SHIFT;
    /**
     * 父控件不强加任何约束给子控件，它可以是它想要任何大小
     */
    public static final int UNSPECIFIED = 0 << MODE_SHIFT;
    /**
     * 父控件已为子控件确定了一个确切的大小，孩子将被给予这些界限，不管子控件自己希望的是多大
     */
    public static final int EXACTLY = 1 << MODE_SHIFT;
    /**
     * 父控件会给子控件尽可能大的尺寸
     */
    public static final int AT_MOST = 2 << MODE_SHIFT;

    /**
     * 根据所提供的大小和模式创建一个测量规范
     */
    public static int makeMeasureSpec(int size, int mode) {
        if (sUseBrokenMakeMeasureSpec) {
            return size + mode;
        } else {
            return (size & ~MODE_MASK) | (mode & MODE_MASK);
        }
    }
    /**
     * 从所提供的测量规范中提取模式
     */
    public static int getMode(int measureSpec) {
        return (measureSpec & MODE_MASK);
    }
    /**
     * 从所提供的测量规范中提取尺寸
     */
    public static int getSize(int measureSpec) {
        return (measureSpec & ~MODE_MASK);
    }
    ...
}

```

</br>

&emsp;&emsp;从源码中我们知道，**MeasureSpec** 其实就是尺寸和模式通过各种位运算计算出的一个整型值，它提供了三种模式，还有三个方法（合成约束、分离模式、分离尺寸）。在[《Android自定义View（一、初体验）》](http://blog.csdn.net/xmxkf/article/details/51454685)（如果不太了解的可以参考这篇博客）这篇博客中，我们得到了这三种模式对应的意思和布局文件中的参数值的对应关系（父控件填充屏幕*MATCH_PARENT*的情况，如果其他情况，请参考下面getChildMeasureSpec方法的结果）：


| **约束** | **布局参数** | **值** | **说明** |
| :-------:  |:-----: | :----: | :-----: |
| UNSPECIFIED(未指定) |   | 0| 父控件没有对子控件施加任何约束，子控件可以得到任意想要的大小（使用较少）。|
| EXACTLY(完全) |match_parent/具体宽高值| 1073741824 |父控件给子控件决定了确切大小，子控件将被限定在给定的边界里而忽略它本身大小。特别说明如果是填充父窗体，说明父控件已经明确知道子控件想要多大的尺寸了（就是剩余的空间都要了）</br>栗子：碗柜最大的碗就这么大，菜有多少都只能盛到这个最大的碗里，盛不下的我就管不了了（吃掉或者倒掉）|
| AT_MOST(至多)|  wrap-content  | -2147483648 |子控件至多达到指定大小的值。包裹内容就是父窗体并不知道子控件到底需要多大尺寸（具体值），需要子控件自己测量之后再让父控件给他一个尽可能大的尺寸以便让内容全部显示但不能超过包裹内容的大小</br>栗子：碗柜有各种大小的碗，菜少就拿小碗放，菜多就拿大碗放，但是不能浪费碗的容积，要保证碗刚好盛满菜|


# 4. 从ViewGroup的onMeasure()到View的onMeasure()

## ①. ViewGroup中三个测量子控件的方法

&emsp;&emsp;通过上面的介绍，我们知道，如果要自定义`ViewGroup`就必须重写`onMeasure()`方法，在这里测量子控件的尺寸。子控件的尺寸怎么测量呢？ViewGroup中提供了三个关于测量子控件的方法：

```java
 /**
  *遍历ViewGroup中所有的子控件，调用measuireChild测量宽高
  */
 protected void measureChildren (int widthMeasureSpec, int heightMeasureSpec) {
    final int size = mChildrenCount;
    final View[] children = mChildren;
    for (int i = 0; i < size; ++i) {
        final View child = children[i];
        if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
	        //测量某一个子控件宽高
            measureChild(child, widthMeasureSpec, heightMeasureSpec);
        }
    }
}
        
/**
* 测量某一个child的宽高
*/
protected void measureChild (View child, int parentWidthMeasureSpec,
       int parentHeightMeasureSpec) {
   final LayoutParams lp = child.getLayoutParams();
   //获取子控件的宽高约束规则
   final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
           mPaddingLeft + mPaddingRight, lp. width);
   final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
           mPaddingTop + mPaddingBottom, lp. height);

   child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
 
/**
* 测量某一个child的宽高，考虑margin值
*/
protected void measureChildWithMargins (View child,
       int parentWidthMeasureSpec, int widthUsed,
       int parentHeightMeasureSpec, int heightUsed) {
   final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
   //获取子控件的宽高约束规则
   final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
           mPaddingLeft + mPaddingRight + lp. leftMargin + lp.rightMargin
                   + widthUsed, lp. width);
   final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
           mPaddingTop + mPaddingBottom + lp. topMargin + lp.bottomMargin
                   + heightUsed, lp. height);
   //测量子控件
   child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```

&emsp;&emsp;这三个方法分别做了那些工作大家应该比较清楚了吧？`measureChildren()` 就是遍历所有子控件挨个测量，最终测量子控件的方法就是`measureChild()` 和`measureChildWithMargins()`了，我们先了解几个知识点：

* **measureChildWithMargins跟measureChild的区别就是父控件支不支持margin属性**

&emsp;&emsp;支不支持margin属性对子控件的测量是有影响的，比如我们的屏幕是1080x1920的，子控件的宽度为填充父窗体，如果使用了marginLeft并设置值为100；在测量子控件的时候，如果用measureChild，计算的宽度是1080，而如果是使用measureChildWithMargins，计算的宽度是1080-100 = 980。

* **怎样让ViewGroup支持margin属性？**

&emsp;&emsp;ViewGroup中有两个内部类`ViewGroup.LayoutParams`和`ViewGroup. MarginLayoutParams`，MarginLayoutParams继承自LayoutParams ，这两个内部类就是VIewGroup的布局参数类，比如我们在LinearLayout等布局中使用的layout_width\layout_hight等以“layout_ ”开头的属性都是布局属性。在`View`中有一个`mLayoutParams`的变量用来保存这个View的所有布局属性。

* **LayoutParams和MarginLayoutParams 的关系：**

`LayoutParams` 中定义了两个属性（现在知道我们用的layout_width\layout_hight的来头了吧？）：

```xml
<declare-styleable name= "ViewGroup_Layout">
    <attr name ="layout_width" format="dimension">
        <enum name ="fill_parent" value="-1" />
        <enum name ="match_parent" value="-1" />
        <enum name ="wrap_content" value="-2" />
    </attr >
    <attr name ="layout_height" format="dimension">
        <enum name ="fill_parent" value="-1" />
        <enum name ="match_parent" value="-1" />
        <enum name ="wrap_content" value="-2" />
    </attr >
</declare-styleable >
```

`MarginLayoutParams` 是`LayoutParams`的子类，它当然也延续了layout_width\layout_hight 属性，但是它扩充了其他属性：

```xml
< declare-styleable name ="ViewGroup_MarginLayout">
    <attr name ="layout_width" />   <!--使用已经定义过的属性-->
    <attr name ="layout_height" />
    <attr name ="layout_margin" format="dimension"  />
    <attr name ="layout_marginLeft" format= "dimension"  />
    <attr name ="layout_marginTop" format= "dimension" />
    <attr name ="layout_marginRight" format= "dimension"  />
    <attr name ="layout_marginBottom" format= "dimension"  />
    <attr name ="layout_marginStart" format= "dimension"  />
    <attr name ="layout_marginEnd" format= "dimension"  />
</declare-styleable >
```

是不是对布局属性有了一个全新的认识？原来我们使用的margin属性是这么来的。


* **为什么LayoutParams 类要定义在ViewGroup中？**

&emsp;&emsp;大家都知道ViewGroup是所有容器的基类，一个控件需要被包裹在一个容器中，这个容器必须提供一种规则控制子控件的摆放，比如你的宽高是多少，距离那个位置多远等。所以ViewGroup有义务提供一个布局属性类，用于控制子控件的布局属性。

* **为什么View中会有一个mLayoutParams 变量？**

&emsp;&emsp;我们在之前学习自定义控件的时候学过自定义属性，我们在构造方法中，初始化布局文件中的属性值，我们姑且把属性分为两种。一种是本View的绘制属性，比如TextView的文本、文字颜色、背景等，这些属性是跟View的绘制相关的。另一种就是以“layout_”打头的叫做布局属性，这些属性是父控件对子控件的大小及位置的一些描述属性，这些属性在父控件摆放它的时候会使用到，所以先保存起来，而这些属性都是ViewGroup.LayoutParams定义的，所以用一个变量保存着。

* **怎样让ViewGroup支持margin属性？**

&emsp;&emsp;这一部分知识点我们在下一篇博客《自定义ViewGroup》中再去讲解

</br>

## ②. getChildMeasureSpec()

&emsp;&emsp;`measureChildWithMargins()`跟`measureChild()`都调用了这个方法，其作用就是通过父控件的宽高约束规则和父控件加在子控件上的宽高布局参数生成一个子控件的约束。我们知道View的onMeasure方法需要两个参数（父控件对View的宽高约束），这个宽高约束就是通过这个方法生成的。有人会问为什么不直接拿着子控件的宽高参数去测量子控件呢？打个比方，父控件的宽高约束为wrap_content，而子控件为match_perent，是不是很有意思，父控件说我的宽高就是包裹我的子控件，我的子控件多大我就多大，而子控件说我的宽高填充父窗体，父控件多大我就多大。最后该怎么确定大小呢？所以我们需要为子控件重新生成一个新的约束规则。只要记住，子控件的宽高约束规则是父控件调用getChildMeasureSpec方法生成。

&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwNTI0MTQzODIzMTk4)

`getChildMeasure()`方法代码不多，也比较简单，就是几个switch将各种情况考虑后生成一个子控件的新的宽高约束，这个方法的结果能够用一个表来概括：

| 父控件的约束规则 | 子控件的宽高属性 | 子控件的约束规则 | 说明 |
|:--- |:-----|:-----|:--------|
| EXACTLY（父控件是填充父窗体，或者具体size值）|具体的size（20dip）/MATCH_PARENT|EXACTLY | 子控件如果是具体值，约束尺寸就是这个值，模式为确定的；</br>子控件为填充父窗体，约束尺寸是父控件剩余大小，模式为确定的。|
||WRAP-CONTENT|AT_MOST|子控件如果是包裹内容，约束尺寸值为父控件剩余大小 ，模式为至多|
|AT_MOST（父控件是包裹内容）|具体的size（20dip）|EXACTLY|子控件如果是具体值，约束尺寸就是这个值，模式为确定的；|
||MATCH_PARENT/WRAP_CONTENT|AT_MOST|子控件为填充父窗体或者包裹内容 ，约束尺寸是父控件剩余大小 ，模式为至多|
|UNSPECIFIED（父控件未指定）|具体的size（20dip）|EXACTLY|子控件如果是具体值，约束尺寸就是这个值，模式为确定的；|
||MATCH_PARENT/WRAP_CONTENT|UNSPECIFIED|子控件为填充父窗体或者包裹内容 ，约束尺寸0，模式为未指定|


进行了上面的步骤，接下来就是在`measureChildWithMarginsh()`或者`measureChild()`中 调用子控件的`measure()`方法测量子控件的尺寸了。

##  ③. View的onMeasure()
  
&emsp;&emsp;View中onMeasure方法已经默认为我们的控件测量了宽高，我们看看它做了什么工作：

```java
protected void onMeasure( int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension( getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
/**
 * 为宽度获取一个建议最小值
 */
protected int getSuggestedMinimumWidth () {
    return (mBackground == null) ? mMinWidth : max(mMinWidth , mBackground.getMinimumWidth());
}
/**
 * 获取默认的宽高值
 */
public static int getDefaultSize (int size, int measureSpec) {
    int result = size;
    int specMode = MeasureSpec. getMode(measureSpec);
    int specSize = MeasureSpec. getSize(measureSpec);
    switch (specMode) {
    case MeasureSpec. UNSPECIFIED:
        result = size;
        break;
    case MeasureSpec. AT_MOST:
    case MeasureSpec. EXACTLY:
        result = specSize;
        break;
    }
    return result;
}
```

从源码我们了解到：

* 如果View的宽高模式为未指定，他的宽高将设置为android:minWidth/Height =""值与背景宽高值中较大的一个；
* 如果View的宽高 模式为 EXACTLY （具体的size ），最终宽高就是这个size值；
* 如果View的宽高模式为EXACTLY  （填充父控件 ），最终宽高将为填充父控件；
* 如果View的宽高模式为AT_MOST （包裹内容），最终宽高也是填充父控件。

也就是说如果我们的自定义控件在布局文件中，只需要设置指定的具体宽高，或者`MATCH_PARENT` 的情况，我们可以不用重写`onMeasure()`方法。

但如果自定义控件需要设置包裹内容`WRAP_CONTENT` ，我们需要重写`onMeasure()`方法，为控件设置需要的尺寸；默认情况下`WRAP_CONTENT`的处理也将填充整个父控件。

##  ④. setMeasuredDimension()

&emsp;&emsp;`onMeasure()`方法最后需要调用`setMeasuredDimension()`方法来保存测量的宽高值，如果不调用这个方法，可能会产生不可预测的问题。

这篇博客我们学习了onMeasure方法测量控件大小的流程，以及里面执行的一些细节，总结一下知识点：

1. 测量控件大小是父控件发起的
2. 父控件要测量子控件大小，需要重写`onMeasure()`方法，然后调用`measureChildren()`或者`measureChildWithMargin()`
3. `onMeasure()`方法的参数是通过getChildMeasureSpec生成的
4. 如果我们自定义控件需要使用`wrap_content`，我们需要重写`onMeasure()`方法
5. 测量控件的步骤：
> 父控件`onMeasure`->`measureChildren`\`measureChildWithMargin`->`getChildMeasureSpec`->
> 子控件的`measure`->`onMeasure`->`setMeasureDimension`->
> 父控件`onMeasure`结束调用`setMeasureDimension`保存自己的大小


这篇博客的内容比较简单，稍微有点绕的就是`getChildMeasureSpec()`这个方法，我们可以结合我们平时写布局的经验，去推导约束规则，只要理解之后就不会感觉绕了。可能我写得不是很清楚，如果有什么建议欢迎留言，如果认为对你有帮助，那就点个赞呗~

