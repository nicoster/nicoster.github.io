---
id: 655
title: rpath, loader_path, executable_path
date: 2012-09-18T14:23:35+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=655
permalink: /rpath-loader_path-executable_path/
tags:
  - build
  - rpath
  - loader_path
  - executable_path
  - OSX
  - INSTALL_PATH
  - otool
  - install_name_tool
  - dyld
---
在 OSX 上初次接触到这些变量, 有些晕. 看了一些文档之后, 觉得弄明白了. 做一个总结.

在编译一个动态库比如 libfoo.dylib 的时候, 你需要指定 INSTALL_PATH. 也就是它的安装路径.

一个可执行程序比如 bar.app 使用 libfoo.dylib, 那么在编译 bar.app 的时候, libfoo.dylib 的 `INSTALL_PATH` 会被记录到 bar.app 中, 用来定位这个 dylib. 用如下命令可以查看:

	$ otool -L bar.app/Contents/MacOS/bar
	bar.app/Contents/MacOS/bar:
	     /usr/local/lib/libfoo.dylib (compatibility version 1.0.0, current version 1.0.0)
	     /System/Library/Frameworks/Cocoa.framework/Versions/A/Cocoa (compatibility version 1.0.0, current version 19.0.0)
	     /System/Library/Frameworks/Foundation.framework/Versions/C/Foundation (compatibility version 300.0.0, current version 945.0.0)
	     …
     
这里的 /usr/local/lib 就是默认的 `INSTALL_PATH`. 要 bar.app 能正常运行, 必须先把 libfoo.dylib 拷贝到这个目录. 如果 libfoo.dylib 只是被 bar.app 使用, 那么拷贝到系统目录可能是不合适的. 一个解决问题的办法就是修改 libfoo.dylib 的 INSTALL_PATH, 使用相对路径. 因而就需要用到下面这三个变量.
看 dyld 的 manual, 有这三个变量的解释. 

* @executable_path 这个变量表示可执行程序所在的目录. 比如 /path/bar.app/Contents/MacOS/ .
* @loader_path 这个变量表示每一个被加载的 binary (包括可执行程序, dylib, framework 等) 所在的目录. 在一个进程中, 对于每一个模块, @loader_path 会解析成不用的路径, 而 @executable_path 总是被解析为同一个路径(可执行程序所在目录). 比如一个会被多个程序调用的 plugin, 位于 /path/Myfilter.plugin/Contents/MacOS/Myfilter, 依赖 /path/Myfilter.plugin/Contents/dylib/libfoo.dylib. 那么 libfoo.dylib 的 INSTALL_PATH 可以设置为 @loader_path/../dylib, 这样设置的话, 不论 Myfilter.plugin 目录放到什么位置, libfoo.dylib 都能正确的被加载.
	* @rpath 和前面两个不同, 它只是一个保存着一个或多个路径的变量. 比如 libfoo.dylib 被两个 .app 使用, 且被包含的路径不同, 如下:
	
	bar.app/
	  Contents/
	       MacOS/
	            bar
	       libfoo.dylib
	
	baz.app
	  Contents/
	       MacOS/
	            baz
	       dylibs/
	            libfoo.dylib


将 libfoo.dylib 的 `INSTALL_PATH` 设置成 `@loader_path/..` 或 `@loader_path/../dylibs` 都只能满足其中一个 .app 的需求. 要解决这个问题, 就可以用 @rpath. 将 libfoo.dylib 的 `INSTALL_PATH` 设置成 @rpath, 然后在编译 bar.app, baz.app 时分别指定 @rpath 为 `@loader_path/..`, `@loader_path/../dylibs`, 问题得到了解决. @rpath 的另一个优点是可以设置多个路径. 如果 bar.app 还需要使用另一个 .framework (假设它的 `INSTALL_PATH` 也设置成了 @rpath), 位于 `@loader_path/../frameworks`, 把这个路径加到 @rpath 即可.
道理说清楚了, 怎么设置这些变量呢?

在 libfoo.dylib 的项目属性中设置 INSTALL_PATH 设置为 @rpath
![Screen-Shot-2012-09-18-at-12.31.22-PM.png]({{site.url}}/attachments/2012/09/Screen-Shot-2012-09-18-at-12.31.22-PM.png)

在 bar.app 中设置 @rpath 为 @loader_path/.. @loader_path/dylibs
![Screen-Shot-2012-09-18-at-12.29.55-PM.png]({{site.url}}/attachments/2012/09/Screen-Shot-2012-09-18-at-12.29.55-PM.png)

对于一个编译好的 binary, 有几个命令行工具可以查看, 修改 rpath:
查看 @rpath

	$ otool -l bar.app/Contents/MacOS/bar 
	...
	Load command 19
	          cmd LC_RPATH
	      cmdsize 32
	         path @loader_path/.. (offset 12)
	Load command 20
	          cmd LC_RPATH
	      cmdsize 40
	         path @loader_path/../dylibs (offset 12)
	…


`install_name_tool` 还可以删除/替换/添加 @rpath, 比如:


	$ install_name_tool -add_rpath @loader_path/../frameworks bar.app/Contents/MacOS/bar 


它还可以修改依赖的动态库的加载路径, 比如:


	$ otool -L bar.app/Contents/MacOS/bar
	bar.app/Contents/MacOS/bar:
	     @rpath/libfoo.dylib (compatibility version 1.0.0, current version 1.0.0)
	…
	$ install_name_tool -change @rpath/libfoo.dylib @loader_path/libfoo.dylib bar.app/Contents/MacOS/bar
	$ otool -L bar.app/Contents/MacOS/bar
	bar.app/Contents/MacOS/bar:
	     @loader_path/libfoo.dylib (compatibility version 1.0.0, current version 1.0.0)
	...


看看 otool, `install_name_tool` 的帮助了解这几个命令的细节
