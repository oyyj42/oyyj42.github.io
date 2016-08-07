---
layout: post
title: Android移植FFmpeg
category: android
---




------



编译前准备：


Linux系统：ubuntu 14.10 LTS 

NDK版本:  android-ndk-r9 

AndroidStudio 1.3以上


[ffmpeg源码](http://ffmpeg.org)



------

#### 1. Android完整版本交叉编译

[参考1](http://blog.csdn.net/gobitan/article/details/22750719)：


> 如果直接按照未修改的配置进行编译，结果编译出来的so文件类似libavcodec.so.55.39.101，版本号位于so之后，Android上似乎无法加载。



参考连接1，修改ffmpeg-2.8.1/configure文件：

```

SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'

LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'

SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'

SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR)$(SLIBNAME)'

```

替换为：

```

SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'

LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'

SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'

SLIB_INSTALL_LINKS='$(SLIBNAME)'


```

创建自动编译脚本

```

vim build_android.sh

```
输入以下内容：


```
#!/bin/bash

export  NDK=/home/xmission/oyyj/android-ndk-r9

export SYSROOT=$NDK/platforms/android-9/arch-arm/

export TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.8/prebuilt/linux-x86_64

export  CPU=arm

export PREFIX=$(pwd)/android/$CPU

export  ADDI_CFLAGS="-marm"

function build_one
{

./configure \

--prefix=$PREFIX \

--enable-shared \

--disable-static \

--disable-doc \

--disable-ffserver \

--enable-cross-compile \

--enable-runtime-cpudetect \

--disable-small \

--disable-ffprobe --disable-ffplay --enable-ffmpeg --disable-ffserver --disable-debug \

--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \

--target-os=linux \

--arch=arm \

--enable-gpl \

--sysroot=$SYSROOT \

--extra-cflags="-DANDROID -mfloat-abi=softfp -marm -march=armv7-a" \

--extra-ldflags="$ADDI_LDFLAGS" \

--extra-libs=-lgcc \

$ADDITIONAL_CONFIGURE_FLAG

}
build_one

                  
```

并增加可执行权限

```

chmod +x build_android.sh

./build_androud.sh

```


成功执行后，在*** ./android/arm/lib ***目录下就能找到需要的各种 *.so库


```
-rwxr-xr-x 1 root root 9088904 Dec  3  2015 libavcodec-56.so

-rwxr-xr-x 1 root root   51188 Dec  3  2015 libavdevice-56.so

-rwxr-xr-x 1 root root 1136584 Dec  3  2015 libavfilter-5.so

-rwxr-xr-x 1 root root 1733260 Dec  3  2015 libavformat-56.so

-rwxr-xr-x 1 root root  353712 Dec  3  2015 libavutil-54.so

-rwxr-xr-x 1 root root   38200 Dec  3  2015 libpostproc-53.so

-rwxr-xr-x 1 root root   71068 Dec  3  2015 libswresample-1.so

-rwxr-xr-x 1 root root  349596 Dec  3  2015 libswscale-3.so


```

成了，可以拿各种so库来用了。

#### 2.彩蛋
在1.2的基础上，要开启neon的话，需要在build_android.sh脚本里增加以下内容：

```

--cpu=armv7-a \

--enable-neon \

--enable-asm \

--enable-gpl \

--sysroot=$SYSROOT \

--extra-cflags="-fPIC -DANDROID -mfpu=neon -mfloat-abi=softfp -marm -march=armv7-a -I$NDK/platforms/android9/arch-arm/usr/include " \

```

目前最大的不足是so库太大了，光这个库***libavcodec-56.so ***就9Mb了

后续可以慢慢参考官方文档，禁用其他用不到的功能，逐步削减大小。


### 3.app例子





在Eclipse或者AndrodStudio里创建工程，（这里用的是*** AndroidStudio***），添加jni文件夹，把上面的各个.so文件和/arm/include/目录下的头文件拷贝到jni目录下，编写一个java类***ffcodec_controller.java***，粘合层*** ffcodec_controller.***,再编写Application.mk和Android.mk


![jni目录]({{site.baseurl}}/img/ffmpeg_jni_dir_edit.png)




FFCodecController.java


```

package com.oyyj.media.ffmpeg;

import android.util.Log;

public class FFCodecController {
	private static final String TAG =FFCodecController.class.getSimpleName();

	public FFCodecController() {

		for (int i = 0; i < FFMPEG_LIBS.length; i++) {
			try {				
				System.loadLibrary(FFMPEG_LIBS[i]);
			} catch (Exception e) {
					Log.e(TAG, "load:"+FFMPEG_LIBS[i]+"error!");
					e.printStackTrace();
			}
		}
	}
	
	private static final String[] FFMPEG_LIBS = { 
			"avcodec-56", 
			"avdevice-56", 
			"avfilter-5", 
			"avformat-56",
			"avutil-54",
			"postproc-53",
			"swresample-1", 
			"swscale-3", 
			"ffcodec_controller"
	};

	
	/* new api*/
	public native int ffcodec_init_h264_decoder(int width ,int height);
	public native int ffcodec_decode_h264_frame(byte[] frame,int length,byte[] out);
	public native int ffcodec_deinit_h264_decoder();

	//测试用
	public native int ffcodec_decodefile_h264_to_yuv(String h264file,String yuvFile);
}





```


-------------


Android.mk
 
```
 
LOCAL_PATH := $(call my-dir)  
  

include $(CLEAR_VARS)  
LOCAL_MODULE := avcodec  
LOCAL_SRC_FILES := libavcodec-56.so  
include $(PREBUILT_SHARED_LIBRARY)  
  
include $(CLEAR_VARS)  
LOCAL_MODULE := avdevice  
LOCAL_SRC_FILES := libavdevice-56.so  
include $(PREBUILT_SHARED_LIBRARY)  
  
include $(CLEAR_VARS)  
LOCAL_MODULE := avfilter  
LOCAL_SRC_FILES := libavfilter-5.so  
include $(PREBUILT_SHARED_LIBRARY)  
  
include $(CLEAR_VARS)  
LOCAL_MODULE := avformat  
LOCAL_SRC_FILES := libavformat-56.so  
include $(PREBUILT_SHARED_LIBRARY)  
  
include $(CLEAR_VARS)  
LOCAL_MODULE := avutil  
LOCAL_SRC_FILES := libavutil-54.so  
include $(PREBUILT_SHARED_LIBRARY)  
  
include $(CLEAR_VARS)  
LOCAL_MODULE := postproc  
LOCAL_SRC_FILES := libpostproc-53.so  
include $(PREBUILT_SHARED_LIBRARY)  
  
include $(CLEAR_VARS)  
LOCAL_MODULE := swresample  
LOCAL_SRC_FILES := libswresample-1.so  
include $(PREBUILT_SHARED_LIBRARY)  
  
include $(CLEAR_VARS)  
LOCAL_MODULE := swscale  
LOCAL_SRC_FILES := libswscale-3.so  
include $(PREBUILT_SHARED_LIBRARY)  
  
 
include $(CLEAR_VARS)  
LOCAL_MODULE := ffcodec_controller  
LOCAL_SRC_FILES := ffcodec_controller.c  
LOCAL_C_INCLUDES += $(LOCAL_PATH)/include  
LOCAL_CFLAGS := -D__STDC_CONSTANT_MACROS -Wno-sign-compare -Wno-switch -Wno-pointer-sign -DHAVE_NEON=1 -mfpu=neon -mfloat-abi=softfp -fPIC -DANDROID 
LOCAL_LDLIBS := -L$(NDK_PLATFORMS_ROOT)/$(TARGET_PLATFORM)/arch-arm/usr/lib -L$(LOCAL_PATH) -llog -ljnigraphics -lz -ldl 
LOCAL_SHARED_LIBRARIES := avcodec avdevice avfilter avformat avutil postproc swresample swscale  
include $(BUILD_SHARED_LIBRARY)  
 
 
```
 
 
 
 Application.mk  (ABI总共有四种，分别是armeabi、armeabi-v7a、mips、x86)，后来最新的还有 arm64-v8a（例如heilo-X25）
 
 
```
 
 APP_ABI := armeabi-v7a
 

```
 

 
 
 切换到/src/main/jni/,执行ndk-build，so文件会输出到/libs/armeabi-v7a/
 
```

[armeabi-v7a] Prebuilt       : libpostproc-53.so <= jni/
[armeabi-v7a] Prebuilt       : libswresample-1.so <= jni/
[armeabi-v7a] Prebuilt       : libswscale-3.so <= jni/
[armeabi-v7a] SharedLibrary  : libffcodec_controller.so
[armeabi-v7a] Install        : libffcodec_controller.so => libs/armeabi-v7a/libffcodec_controller.so
[armeabi-v7a] Install        : libpostproc-53.so => libs/armeabi-v7a/libpostproc-53.so
[armeabi-v7a] Install        : libswresample-1.so => libs/armeabi-v7a/libswresample-1.so
[armeabi-v7a] Install        : libswscale-3.so => libs/armeabi-v7a/libswscale-3.so


```
 
 为了让AndroidStudio打包的时候加上头文件，记得在***build.gradle***里加上
 
 
```
 
 android
 {
 
	 
 
     sourceSets {
        main {
            jniLibs.srcDirs = ['src/main/libs']
        }
    }
 {
 
```
 
  
简单的Activity测试代码：
 
 
```

package com.oyyj.ex;

import android.app.Activity;
import android.os.Bundle;

import com.oyyj.media.ffmpeg.FFCodecController;
import com.oyyj.ex.R;


/**
 * Created by oyyj on 16/8/6.
 */
public class MainActivity extends Activity {
    private static final String TAG = "DecodeActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_start);

        new Thread(new Runnable() {
            @Override
            public void run() {


                FFCodecController controller = new FFCodecController();
                controller.ffcodec_decodefile_h264_to_yuv("/mnt/sdcard/hand.h264","/mnt/sdcard/out.yuv");

            }
        }).start();
    }
}

```

运行到手机里，即可以解码出来yuv（YUV420P格式）文件：

![]({{site.baseurl}}/img/save_yuv_edit.png)



***ffcodec_controller.c的源码***实在太长，就不贴了，详情请看
[github 项目主页](https://github.com/oyyj42/SimpleFFmpegDemo)



------------------

### 4.参考连接：



> [英年早逝，纪念永远的雷神：雷霄骅(leixiaohua1020)的专栏](http://blog.csdn.net/leixiaohua1020/article/details/15811977)

> [雨水的专栏](http://blog.csdn.net/gobitan/article/details/22750719)

