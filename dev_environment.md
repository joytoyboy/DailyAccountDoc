#  开发环境搭建

下面介绍Windows下开发环境的搭建。

1. 安装Node.js
1. 安装Cordova。Cordova用来将我们的APP包裹成原生APP。

 > npm install -g cordova

1. 安装Java JDK [下载地址](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

 安装完成后， 设置一下环境变量：

 - 添加`JAVA_HOME`，指向：jdk的安装目录，如：C:\Program Files\Java\jdk1.8.0_91。
 - 将`%JAVA_HOME%\bin`追加到Path变量中。

1. Apache Ant [下载地址](http://ftp.tsukuba.wide.ad.jp/software/apache//ant/binaries/apache-ant-1.9.7-bin.zip)

 - 下载后解压到任意位置，如C:\Program File\
 - 将C:\Program File\apache-ant-1.9.7\bin追加到环境变量的Path中。

1. 安装Android SDK

 这里建议将Android Studio一起安装。[下载地址](http://developer.android.com/sdk/index.html) [当前版本](https://dl.google.com/dl/android/studio/install/2.1.0.9/android-studio-bundle-143.2790544-windows.exe)
  安装后，设置环境变量：
  - 添加`ANDROID_HOME`，指向：[ANDROID_SDK_DIR]\android-sdk目录。
  - 将`%ANDROID_HOME%\tools`和`%ANDROID_HOME%\platform-tools`追加到环境变量的Path中。

1. 安装python-2.7.9 [下载地址](https://www.python.org/downloads/) [当前版本](https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe)

1. 安装Visual C++ Redistributable Packages for Visual Studio 2013 [下载地址](https://www.microsoft.com/zh-CN/download/details.aspx?id=40784)

1. 安装ionic

 > npm install -g ionic

1. 安装chrome(非必需)

这样，开发环境就设置完毕。
