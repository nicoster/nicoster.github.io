---
id: 720
title: 探寻 DWARF CFI
date: 2014-12-29T00:02:56+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=720
permalink: /explore-dwarf-cfi/
tags:
  - dbg
  - osx
  - dwarf
  - breakpad
  - symbol
  - dump_sys
  - STACK CFI
  - .dSYM
---
最近在使用 Breakpad 的过程当中, 发现了在指定了 symbol 的情况下 stackwalk 一个 dump 文件的调用栈只显示了一个 frame, 而在不指定 symbol 的情况下, 却能显示完整的调用栈, 当然, 所有的函数名都没有.
指定了 symbol, 只显示了一个 frame. 同时显示了 crash 的函数

	Thread 0 (crashed)
	 0  BreakpadTest!-[Controller causeCrash] [Controller.m : 43 + 0x3]
	    eip = 0x0002b96d   esp = 0xbffd5f30   ebp = 0xbffd5f58   ebx = 0x7b8aec50
	    esi = 0x0002d06c   edi = 0x7b89e6a0   eax = 0x0002b93c   ecx = 0x0002d07c
	    edx = 0x00000000   efl = 0x00010286
	    Found by: given as instruction pointer in context

不指定 symbol, 显示了所有的 frame, 虽然没有函数名

	Thread 0 (crashed)
	 0  BreakpadTest + 0x196d
	    eip = 0x0002b96d   esp = 0xbffd5f30   ebp = 0xbffd5f58   ebx = 0x7b8aec50
	    esi = 0x0002d06c   edi = 0x7b89e6a0   eax = 0x0002b93c   ecx = 0x0002d07c
	    edx = 0x00000000   efl = 0x00010286
	    Found by: given as instruction pointer in context
	 1  BreakpadTest + 0x1f55
	    eip = 0x0002bf55   esp = 0xbffd5f60   ebp = 0xbffd5fa8
	    Found by: previous frame's frame pointer
	 2  libobjc.A.dylib + 0x8853
	    eip = 0x99ca8853   esp = 0xbffd5fb0   ebp = 0xbffd5fc8
	    Found by: previous frame's frame pointer
	 3  AppKit + 0x39a328
	    eip = 0x98179328   esp = 0xbffd5fd0   ebp = 0xbffd5fe8
	    Found by: previous frame's frame pointer
	 4  libsystem_trace.dylib + 0xc03
	    eip = 0x97918c03   esp = 0xbffd5ff0   ebp = 0xbffd6018
	    Found by: previous frame's frame pointer
	 5  AppKit + 0x20da11
	    eip = 0x97feca11   esp = 0xbffd6020   ebp = 0xbffd60d8
	    Found by: previous frame's frame pointer
	 6  AppKit + 0x20d7ad
	    eip = 0x97fec7ad   esp = 0xbffd60e0   ebp = 0xbffd6108
	    Found by: previous frame's frame pointer
	 7  AppKit + 0x406c96
	    eip = 0x981e5c96   esp = 0xbffd6110   ebp = 0xbffd6138
	    Found by: previous frame's frame pointer
	 8  libsystem_trace.dylib + 0xc03
	    eip = 0x97918c03   esp = 0xbffd6140   ebp = 0xbffd6168
	    Found by: previous frame's frame pointer
	 9  AppKit + 0x2598f5
	    eip = 0x980388f5   esp = 0xbffd6170   ebp = 0xbffd61c8
	    Found by: previous frame's frame pointer
	10  AppKit + 0x3ea84a
	    eip = 0x981c984a   esp = 0xbffd61d0   ebp = 0xbffd61e8
	    Found by: previous frame's frame pointer
	11  AppKit + 0x4085b6
	    eip = 0x981e75b6   esp = 0xbffd61f0   ebp = 0xbffd6208
	    Found by: previous frame's frame pointer
	12  libsystem_trace.dylib + 0xc03
	...
	
