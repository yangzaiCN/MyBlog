---
title: node方式搭建appium环境
categories:
- 自动化测试
tags:
- appium

---

### 需要准备的材料 ###
- java环境（相关教程非常多，这里就不赘述了）
- android环境
- node环境
- appium-doctor

### android环境 ###
android环境最麻烦的就是sdk了，我建议大家直接下载android studio,安装时会默认下载sdk。反正以后也要分析appium-uiautomator2-server工程，都得用studio，所以这里我们用这种方式安装。如果不想安装studio，可以下载*仅获取命令行工具*[单独的sdk](https://developer.android.com/studio/index.html?hl=zh-cn)但是需要科学上网,然后通过sdk manager命令行下载。

<!-- more -->

下载完成后我们需要配置ANDROID_HOME系统变量，不需要加入到path中。

### node环境 ###

简单说下，appium是基于node平台的express框架开发的，官方建议我们使用node方式管理appim，这样能够通过代码很方便的控制appium，而且方便收集日志。但是要求node版本要大于6，从appium的源码config.js可以看到：
```
function checkNodeOk () {
  let [major, minor] = getNodeVersion();
  if (major < 6) {
    let msg = `Node version must be >= 6. Currently ${major}.${minor}`;
    logger.errorAndThrow(msg);
  }
}
```
这里我们安装的是8.9.4，具体安装过程不再赘述.
安装完成后执行node -v:
>C:\Users\zhang>node -v
v8.9.4
C:\Users\zhang>

npm是node平台默认的模块管理工具，但是由于国内网络环境问题，导致npm安装模块是龟速，建议大家使用[cnpm镜像](https://npm.taobao.org/)来安装各种模块。我们先用执行npm命令看看效果：
>C:\Users\zhang>npm
>Usage: npm &lt;command&gt;

这样就说明npm已经安装了，接下来使用npm安装cnpm模块,执行命令如下：
> npm install -g cnpm --registry=https://registry.npm.taobao.org

安装完成后我们执行cnpm命令：
>C:\Users\zhang>cnpm
Usage: cnpm [option] <command>
Help: http://cnpmjs.org/help/cnpm

这样我们的node环境搭建完毕，以后可以通过cnpm代替npm命令了。

### appium-doctor ###
appium-doctor是appium提供的一个检查appium环境的模块，也托管在npm上，所以我们通过cnpm下载：
> cnpm install -g appium-doctor

这里要简单说下 -g 的作用，-g是全局安装（global）的意思 如果不添加-g则是本地安装。两者的区别是**全局安装可以直接在命令行里使用**。

安装appium-doctor完成后我们使用appium-doctor --dev来查看还差哪些appium需要的环境：
> appium-doctor --dev

![](https://i.imgur.com/Djb5p2O.png)

目前显示我的adb已经有了但是还没有加入到path中，maven与ant还没有安装，android-16与19都没有(在sdk\platforms中查看)，我这只有android-25，为了方便以后使用adb，建议将adb添加到path中

这里有几点要说：
- adb 是appium操作app的工具，必须要有的
- maven与ant是我们打jar包需要的工具，我猜测应该是打bootstrap.jar使用的
- android-16是用来兼容低版本的，
- android-19是使用uiautomator2的
- ** 如果我们不打算编译appium源码，也不打算兼容19以下的版本，我们可以不安装android-16跟android-19的版本 以及maven、ant环境 **

### 开始安装appium ###
现在我们尝试用cnpm安装appium试试：
> cnpm install -g appium

** 需要注意 **如果下载过程有中断最好先使用npm(不是用cnpm) uninstall -g appium卸载原有模块然后从新执行cnpm install -g appium .如果网络比较好几分钟就安装完成，我这大概有十分钟安装完成。
安装完成后执行appium命令，如果出现以下内容则安装成功：
>C:\Users\zhang>appium
[Appium] Welcome to Appium v1.7.2
[Appium] Appium REST http interface listener started on 0.0.0.0:4723

如果想编译appium源码并调试请移步[appium源码编译](http://www.kikijie.com/2018/03/12/appium%E6%BA%90%E7%A0%81%E7%BC%96%E8%AF%91/#more)

----

补充说明：如果出现如下错误，则不必担心，并不影响安装：

```
MSBUILD : error MSB3428: 未能加载 Visual C++ 组件“VCBuild.exe”。要解决此问题，1) 安装 .NET Framework 2.0 SDK；2) 安装 Microsoft Visual Stu
dio 2005；或 3) 如果将该组件安装到了其他位置，请将其位置添加到系统路径中。 [C:\Users\zhang\AppData\Roaming\npm\node_modules\appium\node_modules\_heapd
ump@0.3.9@heapdump\build\binding.sln]
```






