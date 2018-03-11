---
title: appium源码编译
categories:
- 自动化测试
tags:
- appium
- 源码

---
学习一门新技术，在有源码的情况下，最好能够自己编译、调试，这样才能最快的知其所以然，否则还是停留在使用的阶段，得不到质的提升。
下面来编译appium
<!-- more -->
#### 准备 ###

- [appium](https://github.com/appium/appium) 源码
- [cnpm](https://npm.taobao.org/) 环境(加速npm)
- node 环境(appium是node平台应用程序)
- java 环境(android 需要)
- android sdk
- maven 环境(编译android相关app需要)
- ant 环境(编译bootstrap需要)

>主要配置：JAVA_HOME、ANDROID_HOME、ANT_HOME、M2_HOME、adb添加到path中、android-16跟android-19。可以使用sdk中的SDK Manager下载

具体的环境配置可以网上搜索。最后可以使用appium-doctor --dev检查环境是否ok

### 检查环境 ###
设置好cnpm后可以以全局方式安装appium-doctor. 再用--dev查看环境状态
> cnpm install -g appium-doctor 
> appium-doctor --dev

![](https://i.imgur.com/VtCydyu.png)

这里显示有一项maven没配置好，其实我的maven已经加入到path中，只是没有mvn.bat文件，并不影响。

### 开始安装依赖 ###
接下来进入源码目录开始编译appium
> cnpm install

> 注意：如果安装过程中有中断，或者更新源码，或者有异常，最好删除根目录下的node_modules 再cnpm install

安装过程出现：
>MSBUILD : error MSB3428: 未能加载 Visual C++ 组件“VCBuild.exe”  不必担心，并不影响正常编译

编译完成后再通过gulp transpile构建下
最后执行 node . 启动appium
 ![](https://i.imgur.com/BzMBAQV.png)

这样我们编译appium的工作就完成了。

### webstrom调试 ###
直接用webstrom打开根目录后，再添加一个node .配置：

![](https://i.imgur.com/9CcVeuu.png)

然后我们就可以用webstrom的debug模式调试了！




