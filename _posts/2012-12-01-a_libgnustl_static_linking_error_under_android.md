---
id: 683
title: android 下关于 libgnustl_static.a 的一个编译错误
date: 2012-12-01T17:52:15+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=683
permalink: /a_libgnustl_static_linking_error_under_android/
tags:
  - android
  - gnustl
  - ndk
  - boost
  - basic_string
---
用 NDK 编译一个 .so, 代码里用到了 boost::algorithm 里的一些字符串算法比如: `boost::find_head()`. 链接的 c++ 库是 libgnustl_shared.so, 或者 libgnustl_static.a. 遇到的编译错误是:
	
	undefined reference to `std::basic_string<char, std::char_traits<char>, std::allocator<char> >::basic_string<__gnu_cxx::__normal_iterator<char const*, std::basic_string<char, std::char_traits<char>, std::allocator<char> > > >(__gnu_cxx::__normal_iterator<char const*, std::basic_string<char, std::char_traits<char>, std::allocator<char> > >, __gnu_cxx::__normal_iterator<char const*, std::basic_string<char, std::char_traits<char>, std::allocator<char> > >, std::allocator<char> const&#038;)'
	
先查看 basic_string.h 的源码:

      template<class _InputIterator>
        basic_string(_InputIterator __beg, _InputIterator __end,
               const _Alloc&#038; __a = _Alloc());
               
的确有声明, 但实现应该是放到了 libgnustl_static.a 里了. 用 nm 查看一下 `libgnustl_static.a` 里的符号吧. 注意要使用 ndk toolchain 中的 nm:

	/android-ndk-r8/toolchains/arm-linux-androideabi-4.4.3/prebuilt/darwin-x86/bin/arm-linux-androideabi-nm obj/local/armeabi/libgnustl_static.a |/android-ndk-r8/toolchains/arm-linux-androideabi-4.4.3/prebuilt/darwin-x86/bin/arm-linux-androideabi-c++filt

使用 c++filt 来 demangling.下面是部分输出:

	…
	00000000 W std::basic_string<char, std::char_traits<char>, std::allocator<char> >::basic_string<__gnu_cxx::__normal_iterator<char*, std::basic_string<char, std::char_traits<char>, std::allocator<char> > > >(__gnu_cxx::__normal_iterator<char*, std::basic_string<char, std::char_traits<char>, std::allocator<char> > >, __gnu_cxx::__normal_iterator<char*, std::basic_string<char, std::char_traits<char>, std::allocator<char> > >, std::allocator<char> const&#038;)
	…

可以看到, `libgnustl_static.a` 提供了一个普通的 iterator 的版本 (`char*`). 而我们需要一个 `const_iterator` 的版本(`char const*`).

折腾很久 (大概两三个小时), 无意间在 `basic_string.h` 所在目录下发现了一个 `basic_string.tcc` 文件. 进去一看, 就有我们需要的
 `basic_string(_InputIterator __beg, _InputIterator __end, const _Alloc __a)` 的实现. 在头文件中搜索了一下这个 `basic_string.tcc` 有没有被包含. 发现在 string 头文件中有如下代码:
 
	#ifndef _GLIBCXX_EXPORT_TEMPLATE
	//# include <bits/basic_string.tcc>
	#endif

本来需要包含这个头文件的, 不知为何却被注释掉了. 查看了其他几个版本的 NDK, 这里都没有被注释掉. 蹊跷. 先不管那么多了. 直接在自己的头文件中包含这个 `basic_string.tcc` 文件吧. 编译, 问题解决了.

之前还遇到一个奇怪的情况就是, 如果把它编译成可执行文件, 就没有编译错误. 后来发现是因为这个代码在可执行文件中没有被调到. 所以编译器直接把它给优化掉了. 所以没有错误. 而编译成 .so 时, 会导出所有的符号. 所以出现了这个链接错误.