为什么会出现这种情况呢? 跟踪 `minidump_stackwalk` 的代码发现, 问题出在 .sym 文件. .sym 就是由系统符号文件(在 mac 上是 .dSYM 文件) 通过 breakpad 提供的一个工具 `dump_syms` 生成的文本格式的符号文件. 其中包含各种记录, 比如 FUNC, 记录函数的地址, 以及对应的代码行. STACK 记录. 用于重建 callstack. 以及恢复 callstack 中各个 frame 的寄存器. 关于 .sym 文件的细节, 参见 
<https://code.google.com/p/google-breakpad/wiki/SymbolFiles>
现在问题就出在这个 STACK CFI 记录上. 以下是这个例子中 .sym 文件的部分 CFI (Call Frame Info) 记录:

	STACK CFI INIT 1930 55 .cfa: $ebp 4 + .ra: .cfa -4 + ^
	STACK CFI 1931 $esp: .cfa -8 + ^ .cfa: $ebp 8 +
	STACK CFI 1933 .cfa: $esp 8 +
	STACK CFI 1937 $esi: .cfa -12 + ^

其中, 1930 是函数地址, 也就是前面看到的 `-[Controller causeCrash]`. 55 是这个函数占用的字节数. .cfa (Canonical Frame Address) 可以认为是一个 frame 的基准地址. 它在一个 frame 中的确切位置不重要, 可以是 frame 的起始地址, 或者结束地址, 或者其他. `.cfa: $ebp 4 +` 的意思是, 在刚进入这个函数时, .cfa 就是 ebp + 4. 同样的, `.ra (Return Address) = *(int*)(.cfa - 4)`. 但这个解释不符合常理. 刚进入一个函数时, 一般的, ebp 是上一个 frame 的基址寄存器. 它的位置是不定的, 依赖于调用者的 frame. 而 esp 则是确定的, 它指向函数的返回地址. 堆栈如下图

	| Return Address |      <- esp
	| Arg1           |
	| Arg2           |

前面的链接举的例子描述的也是这样的情况. 再看看其他的 CFI 记录, 做一个大胆的猜测: 这些记录中的 ebp 和 esp 弄反了. 互换一下变成如下的样子:

	STACK CFI INIT 1930 55 .cfa: $esp 4 + .ra: .cfa -4 + ^
	STACK CFI 1931 $ebp: .cfa -8 + ^ .cfa: $esp 8 +
	STACK CFI 1933 .cfa: $ebp 8 +
	STACK CFI 1937 $esi: .cfa -12 + ^
	
再次执行 stackwalk 命令, 这次调用栈都显示完全了. 而且函数名也正确.
那么现在问题来了, 到底是哪里出错了呢. 是 dump_syms 解析调试信息有误, 还是 mac 的调试信息本来就有问题呢? 

继续分析 `dump_syms` 的代码. 它是读取 .dSYM 文件中的 DWARF 调试信息来生产的 .sym 文件. DWARF 信息很繁杂. 它只读取三个 section:

* debug-info
* debug-frame
* eh-frame

其中 debug-info 提供函数名, 代码行等信息. debug-frame 和 eh-frame 用于提供 CFI 信息. 经过跟踪调试, 发现 debug-frame 中记录着如何恢复哪一个寄存器. 其中寄存器保存的是一个编号. `dump_syms` hardcode 所有的寄存器的名字, 用编号来查表. 难道这里出现了不匹配, 正好把两者弄反了? 

想找找 Apple 的 DWARF 文档来求证一下. google之后知道了 dwarfdump 命令. 用来显示 .dSYM 中的 dwarf 信息.
	
	$ dwarfdump --debug-frame -e BreakpadTest
	----------------------------------------------------------------------
	 File: BreakpadTest (i386)
	----------------------------------------------------------------------
	.debug_frame contents:
	...
	0x00000014: FDE
	        length: 0x00000018
	   CIE_pointer: 0x00000000
	    start_addr: 0x00002930 -[Controller causeCrash]
	    range_size: 0x00000055 (end_addr = 0x00002985)
	  Instructions: 0x00002930: CFA=ebp+4     eip=[ebp]
	                0x00002931: CFA=ebp+8     esp=[ebp]  eip=[ebp+4]
	                0x00002933: CFA=esp+8     esp=[esp]  eip=[esp+4]
	                0x00002937: CFA=esp+8     esp=[esp]  esi=[esp-4]  eip=[esp+4]
	                
