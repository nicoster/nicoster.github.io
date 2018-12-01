---
id: 690
title: 在 lldb 中搜索字符串
date: 2013-08-12T22:19:52+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=690
permalink: /search_cstring_in_lldb/
tags:
  - dbg
  - ios
  - osx
  - cstr_refs
  - DYLB_INSERT_LIBRARIES
  - find_cstring_in_heap
  - heap_find
  - introspect
  - ios
  - lldb
  - malloc
---
最近在调试一个 iOS app 的时候需要在内存中查找一个特定的字符串.
搜索了一下调试器 (lldb/gdb) 的文档. 貌似在 osx/ios 上这两个调试器都没有 search memory 的命令!? 然后就开始了漫长的寻找. 整个过程颇为周折. 记下来备忘, 也希望能帮助到别人.

因为 Apple 在逐渐废弃 gdb, 转到 lldb. 所以下面涉及的都是 lldb. 也许使用 gdb 有更好的解决办法. 笔者就没有去研究了.

lldb 虽然没有 search 命令, 但最基本的 内存读写 命令还是有的. 简单的说, 如果可以写一个脚本读出内存然后比对目标字符串就可以实现我的目标了. 
lldb 支持 python 脚本. 其 Python API Reference[1] 的 SBProcess 类有一个函数:

     ReadMemory(self, *args)
     
它可以读取进程中的内存. 搜索还发现在 lldb 的源代码中有一个 memory.py 位于 lldb/examples/python, 就使用了这个函数, 实现的功能是在一个指定的区域搜索内存. 看来可以用的上. 现在的问题是怎么确定搜索区域. 考虑到其实我只需要找 heap 上搜索字符串就行了, 如果我可以枚举进程内所有的 heap, 在这些区域搜索就好了.

在搜索的过程找无意中发现 mac 下有一个命令行工具 /usr/bin/heap. 下面是它执行时的部分输出:
	
	$ heap TextEdit
	Process 89327: 5 zones
	Zone DefaultMallocZone_0x104b1a000: Overall size: 4090KB; 26413 nodes malloced for 1488KB (36% of capacity); largest unused: [0x6100002c04d0-126KB]
	Zone GFXMallocZone_0x104b26000: Overall size: 0KB
	Zone MallocHelperZone_0x104aea000: Overall size: 36868KB; 1790 nodes malloced for 1995KB (5% of capacity); largest unused: [0x7fe47203e000-7943KB]
	Zone QuartzCore_0x7fe471022600: Overall size: 48KB; 267 nodes malloced for 13KB (27% of capacity); largest unused: [0x104c48000-8KB]
	Zone DefaultPurgeableMallocZone_0x10657d000: Overall size: 0KB
	All zones: 28470 nodes malloced : 3495KB

可以看到, 它列出了 TextEdit 进程的所有的 zone 信息. 这里的 zone 就是 Apple 上对 heap 的叫法.
继续搜索, 在 stackoverflow 上发现了一篇帖子[2], 了解到 heap 这个命令行程序源码没有公开. 但它的实现使用了 malloc zone introspection APIs. 顺藤摸瓜, Mac OS X Internals [3] 对 malloc API 有一些描述. 还给出例子代码描述如何获取本进程的 heap 信息.

现在的问题是, 如何用 python 中调用这些 API. 用 python 调用简单的函数很容易. 原理其实和在 lldb 提示符下使用 expression 命令调用是一样的. 但复杂的函数就有问题了. 比如 introspect 里的 enumator(), 它需要一个回调函数. 如何用 python 写一个回调能让 c 来调用呢? 看起来这个问题比较麻烦.

继续的搜索发现这个问题其实已经有解决方案了. 之前没有仔细看 lldb 的文档. 其实 lldb 已有的脚本里有一个 heap.py 位于 `lldb/examples/darwin/heap/`
这个脚本通过 lldb 解析 c++ 表达式, 运用了匿名函数等技巧解决了这个问题. 在 Mac 上试了一下. 效果很好. 但问题是文档里说这个脚本只支持 OS X. 在 iOS 试了一下, 虽然脚本 load 成功. 但一执行 `cstr_refs`, lldb 连带 Xcode 就 crash 了. 无奈, 在 lldb-dev 的邮件列表问了一下这个脚本有没有可能支持 iOS. 一个大牛的回答[4]让人摸不着头脑. 大概是说 MCJIT 对 iOS 的支持还有点问题. 而且一时半会是支持不了. 咱水平太差, 不知道该怎么回复. 先自己摸索吧.

