> 版权声明：本文为openXu原创文章[【openXu的博客】](http://blog.csdn.net/xmxkf)，未经博主允许不得以任何形式转载


[](https://bintray.com账号openXu   a19921125 OPEN-XU   a19921125)
gradlew clean build bintrayUpload -PbintrayUser=BINTRAY_USERNAME -PbintrayKey=BINTRAY_KEY -PdryRun=false

gradlew clean build bintrayUpload -PbintrayUser=OPEN-XU -PbintrayKey=2309af22ccabc7ddc2e957de0577a26997d48de5 -PdryRun=false



目录：

@[toc]

&emsp;&emsp;在我们工作过程中，一定有不少同学自己写了一些框架性的东西，或者一些好用的工具、一些自定义的控件，总之就是能复用的代码。然而也有不少同学为了复用这些代码不得不复制粘贴到不同项目中，这样相同的功能出现了多份代码，在后期的维护过程中极为不便，修复其中一处bug将要改动几份代码。为了解决这些问题，我们不妨将这些代码部署到maven仓库，一处编写多处使用。


## 1、Maven是什么？

&emsp;&emsp;Maven这个单词来自犹太语，意为只是的积累，最初在Jakata Turbine项目中用来简化构建过程。当时有一些项目，仅有细微的差别，于是希望有一种标准化的方式构建项目，一个清晰的方式定义项目的组成，一个容易的方式发布项目信息，以及一种简单的方式在多个项目中共享jar包。Maven主要目标是让项目可重复使用、易维护、更容易理解... 说人话emm  &emsp;&emsp;   Maven是一个项目管理工具，我们主要用于存放一些类库、插件等方便共享。


## 2、Maven仓库在哪里？

&emsp;&emsp;我的理解，Maven就像是svn和git，我们项目开发过程中，通常会使用这些版本控制工具托管代码，svn和git使得我们清楚的了解两个版本之间的差异。以git为例，我们项目根目录下通常会有个.git文件夹，这个文件夹就是git的本地仓库，如果我们需要远程开发，通常需要将代码推到远程仓库中，比如github、码云等等。版本控制工具作用重在区别版本之间的差异，而Maven作用重在共享依赖库。

&emsp;&emsp;Maven仓库和git一样，可以放在本地，可以放在服务器（私有或者公有）。下面，我们用一个项目演示一下三种Maven仓库的构建及使用方法。

## 3、本地仓库

&emsp;&emsp;有一个自定义的控件类库，需要在不同的项目中使用，我可以将类库拷贝到这些项目中，项目结构如下：

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612172654615?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

&emsp;&emsp;然后在app/build.gradle中添加依赖implementation project(':OXViewLib')，这样就可以在项目中使用了。突然有一天我发现了一个bug，并把它修复了，但是只有一个项目中的类库修复，其他的项目中类库还是有bug，于是我不得不将lib代码复制到不同项目库中，这样还算是共享库吗？

&emsp;&emsp;为了解决这个问题，我们可以将这个类库发布到本地Maven仓库中，然后在不同的项目中依赖此类库即可。下面是发布本地仓库流程：

### 3.1 . uploadArchives

&emsp;&emsp;`uploadArchives`是一个发布类库到中央仓库的Task，我们需要为它指定本地仓库路径以及类库的一些信息，在类库模块的build.gradle中添加task
```xml
apply plugin: 'maven'

uploadArchives{
    repositories.mavenDeployer{
        // 配置本地仓库路径，项目根目录下的repository目录中
        repository(url: uri('../repository'))
        pom.groupId = "com.openxu.viewlib"// 唯一标识（通常为模块包名，也可以任意）
        pom.artifactId = "OXViewLib" // 项目名称（通常为类库模块名称，也可以任意）
        pom.version = "1.0.0" // 版本号
    }
}
```
&emsp;&emsp;配置完成后点击**Sync Now**，然后在Gradle projects窗口中:OXViewLib->Tasks中会多出一个upload目录，里面就有一个名为`uploadArchives`的task，这个`uploadArchives`就是将类库发布到仓库的task。
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612173013729?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 3.2 . 执行uploadArchives

&emsp;&emsp;执行uploadArchives有两种方式，一种是直接双击上面截图中的task；另一种是在Terminal中输入 `gradlew uploadArchives` 然后回车。执行完成后，在项目根目录下 多出一个**repository**目录，目录结构就是上面再`build.gradle`中的配置。其中**OXViewLib-1.0.0.arr**就是我们需要的类库，到此为止就完成了本地Maven仓库的发布。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612173123380?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 3.3 . 使用本地仓库中的类库

&emsp;&emsp;在使用的时候，我们需要告诉gradle构建工具去哪里找到我们所需要的类库，在项目工程的build.gradle中，我们需要告诉gradle本地maven仓库的地址：

```xml
allprojects {
    repositories {
        google()
        jcenter()
        //本地Maven仓库地址
        maven { 
            url 'file://E://ASWorkSpace//openxu//OXChart//repository' 
        }
    }
}
```

&emsp;&emsp;需要注意的是，在build.gradle中有两个地方可以配置repository（仓库）。`buildscript`里面配置的是gradle构建项目时需要的插件的maven仓库；而`allprojects`中是项目依赖库的仓库配置。如果将我们的仓库地址配置到`buildscript`中，将会导致依赖库找不到**Failed to resolve: com.openxu.viewlib:OXViewLib:1.0.0**。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612173413888?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

&emsp;&emsp;然后在app下build.script中添加依赖`implementation 'com.openxu.viewlib:OXViewLib:1.0.0'`
：
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/2018061217351540?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

到此为止本地Maven仓库就介绍完毕。

## 4、局域网私有仓库

&emsp;&emsp;将类库发布到本地仓库后，我们可以方便的在多个项目中使用同一个类库，但是，日常工作中通常是一个团队在开发，我将仓库发布在本地，队友根本访问不了。如果将仓库搭建在公司内网服务器上，整个公司的成员就都能访问了，最常用的搭建Maven私有仓库的工具Nexus就可以为我们解决这个问题，按照步骤一步步来吧：

### 4.1 . 下载Nexus

&emsp;&emsp; 点击右上角TRY NEXUS-->选择操作系统-->DOWNLOAD：

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612173650780?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

&emsp;&emsp;选择带管理Maven的版本nexus-2.14.8-01-bundle：

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612173719661?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

&emsp;&emsp;下载完成后，解压，选择不同操作系统对应的目录，由于我电脑为windows 64位，所以选择nexus-2.14.8-01-bundle\nexus-2.14.8-01\bin\jsw\windows-x86-64，此目录下的.bat文件即为nexus的批处理文件。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612173827593?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 4.2 . 运行

&emsp;&emsp; 双击**console-nexus.bat**启动nexus，在浏览器中访问 http://127.0.0.1:8081/nexus/，如果出现下图所示则为启动成功，点击Repositories查看Maven仓库管理界面。
        
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612173922103?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

&emsp;&emsp; 可以在conf\nexus.properties文件中修改nexus的端口号：

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612174001364?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


### 4.3 . 发布

&emsp;&emsp;首先我们要清楚，我们需要发布什么东西到maven仓库中。在Android Studio项目中依赖模块后，Android Studio会将被依赖的模块打包成.arr包，这个包就是被依赖的包，其中包含了编译后的java文件和一些资源文件。`.arr`和`.jar`的性质是一样的，无非`.arr`多了一些资源文件，这是google为android设计的一种也有的lib格式，我们需要共享的正是这个`.arr`文件。

&emsp;&emsp;在需要发布的Modul下`build.gradle`中添加`uploadArchives`任务（nexus默认的用户名和密码分别是：**admin**和**admin123**）：

```xml
/**②.发布到私有服务器maven仓库*/
apply plugin: 'maven'

//打包main目录下代码和资源的 task
task androidSourcesJar(type: Jar) {
    classifier = 'sources'
    from android.sourceSets.main.java.srcDirs
}
//配置需要上传到maven仓库的文件
artifacts {
    archives androidSourcesJar
}
//上传到Maven仓库的task
uploadArchives {
    repositories {
        mavenDeployer {
            //指定maven仓库url
            repository(url: "http://localhost:8081/nexus/content/repositories/releases/") {
            //nexus登录默认用户名和密码
            authentication(userName: "admin", password: "admin123")
            }
            pom.groupId = "com.openxu.viewlib"// 唯一标识（通常为模块包名，也可以任意）
            pom.artifactId = "OXViewLib" // 项目名称（通常为类库模块名称，也可以任意）
            pom.version = "1.0.0" // 版本号
        }
    }
}
```

&emsp;&emsp;发现跟发布到本地maven仓库的配置差不多，就是一个`uploadArchives`任务，最大的区别就是指定的**repository**仓库地址不一样而已。接下来执行`uploadArchives`任务（双击 or `gradlew uploadArchives`），**BUILD SUCCESSFUL**后，我们使用浏览器看看仓库中发布成功的依赖包：

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612174410826?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

&emsp;&emsp;浏览器右上角**Log In**点击登录（admin/ admin123）后可以在浏览器上操作上面仓库：

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612174442133?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 4.4 . 使用私有仓库中的依赖包

&emsp;&emsp;和使用本地仓库依赖一样，我们告诉gradle依赖包仓库的位置，在项目根目录下`build.gradle`中添加：

```xml
allprojects {
    repositories {
        jcenter()
        //本地仓库地址
        // maven { url 'file://E://ASWorkSpace//openxu//OXChart//repository' }
        //私有服务器仓库地址
        maven { 
            url 'http://localhost:8081/nexus/content/repositories/releases/' 
        }
    }
}
```

&emsp;&emsp; 然后在app模块build.gradle中添加依赖编译运行成功：

```xml
implementation 'com.openxu.viewlib:OXViewLib:1.0.0'
```

## 5、发布到JCenter

&emsp;&emsp;局域网私有仓库能解决团队内部资源共享，然而你觉得你写的框架或者类库很牛逼，不仅仅想让队友使用，还想让更多的开发者用到这么好的东西，这时候就需要将项目发布到中央仓库中，比如**mavenCentral**、**jcenter**等。Android Studio最开始将 Maven Central作为默认仓库，但由于某些原因，从Android Studio 0.8开始将**JCenter**作为默认仓库。不管选用那个仓库，他们的作用和目的都是一样的，就像你想买手机，选择华为还是小米。接下来我们以Jcenter为例，看看怎么将项目发布到中央仓库中。

### 5.1 . 注册账号

&emsp;&emsp; 首先你需要注册一个Bintray账号，注意此处需要点击**Sign Up Here**注册（或者直接访问https://bintray.com/signup/oss） 。绿色按钮是注册企业账号的，需要付费，有试用期，我们需要注册开发者账户，有很多文章没有标明这个问题。如果有人错误注册为企业账号，可从Edit Profile-->Delete Account删除后重新注册。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/2018061217482685?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 5.2 . 配置

&emsp;&emsp; 网上有很多关于上传到JCenter的文章，如果根据这些文章一步步走下去或多或少都会出现一些意想不到的问题，可能某一个问题就会缠绕你一整天，包括你现在读到我的文章，我也不能确保你能顺利一次成功，遇到问题找到问题的根源才能解决问题。上传到JCenter需要使用一些插件（gradle中配置），一般有两种方式：

 - **gradle-bintray-plugin**
	
	这个插件的地址 https://github.com/bintray/gradle-bintray-plugin，使用此插件的人非常多，但是gradle配置内容较多，稍不小心就会出问题，这也就导致很多人根据别人的文章一步步走一般都不会成功了。

 - **bintray-release**
	
	地址https://github.com/novoda/bintray-release，此插件相比上面的就简单很多了，主要是配置内容少了很多，即使出了问题解决也相对容易。

&emsp;&emsp; 这两个插件相关配置在各自插件的github上都有步骤，感兴趣的可以看看。既然两个插件的目标都是一致的，我们就选用简单点的试试吧。

#### Step 1

&emsp;&emsp; 在项目的build.gradle中buildscript中添加如下脚本，<latest-version>使用最新版本号，在https://github.com/novoda/bintray-release查看：

&emsp;&emsp; &emsp;&emsp; ![这里写图片描述](https://img-blog.csdn.net/20180612175205182?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

```xml

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        //classpath 'com.novoda:bintray-release:<latest-version>'
        classpath 'com.novoda:bintray-release:0.8.1'
    }
}
```

#### Step 2

&emsp;&emsp; 在库模块（需要上传的模块）的`build.gradle`中添加：

```xml
// must be applied after your artifact generating plugin (eg. java / com.android.library)
apply plugin: 'com.novoda.bintray-release'

publish {
    userOrg = '组织ID' //bintray账户下某个组织id
    groupId = 'com.openxu.viewlib' //maven仓库下库的包名，一般为模块包名
    artifactId = 'OXViewLib' //项目名称
    publishVersion = '1.0.2' //版本号
    desc = 'custom chart for android' //项目介绍，可以不写
    website = '' //项目主页，可以不写
}
```

**注意：**

 - `groupId`、`artifactId`、`publishVersion`就是使用时依赖的组成；
 - `userOrg`不是用户名，网上很多文章直接写的用户名，是因为老版本，在bintray网站升级之后，增加了组织的概念，需要将仓库放到某个组织下，所以在新版本中这里要填写组织id，如果你的账户下没有组织，需要创建一个。可以点击账户下Create Organization-->Create new organization创建，也可以点击Manage Organizations-->New Organization-->Create new organization创建。创建完成后，将组织ID填写到userOrg中

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612175530376?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612175558620?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

&emsp;&emsp; 组织创建完成之后，我们还需要在组织下创建一个maven仓库，步骤如下：

&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612175645461?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612175658183?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612175720432?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

创建完成之后，就可以继续下面的步骤了


#### Step 3

&emsp;&emsp; 上传，首先我们需要获取API Key，Bintray网站点击右上角用户名-->Edit Your Profile -> API Key -->输入密码-->Submit-->Show。

&emsp;&emsp; &emsp;&emsp; ![这里写图片描述](https://img-blog.csdn.net/20180612175828937?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

&emsp;&emsp; 在Android Studio的Terminal面板中执行下面命令，其中BINTRAY_USERNAME替换为你的binatray用户名，BINTRAY_KEY替换为上面获取的API Key，-PdryRun=false会上传到仓库中，如果为true，只会执行gradle任务，但不会上传。替换完成后回车执行

```xml
gradlew clean build bintrayUpload -PbintrayUser=BINTRAY_USERNAME -PbintrayKey=BINTRAY_KEY -PdryRun=false
```

&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612180016265?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

&emsp;&emsp;执行完成看到**BUILD SUCCESSFUL**就算是成功了。这时候点击组织下maven仓库（或者直接访问`https://bintray.com/组织ID/maven`）就可以看到上传的类库：

&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612180102432?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


#### Step 4

&emsp;&emsp; 使用，我们需要告诉gradle依赖包仓库的位置，在项目根目录下·build.gradle·中添加：

```xml
allprojects {
    repositories {
    google()
    jcenter()
    //本地仓库地址
    // maven { url 'file://E://ASWorkSpace//openxu//OXChart//repository' }
    //私有服务器仓库地址
    // maven { url 'http://localhost:8081/nexus/content/repositories/releases/' }
    //https://bintray.com仓库地址
    maven { url 'https://dl.bintray.com/oxo1/maven' }
    }
}
```

&emsp;&emsp; 和然后在app模块build.gradle中添加依赖：

```xml
implementation 'com.openxu.viewlib:OXViewLib:1.0.0'
```

&emsp;&emsp; 添加完成之后编译就通过了，但是上面仓库地址和依赖是从哪里来的呢？访问我们上传成功的项目页面，可以看到相关信息。如果我们将项目添加到JCenter中，以后依赖就不需要添加`maven { url 'https://dl.bintray.com/oxo1/maven' }`了，当然还是需要告诉gradle依赖库的仓库位置，只需要添加`jcenter()`就行了。
      
&emsp;&emsp; &emsp;&emsp; ![这里写图片描述](https://img-blog.csdn.net/20180612180310309?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


#### Step 5

&emsp;&emsp; 添加到JCenter。点击“Add to JCenter”，填写项目介绍，点击Send发送，然后等待审核，审核成功之后会发送站内通知：

&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612180447343?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

审核非常快，站内通知如下：

&emsp;&emsp;&emsp;&emsp;![这里写图片描述](https://img-blog.csdn.net/20180612180508232?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAxNjM0NDI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

&emsp;&emsp;以上，3种maven仓库的操作基本完成了，当然过程不可能这么顺利，很多人在这个过程中都会遇到不少问题，不同的人问题不一样，网上也有很多文章，这里感谢踩坑的前辈，我就不再赘述，这个流程是我披荆斩棘最后成功的流程，如果让我按照上面的流程再操作一次应该会一次成功，那有同学按照这个步骤出现问题，可以留言。

>欢迎关注，希望在这里有你想要的，博主会持续更新高(di)质(ji)量(shu)的文章和大家交流学习

>喜欢请点赞，no爱请勿喷~O(∩_∩)O谢谢

## 源码下载

[OXChart](https://github.com/openXu/OXChart)