可以看到, Apple 自己的工具也说 CFA=ebp+4. 看来 dump_syms 没有问题. 继续 google 发现 breakpad 的开发者在 mailer 里提到 clang 生成的 CFI 不兼容. 

* <https://code.google.com/p/google-breakpad/issues/detail?id=443>
* <https://code.google.com/p/chromium/issues/detail?id=393594>

再用一个最简单的例子做实验:

	$ cat crasher.c
	int main(int argc, char* argv[]) {
	  __builtin_trap();
	  return 0;
	}
	$ clang -g -m32 crasher.c
	$ dwarfdump -e --debug-frame a.out.dSYM/Contents/Resources/DWARF/a.out
	----------------------------------------------------------------------
	 File: a.out.dSYM/Contents/Resources/DWARF/a.out (i386)
	----------------------------------------------------------------------
	.debug_frame contents:
	0x00000000: CIE
	        length: 0x00000010
	        CIE_id: 0xffffffff
	       version: 0x01
	  augmentation: ""
	    code_align: 1
	    data_align: -4
	   ra_register: 0x08
	  Instructions: Init State: CFA=ebp+4     eip=[ebp]
	0x00000014: FDE
	        length: 0x00000014
	   CIE_pointer: 0x00000000
	    start_addr: 0x00001f90 main
	    range_size: 0x00000027 (end_addr = 0x00001fb7)
	  Instructions: 0x00001f90: CFA=ebp+4     eip=[ebp]
	                0x00001f91: CFA=ebp+8     esp=[ebp]  eip=[ebp+4]
	                0x00001f93: CFA=esp+8     esp=[esp]  eip=[esp+4]
	$ dwarfdump -e --eh-frame a.out
	----------------------------------------------------------------------
	 File: a.out (i386)
	----------------------------------------------------------------------
	No section named __eh_frame was found.
	$ clang -g crasher.c
	$ dwarfdump -e --debug-frame a.out.dSYM/Contents/Resources/DWARF/a.out
	----------------------------------------------------------------------
	 File: a.out.dSYM/Contents/Resources/DWARF/a.out (x86_64)
	----------------------------------------------------------------------
	.debug_frame contents:
	< EMPTY >
	$ dwarfdump -e --eh-frame a.out
	----------------------------------------------------------------------
	 File: a.out (x86_64)
	----------------------------------------------------------------------
	Exception handling frame information for section __eh_frame
	0x00000000: CIE
	        length: 0x00000014
	        CIE_id: 0xffffffff
	       version: 0x01
	  augmentation: "zR"
	    code_align: 1
	    data_align: -8
	   ra_register: 0x10
	 aug_arg["zR"]: DW_GNU_EH_PE_pcrel + DW_GNU_EH_PE_absptr
	  Instructions: Init State: CFA=rsp+8     rip=[rsp]
	$

先编译成 i386 架构的, 可以看到 .dSYM 中的 CFI 是有问题的. eh-frame 为空
再编译成 x86_64 架构. 可以看到 .dSYM 中没有 debug-frame 信息. 而可执行文件中有 eh-frame 信息. 而且是正确的 `CFA=rsp+8`

现在 Apple 推 `x86_64` 架构, i386 架构的可能就顾不上更新了. 所以 `x86_64` 就没有这个问题? 
至于 `x86_64` 中为什么没有 debug-frame, 而只有 `eh-frame`. 这里可能能够解释:

<http://lists.cs.uiuc.edu/pipermail/llvmdev/2014-February/070150.html>

>"There is some interaction in that if you are generating a .eh_frame section for exception handling then there is little sense is generating a DWARF .debug_frame section as well; "

发掘的差不多了. 对于我遇到的问题, 一个 workaround 是 .sym 文件中不生成 STACK CFI 记录. 因为 Apple 没有启用 FPO. 所以并不会影响重建 callstack. 另外一方面就是继续挖掘 breakpad 的代码. 看看 google 的开发者是怎么干的.