在 svn log 中发现早期版本的 heap.py 需要一个 c++ 写的 dylib 配合使用, 后来才把 c++ 代码修改之后挪到了 py 当中. JIT 就是用来编译 python 中的 c++ 代码的. 既然 JIT 有问题. 我们自己编译 dylib 总可以吧. 说干就干. 从 lldb 源码中 update 到有 dylib 的版本. 编译加载之.
 
	nickx:/workspace/lldb $ lldb -n TextEdit
	Attaching to process with:
	    process attach -n 'TextEdit'
	Process 89327 stopped
	Executable module set to '/Applications/TextEdit.app/Contents/MacOS/TextEdit'.
	Architecture set to: x86_64-apple-macosx.
	(lldb) process load /workspace/lldb/examples/darwin/heap_find/heap/heap_find.dylib
	error: failed to load &#8216;/workspace/lldb/examples/darwin/heap_find/heap/heap_find.dylib': unable to load &#8216;/workspace/lldb/examples/darwin/heap_find/heap/heap_find.dylib’
	
失败! lldb 也没说为什么无法加载.
既然调试器不行, 我能不能加上main() 函数, 把 heap_find.cpp 编译成可执行文件看看呢? 也能测试一下 malloc introspect 到底是不是好用. 添加的代码很简单, 一个函数用于输出结果和 main():
	
	int main(int argc, char** argv)
	{
	    char* buf = new char[100];
	    printf("buf:%p\n", buf);
	    memcpy(buf, "hello", 6);
	    *(void**)(buf + 0x20) = buf;
	    find_cstring_in_heap("hello", 0);
	    find_pointer_in_heap(buf, 0);
	    return 0;
	}

执行结果如下:

	nickx:/workspace/lldb/examples/darwin/heap_find/heap $ ./a.out
	buf:0x7febd8500000
	zone:0x103fa0000
	     count:1
	     count:256
	     count:56
	     count:6
	match, addr:0x7febd8500000, size:0&#215;70, offset:0&#215;0, type:1
	zone:0x103fa0000
	     count:1
	     count:256
	     count:56
	     count:6
	match, addr:0x7febd8500000, size:0&#215;70, offset:0&#215;20, type:1 
	
iOS 上的的结果类似. 除了是 32位的. 现在面前貌似有两条路.

1. 让这个程序支持查询另一个进程的 heap
   Mac OS X Internals [3] 提到可以对另一个进程调用 malloc_get_all_zones() 从而获取所有的 zone 信息. 也许可以试试.
2. 编译成 dylib, 想办法将它注入到目标进程. 然后从调试器中调用其中的函数 find_cstring_in_heap().


先试第一种. 通过 task_for_pid() 获取目标进程的 task. (在 iOS 上需要在 entitlement 文件中注明需要使用 task_for_pid. 用 ldid 工具 codesign 才能使用). 测试发现的确可以取到所有的 zone. 但接下来调用 introspect 的 enumator() 函数就 crash 了. 想想也是. 毕竟这里的 zone 地址是通过 vm_read() 拿到的另一个进程中的地址. 想要执行另一个进程中的代码, 哪有那么简单. 命令行工具 heap 可能也是自己实现了类似 szone_ptr_in_use_enumerator() 函数去检查另一个进程中的 heap.

考虑第二条路. 那么如何注入 dylib 到目标进程呢? 常用的做法是使用 环境变量 DYLD_INSERT_LIBRARIES, 但这个方法有几个问题:

* 从 OS X 10.8 开始这个环境变量被 dyld 忽略了. 不过我只是需要在 iOS 上使用, 而 iOS 上目前还可以使用
* 如何在 iOS 上指定这个环境变量?
	* iPhone 越狱工具中的 MobileSubstrate 也是用同样的技术实现了诸如, 它是在 /System/Library/LaunchDaemons/com.apple.SpringBoard.plist 中指定的(奇怪的是我手头的iphone 上这个plist 里却没有MobileSubstrate 的踪迹). 但如果我的目标进程不是通过 LaunchDaemons 启动的. 貌似就没有这么个 plist 文件. 不过 MobileSubstrate 支持加载第三方 dylib. 但需要改写代码使其符合 MobileSubstrate 的规范.
	* 直接从命令行指定, 然后起目标进程. 但这样起的进程没有 UI. 这个时候点击 iphone 上的进程图标, 似乎就解决了这个问题.  这样操作之后可以看到 heap_find.dylib 被加载起来了. 

