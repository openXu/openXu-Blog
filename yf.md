
https://claop.github.io/courses/markdown/

## 字体

&emsp;&emsp;&emsp;&emsp;&emsp; **加粗**     *斜体*    ***斜体加粗***    ~~删除线~~ 

### 设置字体、字号与颜色

> Markdown 是一种可以使用普通文本编辑器编写的标记语言，通过类似 HTML 的标记语法，它可以使普通文本内容具有一定的格式。但是它本身是不支持修改字体、字号与颜色等功能的！
> CSDN-markdown 编辑器是其衍生版本，扩展了 Markdown 的功能（如表格、脚注、内嵌HTML等等）！对，就是内嵌HTML，接下来要讲的功能就需要使用内嵌HTML的方法来实现。
字体，字号和颜色编辑如下代码：

```
<font face="黑体">我是黑体字</font>
<font face="微软雅黑">我是微软雅黑</font>
<font face="STCAIYUN">我是华文彩云</font>
<font color=#0099ff size=7 face="黑体">color=#0099ff size=72 face="黑体"</font>
<font color=#00ffff size=72>color=#00ffff</font>
<font color=gray size=72>color=gray</font>

// Size：规定文本的尺寸大小。可能的值：从 1 到 7 的数字。浏览器默认值是 3

```
<font face="黑体">我是黑体字</font>
<font face="微软雅黑">我是微软雅黑</font>
<font face="STCAIYUN">我是华文彩云</font>
<font color=#0099ff size=7 face="黑体">color=#0099ff size=72 face="黑体"</font>
<font color=#00ffff size=72>color=#00ffff</font>
<font color=gray size=72>color=gray</font>

## 添加背景色

> Markdown 本身不支持背景色设置，需要采用内置 HTML 的方式实现：借助 table, tr, td 等表格标签的 bgcolor 属性来实现背景色的功能。举例如下：

```
<table><tr><td bgcolor=#FF4500>这里的背景色是：OrangeRed，  十六进制颜色值：#FF4500， rgb(255, 69, 0)</td></tr></table>
```

<table><tr><td bgcolor=#FF4500>这里的背景色是：OrangeRed，  十六进制颜色值：#FF4500， rgb(255, 69, 0)</td></tr></table>

## 引用

> 这是引用的内容
>> 这是引用的内容
>>>>>>>>>> 这是引用的内容

## 分割线

> 三个或者三个以上的 - 或者 * 都可以

---
----
***
*****

## 图片

> ![图片alt](图片地址 "图片title")

github图床 ： https://github.com/用户名/仓库名/raw（不动）/master（不动）/文件夹名（没有请忽略）/文件名.后缀名

![例如](https://github.com/openXu/Blog/raw/master/blog_android/pic01/pic1.png "title")

## 超链接

> [超链接名](超链接地址 "超链接title") title可加可不加

> 注：Markdown本身语法不支持链接在新页面中打开，如果想要在新页面中打开的话可以用html语言的a标签代替`<a href="超链接地址" target="_blank">超链接名</a>`

<a href="超链接地址" target="_blank">新页面中打开超链接</a>

[百度](http://baidu.com)

## 列表

> 无序列表用 - + * 任何一种都可以，注意：- + * 跟内容之间都要有一个空格
> 有序列表：数字加点
> 列表嵌套：上一级和下一级之间敲三个空格即可

- 无序列表
- 无序列表

1. 有序列表
2. 有序列表

- 列表嵌套
   - 列表嵌套

## 表格

|表头|表头|表头|
|---|:--:|---:|
|内容|内容|内容|
|内容|内容|内容|


## 代码

> 单行代码：代码之间分别用一个反引号包起来
> 代码块：代码之间分别用三个反引号包起来，且两边的反引号单独占一行

`代码内容`

```
  代码...
  代码...
  代码...
```

## 流程图

```flow
st=>start: 开始
op=>operation: My Operation
cond=>condition: Yes or No?
e=>end
st->op->cond
cond(yes)->e
cond(no)->op
&```