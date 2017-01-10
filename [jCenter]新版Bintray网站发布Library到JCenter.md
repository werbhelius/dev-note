# 新版Bintray网站发布Library到JCenter

> 作者：Tsy远

> 链接：http://www.jianshu.com/p/6a6eca8c24c4

本文介绍了Maven、JCenter、MavenCenter、JitPack、Bintray的概念以及如何在新版的Bintray网站上发布Library并提交到JCenter上

## 前言
由于Bintray网站增加了Organization的概念，所以我在发布Library的时候发现网上很多文章都已经过时了。网站样子发生了很大的变化。所以在这篇文章把如何在最新的Bintray上发布Library到JCenter上做个整理

## 1 什么是Maven、JCenter、MavenCenter、JitPack、Bintray？
相信很多人分不清这几个概念究竟代表什么，只知道跟着开源库的引入步骤走，比如如下的库，要你在root下的gradle中加一个maven地址，然后在app的gradle中加compile

![事例](http://upload-images.jianshu.io/upload_images/1594931-a6ea75a7739aac06.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在我们将Maven、JCenter、MavenCenter、JitPack、Bintray这几个概念分个类解释下

### Maven仓库
在用Eclipse+Ant组合的时候，我们往往引入一个库都是下载jar包或者aar包放到lib目录下，然后右键添加引用。
But！这并不友好，比如当升级版本库的时候往往需要下载新的包替换引用，非常麻烦。
所以，当升级到Android Studio + Gradle 组合后 gradle中提供了可以从远端拉取jar包和aar包引入本地。规则就是：

```

  compile 'com.tsy:pay:1.0.0'  //groupid:projectid:version
```

1. "com.tsy" 即GroupId，就是你在网上标识这是唯一标识你的一个组，就像Android里的包名一样

2. "pay" 就是我的项目名称

3. "1.0.0" 即版本号

这个概念我们懂了。但是这个下载源是哪呢，就是maven仓库。那maven仓库的地址是什么呢，是不是Android Studio都是从一个仓库获取包呢，这时候就需要了解 JCenter、MavenCenter、JitPack 了


### JCenter、MavenCenter、JitPack
这3个名词即具体的Maven仓库的地址，他们都是Maven仓库，但是属于不同的服务源。总的来说，只有两个标准的Android library文件服务器：Jcenter 和 Maven Central，现在JitPack也流行了起来。（比较方便）

从哪引用这几个Maven仓库呢，就是在根目录build.gradle中

```
  allprojects {
      repositories {
          jcenter()      //JCenter仓库
          mavenCenter()    //mvenCenter仓库
          maven { url "https://jitpack.io" }   // jitpack仓库
      }
  }
```
具体使用哪个要看开源项目把Library传到了哪个仓库。它就会要求你在这加哪个仓库。
起初，Android Studio 选择Maven Central作为默认仓库。如果你使用老版本的Android Studio创建一个新项目，mavenCentral()会自动的定义在build.gradle中。

但是Maven Central的最大问题是对开发者不够友好。上传library异常困难。上传上去的开发者都是某种程度的极客。同时还因为诸如安全方面的其他原因，Android Studio团队决定把默认的仓库替换成jcenter。正如你看到的，一旦使用最新版本的Android Studio创建一个项目，jcenter()自动被定义，而不是mavenCentral()

我们发现第三个jitpack的写法和前2个不一样，写法是maven {} 里面加入地址，其实这个才是maven仓库标准引用方法，jcenter和mavenCenter由于是标准的Android仓库，相当于定义了一个别名。
所以一些自定义的仓库都是这种写法然后填入自己的仓库网址，比如Fabric.io的library

```
  maven { url 'https://maven.fabric.io/public' }
```

### Bintray是什么
Bintray其实只是一个网站，他们负责维护JCenter这个库，就是说JCenter库是托管在Bintray网站上的。

但是Bintray不只只有JCenter库，每个人都可以在上面创建自己的账号，生成自己的maven仓库，比如我的账号tangsiyuan下面创建了一个名叫"maven"的maven仓库。那我的maven仓库地址就是
https://dl.bintray.com/tangsiyuan/maven
当然也可以再build中引入

```
  maven { url 'https://dl.bintray.com/tangsiyuan/maven' }
```

而JCenter仓库只是Bintray官方账户创建的一个maven仓库，地址是
https://jcenter.bintray.com

其实个人的仓库和JCenter是平级的，只不过JCenter被Android Studio设为了标准仓库。

## 2 如何在新版的Bintray网站上发布Library到JCenter上

上面把所有的概念都介绍清楚了。现在我们来介绍怎么把自己的Libary传到JCenter上。（mavenCenter已经过时了，jitpack很简单就不作介绍了）

### 2.1 完成自己的Library

发布的前提当然是自己的Library已经完成了。具体怎么写Library就不再赘述。给大家截个我的MyOKHttp的图就行

![png](http://upload-images.jianshu.io/upload_images/1594931-0521562d2057d51e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2.2 Bintray网站上创建账户
由于 Bintray网站 改版了，增加了Organization的概念，首页变成了这样

![Bintray首页](http://upload-images.jianshu.io/upload_images/1594931-1ddef1a62123cf16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对！就是中间那个大大的绿色按钮，用那个点了就错了！！！变成了注册一个组织，注册地址是 https://bintray.com/signup

![signup](http://upload-images.jianshu.io/upload_images/1594931-9ae89cbbd51fea05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

而注册个人的地址应该是 https://bintray.com/signup/oss

![signup-oss](http://upload-images.jianshu.io/upload_images/1594931-dfeb38abb2e2dbcb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

坑爹！重要的事情说3遍！！！

注册地址是 https://bintray.com/signup/oss

注册地址是 https://bintray.com/signup/oss

注册地址是 https://bintray.com/signup/oss

具体注册过程就不多说了，注意一点，好像不能用QQ邮箱注册

### 2.3 创建maven仓库
注册完成，激活邮箱，登录后创建一个maven仓库

![创建maven仓库](http://upload-images.jianshu.io/upload_images/1594931-d13ff3d56f798849.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

name写maven（因为上传的时候不指定的话默认仓库名是maven）
type选择maven

创建成功后就像上图我成功创建的maven仓库一样。

### 2.3 上传Library到自己创建的maven仓库
在这里我使用了开源库上传自己的Library

https://github.com/novoda/bintray-release

根目录build添加

```
buildscript {
  repositories {
      jcenter()
  }
  dependencies {
      classpath 'com.android.tools.build:gradle:2.2.2'
      classpath 'com.novoda:bintray-release:0.3.4'
  }
}
```

需要上传Library的build添加

```
apply plugin: 'com.novoda.bintray-release'

...

publish {
  userOrg = 'tangsiyuan'      //bintray注册的用户名
  groupId = 'com.tsy'         //compile引用时的第1部分groupId
  artifactId = 'myokhttp'     //compile引用时的第2部分项目名
  publishVersion = '1.0.0'    //compile引用时的第3部分版本号
  desc = 'This is a okhttp3 extend library'
  website = 'https://github.com/tsy12321/MyOkHttp'
}
```

最后打开Termainal执行命令

```
./gradlew clean build bintrayUpload -PbintrayUser=BINTRAY_USERNAME -PbintrayKey=BINTRAY_KEY -PdryRun=false

```

其中BINTRAY_USERNAME换成bintray注册的用户名，BINTRAY_KEY换成自己的APIKEY

APIKEY的查看如下

![APIKEY](http://upload-images.jianshu.io/upload_images/1594931-5fe373e35c9f901f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

回车执行命令，看到BUILD SUCCESS即上传成功

这时候我们可以打开maven仓库看到自己提交的项目

![maven仓库](http://upload-images.jianshu.io/upload_images/1594931-b5795c118f1d90dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2.4 提交到JCenter
在我们上传到自己maven仓库后其实就已经可以引用自己的库了。只要在root下的build加上自己maven地址

```
  maven { url 'https://dl.bintray.com/tangsiyuan/maven' }
```
然后在app的build中加上引用即可
```
  compile 'com.tsy:myokhttp:1.0.0'
```

![个人maven仓库](http://upload-images.jianshu.io/upload_images/1594931-f1ce268099e391b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点Sync，可以发现引用成功。

当然如果能够提交到JCenter就更好了，不再需要定义自己maven仓库地址，直接compile即可。
进入项目页，点击Add to JCenter

![Add to JCenter](http://upload-images.jianshu.io/upload_images/1594931-dda4e55bbe489ece.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后直接commit send就行（提交JCenter后groupID和在本地定义的一样，所以本地定义groupID要能标识个人，最好到 https://jcenter.bintray.com 看下有没有重复的包名

![commit to JCenter](http://upload-images.jianshu.io/upload_images/1594931-1dffe275b6f83363.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后等待Bintray审核通过。（我晚上提交，第二天就审核通过了）

通过后会有右上方小邮箱按钮提示信息，提示审核通过

![tips](http://upload-images.jianshu.io/upload_images/1594931-2de0c7f0baf2210b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

项目页信息多了个JCenter图标

![JCenter审核通过](http://upload-images.jianshu.io/upload_images/1594931-adf09d82e6b326bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 彩蛋
教一个高逼格的小技巧，Github上经常会看到2个小图标

![dvg](http://upload-images.jianshu.io/upload_images/1594931-7b3329e458507c36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

就是这两行代码

```
[![License](https://img.shields.io/badge/license-Apache%202-green.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Download](https://api.bintray.com/packages/tangsiyuan/maven/myokhttp/images/download.svg) ](https://bintray.com/tangsiyuan/maven/myokhttp/_latestVersion)

```

具体里面怎么替换就不多说咯。加上后感觉逼格立马提高！

最后安利一波自己的2个库，欢迎star、pr

## MyOkHttp
对Okhttp3进行二次封装,对外提供了POST请求、GET请求、PATCH请求、PUT请求、DELETE请求、上传文件、下载文件、取消请求、Raw/Json/Gson返回、后台下载管理等功能

https://github.com/tsy12321/MyOkHttp

## PayAndroid
对微信支付和支付宝支付的App端SDK进行二次封装，对外提供一个较为简单的接口和支付结果回调

https://github.com/tsy12321/PayAndroid
