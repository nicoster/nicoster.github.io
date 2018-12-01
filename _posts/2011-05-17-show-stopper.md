---
id: 454
title: 'Show Stopper 一次 crash 调试的夺命狂奔'
date: 2011-05-17T23:45:46+00:00
author: nick
layout: post
permalink: /show-stopper/
tags:
  - dbg
  - android 
  - x86 
  - arm 
  - addr2line 
  - logcat
  - tombstone
  - stlport 
  - _STLP_USE_NEWALLOC 
  - _NOTHREADS
---
这几天一直在忙着调试 crash 的问题。周末两天都在加班。 周日更是从早上8：00 到晚上 12：50 一直没离开过办公室. 加上这个项目对我们整个开发组以及 EM 都很重要，不容有失，这不禁让我想起了微软 NT 开发组开发 NT 的情形，所以有了这个标题.
 
这次是在 android 上，但不是 arm，而是 x86 atom。我们的程序是从 windows 上移植到 android 上的, 一个 C++ 写的底层库作为 service，UI 是 java 写的。 因为是在 android 2.2 上，java 也是唯一的写 UI 的选择。 java 通过 aidl/jni 与底层 service/c++ 代码交互。这样的架构让调试很悲剧。
 
之前一直抱怨 xcode 调试多么不给力，而现在在 android 上的调试已经原始到通过 log 分析程序的执行。arm 上的调试还能用 gdb 勉强为之，而 x86 下那简直就是坑爹。好不容易把符号神马的搞定，gdb 远程调试到设备上，却发现程序不 crash 了。
 
