> 转载请标明出处： 
https://www.jianshu.com/p/c2de330789a9
本文出自:[【openXu的博客】](https://www.jianshu.com/u/cf3a4e7531f8)


# 一. Android Stutio配置git
setting-->Version Control-->Git-->Path to Git executable中选择git.exe的位置，这个Stutio一般会默认配置好：

&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/1.png)

配置完路径后点击后面的Test按钮，出现下面提示框则表示配置成功：

&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/2.png)



# 二. 将项目分享到github
## 1. 设置github账号密码
&emsp;&emsp;打开Setting-->Version Control-->GitHub，填写完账号密码后，点击Test测试，如果连接成功会弹出如下提示框：

&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/3.png)

## 2. share project on github

&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/4.png)


输入仓库名和，描述，点击share:


&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/5.png)

&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/6.png)

# 三. 其他git托管平台（以CSDN上的CODE为例）
## 1. 为项目创建git仓库

&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/7.png)


## 2. 将项目中的文件纳入到仓库中（add）
&emsp;&emsp;创建仓库之后，工程中的文件都会变成红色，表示没有添加到仓库中去，接下来，我们将工程下的所有文件add到仓库中：


&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/8.png)

## 3. 提交到本地仓库（commit）
&emsp;&emsp;add成功之后，发现文件名变成了绿色，表示添加成功，下面将添加的文件提交到本地仓库中：工程右键-->Git-->Commit Directory

&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/9.png)

在弹出的窗口中，选择需要提交的文件，在下面填写提交信息，然后点击Commit：

&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/10.png)

提交时可能会弹出一些警告信息提示框，不用管它，继续点击commit就行。

## 4. push到远程仓库


&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/11.png)

&emsp;&emsp;由于现在还没有关联远程仓库，点击Define remote，将你的远程git地址填入URL中（在这之前，我们先进入到自己的CSDN CODE栏目中创建新项目，然后复制仓库地址），点击OK：

&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/12.png)

&emsp;&emsp;第一次可能需要填写密码（这个密码是在第一次Stutio配置Git的时候设置的，具体我也记不太清，反正我的所有git相关的密码都设置一个就行了，碰见需要输入密码就输那一个）：

&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/13.png)

登录CSDN ：

&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/14.png)

## 5. push失败，需要先pull
&emsp;&emsp;push失败，出现被拒绝的警告，这可能是远程仓库中的版本和你本地仓库的版本不一致造成，所以在push之前，需要pull一次：

&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/15.png)

so，pull，项目右键Git-->Repository-->Pull，然后勾选origin/master，点击pull按钮:

&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/16.png)

## 6. Merge合并
&emsp;&emsp;如果远程库很本地库中有冲突，需要Merge合并，点击Merge：


&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/17.png)

&emsp;&emsp;左边是你本地文件的样子，最右边是远程库的版本，中间就是本地仓库中版本的样子，也就是最终合并的结果（可以编辑），将需要的代码复制到中间Result栏，删除废弃的代码，然后点击所有的X，表示合并完成：

&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/18.png)

合并完成之后，会弹出提示框。接下来点击Apply：

&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/19.png)

接下来在继续push，注意应该选择Commit and Push，要不然你就要先Commit然后再Push：

&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/20.png)


push成功弹出提示框：


&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/21.png)

在下面Version Control中，可以查看提交的log信息：


&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/22.png)


## 7. CSDN协同开发
&emsp;&emsp;如果我们的项目需要多人开发，可以在项目设置中邀请别人，如果不邀请，他就不是项目成员，如果你创建的是公开库，他只能pull，不能push的：

&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/23.png)

# 四. 解除关联
&emsp;&emsp;如果希望项目解除git关联，只需要 Settrings -> Version Control 删掉关联就行了：

&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog1/pic/24.png)

到此为止，相信大家都会在Stutio中使用Git了，如果有什么问题，请留言，我会尽快回复，如果对你有帮助

别忘了  点赞！！！O(∩_∩)O谢谢










