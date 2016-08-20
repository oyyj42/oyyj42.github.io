---
layout: post
title: Android端移植vlc视频音频播放器
category: android
---


--------------------------------------

久仰开源多媒体播放器vlc的大名，最近项目用到，所以移植到android玩玩，我电脑运行的是mac平台，其他平台可能略为有点出入。



##0x00:Heatup

+ 一个好网络环境
+ 2小时或以上的时间投入
+ 2.5Gb以上的存储空间 (wow)



##0x01 下载编译


下载目前最新的** vlc android ** 2.0.2版本的apk和源码：[VLC-Android-2.0.2](http://get.videolan.org/vlc-android/2.0.2/)



AndroidStudio打开后，目录是这个样子的:
![]({{site.baseurl}}/img/vlc/1.png)


三个红框分别对应：

1. Android ndk的header文件
2. libvlc在AndroidStudio的module，主要是jni部分
3. **vlc-android**是Android vlc app的界面和逻辑层，依赖和调用libvlc模块的对应api来进行播放器的各种操作。



在AndroidStudio下折腾了几下，还是报错，看了下官网资料，原来可以直接在命令行调用源码根目录下的*** conpile.sh ***，其中会下载很多第三方库和编译，需要一点时间。


调用 *** conpile.sh *** 的时候，提示需要指定
ANDROID_NDK和ANDROID_SDK两个环境变量。


```

42s-mbp:VLC-Android-2.0.2 oyyj$ ./compile.sh 
Please set the ANDROID_NDK environment variable with its path.


```

在 *** conpile.sh ***开始的地方加上自己电脑上的SDK，NDK目录，还有ABI的选项

```

export ANDROID_NDK="/Users/oyyj/code/android-ndk-r11c"
export ANDROID_SDK="/Users/oyyj/Library/Android/sdk"
export ANDROID_ABI="armeabi-v7a"
```

如无意外，应该能跑起来了，但当前目录缺/vlc文件夹，会自动下载,这里看网速了，我公司的龟速,**TAT**


```

42s-mbp:VLC-Android-2.0.2 oyyj$ ./compile.sh 
*** No ANDROID_ABI defined architecture: using ARMv7
Cloning into 'sdk-manager-plugin'...

BUILD SUCCESSFUL

Total time: 2 mins 11.249 secs

This build could be faster, please consider using the Gradle Daemon: https://docs.gradle.org/2.10/userguide/gradle_daemon.html
VLC source not found, cloning
Cloning into 'vlc'...
remote: Counting objects: 486042, done.
remote: Compressing objects: 100% (96466/96466), done.
Receiving objects:   8% (38948/486042), 8.03 MiB | 12.00 KiB/s    


```

下一个错误：*** \*\*\* xz and lzma client not found!*** 


```


Run "make" to start compilation.

Other targets:
 * make install      same as "make"
 * make prebuilt     fetch and install prebuilt binaries
 * make list         list packages
 * make fetch        fetch required source tarballs
 * make fetch-all    fetch all source tarballs
 * make distclean    clean everything and undo bootstrap
 * make mostlyclean  clean everything except source tarballs
 * make clean        clean everything
 * make package      prepare prebuilt packages
 * make help         show this text
make: Nothing to be done for `fetch'.
make: `.gettext' is up to date.
../../contrib/src/ffmpeg/rules.mak:202: *** xz and lzma client not found!.  Stop.
contribs: make failed


```

这是因为没装xz，由于我的时mac系统，直接

```

brew install xz

```


基本就ok了。






#参考


+ [vlc的官网文档链接](http://www.videolan.org/developers/vlc/doc/doxygen/html/group__libvlc.html)

+ [最简单的基于libVLC的视频播放器](http://blog.csdn.net/leixiaohua1020/article/details/42363079)



