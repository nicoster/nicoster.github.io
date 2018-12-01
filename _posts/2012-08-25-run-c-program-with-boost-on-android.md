---
id: 621
title: Run c++ program with boost on Android
date: 2012-08-25T23:24:08+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=621
permalink: /run-c-program-with-boost-on-android/
tags:
  - android
  - application.mk
  - boost
  - ndk
  - android.mk
  - ANDROID_SDKROOT
  - jni
---
Android 很早就支持 NDK, 官方教程上说你可以把 native 代码编译成一个 .so, 在 java 代码中通过 jni 调用这个 .so. 

现在在做一个 C++ 跨平台的库. 需要支持 Android. 为了在 Android 上测试, 自然可以按照教程里写 java, jni. 但其实也可以抛开 java. 直接将 c++ 代码编译成可执行文件. 下面将整个过程记录下来, 备忘, 也希望能帮助用这样的需求的人.

先下载 sdk, ndk. 解压缩就好, 然后设置好环境变量 `ANDROID_SDKROOT`, NDKROOT 指向这两个目录. 并将 $NDKROOT, `$ANDROID_SDKROOT/platform-tools`, `$ANDROID_SDKROOT/tools` 加入 PATH. 下面的操作使用的环境是 NDK r8, OS X 10.8.
开始啦:

	#include <stdio.h>
	int main(){printf("hello world.\n");return 0;}

把这个文件保存为 /tmp/hello.cpp. 要编译这个程序, 需要写一个 Android.mk

	LOCAL_PATH := $(call my-dir)
	include $(CLEAR_VARS)   # mk 文件前面两句都这样, 不解释
	LOCAL_MODULE    := hello-world  # 模块名, 必须唯一
	LOCAL_SRC_FILES := ../hello.cpp # 要编译的源文件
	include $(BUILD_EXECUTABLE) # 编译成可执行文件. 这个是关键.
	# 其他的选项是 BUILD_SHARED_LIBRARY, BUILD_STATIC_LIBRARY 编译成 .so, .a

貌似还只能把这个 .mk 文件放到一个 jni 的文件夹里. 然后在 /tmp 目录执行 ndk-build. 如果环境都设置好了, 这个能够编译成功. 放到模拟器上试试吧.
执行 `android - avd`
打开 Android Virtual Device Manager (注意: avd 之间有一个空格), 创建一个虚拟机, 然后启动它. 确保一切 ok. 然后执行 

	adb push libs/armeabi/hello-world /data/ &#038;&#038; adb shell /data/hello-world
	1372 KB/s (299852 bytes in 0.213s)
	hello world.

push 到模拟器, 然后执行 hello-world.
继续. 修改一下代码, 试试 c++ 的库文件.
	
	#include <iostream>
	int main(){std::cout<<"hello world"; return 0;}

用 ndk-build 编译会发现有错误. 因为没有 stl 的支持.
	
	nickx:/tmp $ ndk-build
	Compile++ thumb  : hello-world <= hello.cpp
	jni/../hello.cpp:1:20: error: iostream: No such file or directory
	jni/../hello.cpp: In function 'int main()':
	jni/../hello.cpp:4: error: 'cout' is not a member of 'std'
	make: *** [obj/local/armeabi/objs/hello-world/__/hello.o] Error 1
	
在 /tmp/jni 下建立 Application.mk, 加上下面一句:

	APP_STL      := gnustl_static # 另一个选项是 gnustl_shared

链接 gnustl. 再编译就 ok 了.
因为项目要链接 boost, 加上 boost header 试试吧.

	#include <iostream>
	#include <boost/thread.hpp>
	void proc(){std::cout << "hello world\n";}
	int main()
	{
		boost::thread thrd(proc);
		thrd.join();
		return 0;
	}
	
首先是找不到 boost 头文件. 这里要做一些准备工作. 将需要编译的 boost 库在 android 编译好. 这里就不赘述了. 得到 `libboost_thread.a`, `libboost_system.a`, 将这些文件这样组织

	/vendor
	  /boost
	    Android.mk
	    /boost
	      thread.hpp
	      ..
	    /lib
	      libboost_thread.a
	      libboost_system.a

其中的 Android.mk 的内容是这样的:
	
	LOCAL_PATH:= $(call my-dir)
	include $(CLEAR_VARS)
	LOCAL_MODULE:= boost_thread
	LOCAL_SRC_FILES:= lib/android/lib$(LOCAL_MODULE).a
	LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)  # 导出头文件, 因为有这句, 在我们的程序中的 Android.mk 文件中就不需要 LOCAL_C_INCLUDES 来指定 boost 头文件所在位置了.
	include $(PREBUILT_STATIC_LIBRARY)  # 所谓的编译好的静态库
	include $(CLEAR_VARS)
	LOCAL_MODULE:= boost_system
	LOCAL_SRC_FILES:= lib/android/lib$(LOCAL_MODULE).a
	LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)

boost 库就准备好了. 现在修改程序的 Android.mk 文件:
	
	LOCAL_PATH := $(call my-dir)
	include $(CLEAR_VARS)
	LOCAL_MODULE    := hello-world
	LOCAL_SRC_FILES := ../hello.cpp
	LOCAL_CPP_FEATURES := exceptions rtti  # boost thread 库需要 exceptions 的支持. rtti 后面也会用到
	LOCAL_STATIC_LIBRARIES := boost_thread boost_system  # 要的就是这两个库.
	include $(BUILD_EXECUTABLE)
	$(call import-module,boost)  # 导入 boost 库.

再次编译会提示 import-module 找不到  boost, 需要定义 NDK_MODULE_PATH
	
	nickx:/tmp $ export NDK_MODULE_PATH=/vendor
	nickx:/tmp $ ndk-build
	Compile++ thumb  : hello-world <= hello.cpp
	Prebuilt       : libboost_thread.a <= /vendor/boost/lib/android/
	Prebuilt       : libboost_system.a <= /vendor/boost/lib/android/
	Executable     : hello-world
	Install        : hello-world => libs/armeabi/hello-world

到这里差不多了.
文中提到的 .mk 文件中的宏 (LOCAL_..) 的具体含义参见 `$NDKROOT/documentation.html`
