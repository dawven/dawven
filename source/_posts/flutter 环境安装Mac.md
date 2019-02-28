---
title: flutter 环境安装Mac
date: 2019-02-23 12:54:21
tags: flutter
---
## 下载Flutter SDK包

网址：<https://flutter.io/setup-macos/>

下载之后解压，放置根目录

## 配置环境变量

由于我用on my zsh，所以在.zshrc配置

```js
path+=('/Users/yourName/flutter/bin')
export PATH
```

配置完

```
source ~/.zshrc
```

完成这部以后，就算我们flutter的安装工作完成了，但是这还不能进行开发。可以使用命令来检测一下，是否安装完成了。

```
flutter -h
```
<!-- more -->

检查一下是否安装成功,出现下面的结果，说明到目前为止，我们安装一切顺利。

![image-20190224133801847](/Users/pangkanghua/Library/Application Support/typora-user-images/image-20190224133801847.png)

##检查开发环境

到上边为止，我们安装好了Flutter，但是还不具备开发环境。开发还需要很多软件和插件的支持，那到底需要哪些插件和软件那？我们可以使用Flutter为我们提供的命令来进行检查：

```
fultter doctor
```

![image-20190224134137881](/Users/pangkanghua/Library/Application Support/typora-user-images/image-20190224134137881.png)

他会检查以上这些方面

有可能你的Android studio也没有安装，那么你要先安装这个编辑器，安装好后，可以顺便下载Android SDK。

Android Studio下载地址：<http://www.android-studio.org/>

打开后选择对应的Mac版本

![image-20190224134314790](/Users/pangkanghua/Library/Application Support/typora-user-images/image-20190224134314790.png)

如果你有安装，那么第一步要作的是允许协议（android-licenses）。允许方法就是在终端运行如下命令：

```
flutter doctor --android-licenses
```

运气好的话一路y到结尾。

但大部分都会

```
flutter doctor --android-licenses
A newer version of the Android SDK is required. To update, run:
C:\Users\Caleb\AppData\Local\Android\sdk\tools\bin\sdkmanager --update
```

```
C:\Users\Caleb\AppData\Local\Android\sdk\tools\bin\sdkmanager --update
Exception in thread "main" java.lang.NoClassDefFoundError: javax/xml/bind/annotation/XmlSchema
        at com.android.repository.api.SchemaModule$SchemaModuleVersion.<init>(SchemaModule.java:156)
        at com.android.repository.api.SchemaModule.<init>(SchemaModule.java:75)
        at com.android.sdklib.repository.AndroidSdkHandler.<clinit>(AndroidSdkHandler.java:81)
        at com.android.sdklib.tool.sdkmanager.SdkManagerCli.main(SdkManagerCli.java:73)
        at com.android.sdklib.tool.sdkmanager.SdkManagerCli.main(SdkManagerCli.java:48)
Caused by: java.lang.ClassNotFoundException: javax.xml.bind.annotation.XmlSchema
        at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:582)
        at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:190)
        at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:499)
        ... 5 more
```

关于协议的问题推荐去这个issue找解决办法

https://github.com/flutter/flutter/issues/16025

你可能需要在.zshrc里配置

PUB 镜像源

```js
export ANDROID_HOME="/Users/yourName/Library/Android/sdk" #android sdk目录
export PATH=${PATH}:${ANDROID_HOME}/tools
export PATH=${PATH}:${ANDROID_HOME}/platform-tools
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```

这个大问题解决以后，我们还需要为Android Studio安装一下Flutter插件（这个有可能你安装过，如果出现下面的提示，说明你还没有安装）

打开Android Stuido 软件，然后找到Plugin的配置，搜索Flutter插件。

![alt](http://jspang.com/static/upload/20181031/Exnsbuo_aOB3s75oTd4r6JCE.png)

出现上图，点中间的`Search in repositories`,然后点击安装。

![alt](http://jspang.com/static/upload/20181031/6-jjLcq_aEZHibMxVyy_3P78.png)

可能出现问题在于安装了，重启Android Stuido了，支持了，但重新flutter doctor检查一下后，依然显示未配置。这里展示发现就算提示未安装此插件，依然不影响开发。故不理会

## 安装AVD虚拟机

1. 现在需要一个虚拟机来运行我们的程序，可以点击 Android Studio 中的上方菜单`tool` -`AVD Manager`选项。

2. 出现新建菜单，选择`Create Virtual Device.....`, 如果你一个虚拟机也没建过，这个选项在对话框的中间（我一定跟我的图一样）。 ![](http://jspang.com/static/upload/20181105/xQuLNq5ZikkOY6qxrV0F5KAK.png)![alt](http://jspang.com/static/upload/20181105/xQuLNq5ZikkOY6qxrV0F5KAK.png)

3. 选择虚拟机类型，这个你随意选就好，我选择的是`Nexus 5x`。（如果你屏幕小，就选择一个小屏幕的虚拟机） ![](http://jspang.com/static/upload/20181105/ZwbObMwjvhcsErM5boZjyvNH.png)![alt](http://jspang.com/static/upload/20181105/ZwbObMwjvhcsErM5boZjyvNH.png)



4. 选择系统，这里尽量选择最新的，我选择了`Android 9.0`系统，选择好后，又是一个漫长的等待过程。



5. 安装好后，点击开始按钮，运行虚拟机了（第一次运行，需要安装系统，会慢一些），运行起来后，如下图。 ![](http://jspang.com/static/upload/20181105/dbln0YAd-njaHfoNp42Ya70w.png)![alt](http://jspang.com/static/upload/20181105/dbln0YAd-njaHfoNp42Ya70w.png)



## 让 Flutter 跑起来

![image-20190224140928784](/Users/pangkanghua/Library/Application Support/typora-user-images/image-20190224140928784.png)

在idea里只需要点击开始，前提想要启动虚拟机，

main.dart是整个项目的主文件

在终端的话，运行flutter run



flutter用Dart语言，之后学习和flutter组件