logcat 和 tombstone 显示的 crash 调用栈是这样的：

	06370 INF/DEBUG   ( 1670): signal 11 (SIGSEGV), fault addr deadbaad
	06371 INF/DEBUG   ( 1670):  eax 00000000  ebx 801614f4  ecx 00000004  edx 00001000
	06372 INF/DEBUG   ( 1670):  esi b3afa000  edi bfca7b18
	06373 INF/DEBUG   ( 1670):  xcs 00000073  xds 0000007b  xes 0000007b  xfs 00000000 xss 0000007b
	06374 INF/DEBUG   ( 1670):  eip 8011c768  ebp bfca7b28  esp bfca7af0  flags 00010286
	06375 INF/DEBUG   ( 1670): #00
	06376 INF/DEBUG   ( 1670):     eip: 8011c768  /system/lib/libc.so (abort)
	06377 INF/DEBUG   ( 1670): #01
	06378 INF/DEBUG   ( 1670):     eip: 8010ed4b  /system/lib/libc.so (dlfree)
	06379 INF/DEBUG   ( 1670): #02
	06380 INF/DEBUG   ( 1670):     eip: 80111503  /system/lib/libc.so (free)
	06381 INF/DEBUG   ( 1670): #03
	06382 INF/DEBUG   ( 1670):     eip: 80200a1d  /system/lib/libstdc++.so (_ZdlPv)
	06383 INF/DEBUG   ( 1670): #04
	06384 INF/DEBUG   ( 1670):     eip: 86138519  /data/data/cip.impservice/lib/libimpjni2.so (_ZSt12__stl_deletePv)
	06385 INF/DEBUG   ( 1670): #05
	06386 INF/DEBUG   ( 1670):     eip: 8613853d  /data/data/cip.impservice/lib/libimpjni2.so (_ZNSt11__new_alloc10deallocateEPvj)
	06387 INF/DEBUG   ( 1670): #06
	06388 INF/DEBUG   ( 1670):     eip: 8613856e  /data/data/cip.impservice/lib/libimpjni2.so (_ZNSaIcE10deallocateEPcj)
	06389 INF/DEBUG   ( 1670): #07
	06390 INF/DEBUG   ( 1670):     eip: 861385b1  /data/data/cip.impservice/lib/libimpjni2.so (_ZNSt4priv17_STLP_alloc_proxyIPccSaIcEE10deallocateES1_j)
	06391 INF/DEBUG   ( 1670): stack:
	06392 INF/DEBUG   ( 1670): #00
	06393 INF/DEBUG   ( 1670):     bfca7af0  00000002   (_ZNSt4priv17_STLP_alloc_proxyIPccSaIcEE10deallocateES1_j)
	06394 INF/DEBUG   ( 1670):     bfca7af4  bfca7b18  [stack] (_ZNSt4priv17_STLP_alloc_proxyIPccSaIcEE10deallocateES1_j)
	06395 INF/DEBUG   ( 1670):     bfca7af8  00000000   (_ZNSt4priv17_STLP_alloc_proxyIPccSaIcEE10deallocateES1_j)
	06396 INF/DEBUG   ( 1670):     bfca7afc  00000000   (_ZNSt4priv17_STLP_alloc_proxyIPccSaIcEE10deallocateES1_j)
	06397 INF/DEBUG   ( 1670):     bfca7b00  00000000   (_ZNSt4priv17_STLP_alloc_proxyIPccSaIcEE10deallocateES1_j)
	06398 INF/DEBUG   ( 1670):     bfca7b04  00000000   (_ZNSt4priv17_STLP_alloc_proxyIPccSaIcEE10deallocateES1_j)
	06399 INF/DEBUG   ( 1670):     bfca7b08  84a60900  /system/lib/libstlport.so (_ZNSt4priv17_STLP_alloc_proxyIPccSaIcEE10deallocateES1_j)
	06400 INF/DEBUG   ( 1670):     bfca7b0c  00000010   (_ZNSt4priv17_STLP_alloc_proxyIPccSaIcEE10deallocateES1_j)
	06401 INF/DEBUG   ( 1670):     bfca7b10  0a1f8300  [heap] (_ZNSt4priv17_STLP_alloc_proxyIPccSaIcEE10deallocateES1_j)
	06402 INF/DEBUG   ( 1670):     bfca7b14  00000001   (_ZNSt4priv17_STLP_alloc_proxyIPccSaIcEE10deallocateES1_j)
	06403 INF/DEBUG   ( 1670):     bfca7b18  fffffbdf   (_ZNSt4priv17_STLP_alloc_proxyIPccSaIcEE10deallocateES1_j)
	06404 INF/DEBUG   ( 1670):     bfca7b1c  801614f4  /system/lib/libc.so (_ZNSt4priv17_STLP_alloc_proxyIPccSaIcEE10deallocateES1_j)
	06405 INF/DEBUG   ( 1670):     bfca7b20  0a1f6c98  [heap] (_ZNSt4priv17_STLP_alloc_proxyIPccSaIcEE10deallocateES1_j)
	06406 INF/DEBUG   ( 1670):     bfca7b24  bfca8410  [stack] (_ZNSt4priv17_STLP_alloc_proxyIPccSaIcEE10deallocateES1_j)
	06407 INF/DEBUG   ( 1670):     bfca7b28  bfca7b68  [stack] (_ZNSt4priv17_STLP_alloc_proxyIPccSaIcEE10deallocateES1_j)
	06408 INF/DEBUG   ( 1670): #01
	06409 INF/DEBUG   ( 1670):     bfca7b2c  8010ed4b  /system/lib/libc.so (dlfree)
	06410 INF/DEBUG   ( 1670):     bfca7b30  80163118   (dlfree)
	06411 INF/DEBUG   ( 1670):     bfca7b34  00000000   (dlfree)
	06412 INF/DEBUG   ( 1670):     bfca7b38  8010f3ab  /system/lib/libc.so (dlmalloc)
	06413 INF/DEBUG   ( 1670):     bfca7b3c  801614f4  /system/lib/libc.so (dlmalloc)
	06414 INF/DEBUG   ( 1670):     bfca7b40  00000066   (dlmalloc)
	06415 INF/DEBUG   ( 1670):     bfca7b44  bfca8410  [stack] (dlmalloc)
	06416 INF/DEBUG   ( 1670):     bfca7b48  0a1f6c90  [heap] (dlmalloc)
	06417 INF/DEBUG   ( 1670):     bfca7b4c  801114d2  /system/lib/libc.so (malloc)
	06418 INF/DEBUG   ( 1670):     bfca7b50  0000002f   (malloc)
	06419 INF/DEBUG   ( 1670):     bfca7b54  80201d98  /system/lib/libstdc++.so
	06420 INF/DEBUG   ( 1670):     bfca7b58  bfca7b68  [stack]
	06421 INF/DEBUG   ( 1670):     bfca7b5c  801614f4  /system/lib/libc.so
	06422 INF/DEBUG   ( 1670):     bfca7b60  0a1f6caf  [heap]
	06423 INF/DEBUG   ( 1670):     bfca7b64  bfca8410  [stack]
	06424 INF/DEBUG   ( 1670):     bfca7b68  bfca7b78  [stack]
	06425 INF/DEBUG   ( 1670):     ......  ......
	 