```
iPhone:~ root# DYLD_INSERT_LIBRARIES=/sbin/heap_find.dylib heap_find &
[1] 5522
iPhone:~ root# vmmap 5522
RC 0 – Task: 3331
Unable to read target task’s memory @0×0 – kr 0×2
Region 0×0-0×90000 [576K](—/—; copy, private, not-reserved) default
… Version: -17958194, 12 images at offset 0×9
Unable to read target task’s memory @0×9 – kr 0×1
0×90000-0×93000 [12K](—/r-x; copy, private, not-reserved) default
… 0×93000-0×94000 [4K](—/rw-; copy, private, not-reserved) default
… 0×94000-0×134000 [640K](—/rw-; copy, private, not-reserved) default
… 0×134000-0×137000 [12K](—/r–; copy, private, not-reserved) default
… 0×137000-0x13a000 [12K](—/rwx; copy, private, not-reserved) default
…
mach_vm_region failed for address 0×40000000 – Error: 1
Image: /sbin/heap_find loaded @0×90000
Image: /sbin/heap_find.dylib loaded @0×137000
Image: /usr/lib/libobjc.A.dylib loaded @0x3458d000
Image: /usr/lib/libstdc++.6.dylib loaded @0x356da000
…
```	

这里用到的 vmmap 来自 Mac OS X and iOS Internals [5].
不过这种方式起的进程在 Xcode 里 Attach to Process 中找不到. 没法用 Xcode 调试. 麻烦.

这个问题最终的解决有些出人意料. 在 Xcode 里 attach 到 iphone 中目标进程之后, 我无意中使用了一下 process load 命令:

	(lldb) process load /sbin/heap_find.dylib
	Loading '/sbin/heap_find.dylib'&#8230;ok
	Image 0 loaded.
	(lldb) tar mo list
	&#8230;
	[138] BB9BFC7D-2423-31D2-9A79-ADF7EF7AAA18 0x2fe1c000 /var/root/Library/Developer/Xcode/iOS DeviceSupport/4.3.3 (8J2)/Symbols/usr/lib/dyld
	[139] 7130AF72-A0E6-36A4-85AE-3D7634576634 0&#215;05139000 /workspace/lldb/examples/darwin/heap_find/heap/heap_find.dylib
	      /workspace/lldb/examples/darwin/heap_find/heap/heap_find.dylib.dSYM/Contents/Resources/DWARF/heap_find.dylib
	      
其中的 `/sbin/heap_find.dylib` 是我复制到 iphone 的目录的. 加载成功之后, list module 可以看到这个 dylib. 奇怪的是, 虽然 load时的路径是 `/sbin/heap_find.dylib`. module list 显示的是我在 Mac 上编译的路径以及 dSYM 文件.
尝试了一下 dylib 中的 `find_cstring_in_heap()`, 已经可以查找字符串了.
	 
	(lldb) p find_cstring_in_heap('Identity_Name', 0)
	(malloc_match *) $1 = 0x0513c0e8
	(lldb) x/8x `$1`
	0x0513c0e8: 0x202ba400 0&#215;00000400 0&#215;00000072 0&#215;00000001
	0x0513c0f8: 0&#215;00000000 0&#215;00000000 0&#215;00000000 0&#215;00000000
	(lldb) p (char*)0x202ba400+0&#215;72
	(char *) $2 = 0x202ba472 'Identity_Name'
	(lldb) 
	
目前还是不知道为什么在 OS X 上这个命令失败了.  以后再研究.
自此大功告成. 在整个过程当中, 了解了 malloc API 的一些细节以及如何使用 introspect API. 对 lldb 的命令也更加熟练了一些. 更重要的是掌握了一种扩展调试器的方法.
上面提到的代码可以从 github [6] 下载.

## 后记
1. Mac OS X and iOS Internals [5] 提到一个工具 corerupt. 这个工具可以在 iOS 上直接对一个进程生成一个 core dump. 拿到这个 core dump 想从中找到一个字符串自然很容易. 但作者在论坛里说这个工具功能太强大了, 打算先不提供下载. 我去~
2. 需要经常的 scp 到 iphone. 用 wifi 不稳定. 可以使用 usbmuxd [7], 这样就可以通过数据线来 ssh 到 iphone 了.

## References
1. <http://lldb.llvm.org/python_reference/index.html>
2. <http://stackoverflow.com/questions/13843844/how-does-the-os-x-heap-command-line-utility-collect-its-information>
3. <http://osxbook.com>
4. <http://lists.cs.uiuc.edu/pipermail/lldb-dev/2013-August/002145.html>
5. <http://www.newosxbook.com/index.php>
6. <https://github.com/nicoster/heap_find>
7. <http://iphonedevwiki.net/index.php/SSH_Over_USB>