每次都几乎看不到我们的代码，好不容易看到一个用户代码的地址，通过 addr2line, 也是 STL 里的 _alloc.h 之类。这其实是一个提示，但一开始我们并没有在意。我们还是坚忍的打着 log，试图逐一的排查 crash， 好不容易向前推进了一些让程序跑得更远一点，再跑一次却发现又回到了起点， crash 又在前面的代码中出现鸟，我勒个去～
 
不稳定的 crash 让我有些沮丧，同时也让我产生了怀疑，这到底是不是我们的代码的问题？ 看看每次 crash 的调用栈，都有 STL 牵涉其中，又联想起我们用的 stlport 不是线程安全的 (编译时使用了 `_NOTHREADS` 宏）,  是不是应该去掉 `_NOTHREADS` 编译试试？ 但遇到的新问题是，我们使用的动态链接的系统自带的 STL，虽然可以替换掉系统的，但势必影响其他的程序。这是不能接受的。 马上想到我们可以编译一个静态库，连接到我们的程序中。说干就干，拿到 stlport 的代码，折腾一番，也就编译好了。这里还有一个小插曲， android 的头文件说它对 `#include_next` 支持的并不好，但最终我们还是用了 gcc 的这个 feature 才让代码编译通过。
 
但测试的结果却让人沮丧，我再次确认了的确是链接到了自己编译的静态版本，悲催的发现这次尝试可耻的失败鸟～ 这时已是周一凌晨 0:50 了，而我已经在办公室呆了 16个小时没有离开。
 
周一早上9:40到达公司。一计不成，又生一计，这次我注意到调用栈顶端的 lib 的 free 函数，既然每次都 crash 在这里，那我能不能让它不调用这个函数。联想起之前看过的 [stlport 文档](http://www.stlport.com/doc/configure.html)，的确有一个宏  `_STLP_USE_NEWALLOC` 可以让 stlport 使用 new/delete 而不是 malloc/free 来分配/释放内存, 无独有偶，目前我们做的这个项目的前身（另一个项目组开发的）编译的时候就加了 stlport 的这个宏（我们在移植代码的时候由于匆忙，急于让代码通过编译，没能加上这个宏），这更坚定了我做这个尝试的决心。于是给十几个模块的 mk 文件都加上了这几个 stlport 相关的宏(看 stlport 文档关于分配内存的宏），rebuild all。 哈利露亚～ 幸福来得那么突然， 程序真的跑起来了，不再 crash！
 
总结这次调试的经验教训:

1. 开始时调试进展缓慢，问题出在编译/调试环境很不给力。
我们需要把代码放到另一个 site 的唯一的 build machine 上让同事帮忙编译，这个过程让调试过程变得很漫长。 所以我让同事给我在 build machine 上单独弄了一个环境，这样我就可以随意的修改/调试代码了
2. 对问题不敏感，虽然每次都看到 stl/free 出现在调用栈中，直到在错误的方向上迷失才意识到我们可能走错路了
3. 对项目本身不够了解，我没能意识到我们其实有一个能用的版本（虽然我们重构了很多），的确应该早些查看别人是怎么做到的。 这里有一个客观原因是我之前一直不在这个项目。
4. Android 的调试环境太坑爹了。既然符号神马的都齐活了，在调研栈里却只显示地址和莫名其妙的函数名（还是 decorated), 尼玛就不能把代码行也加进去啊！有木有！Android 开发小组在忙于追赶/超越苹果的时候，明显对开发者照顾不够。 当然，也有可能是我被微软给惯坏了.
 
 
