---
id: 722
title: Erlang VM debugging
date: 2016-02-25T00:10:02+00:00
author: nick
layout: post
tags:
  - erlang
  - erts
  - gdb
  - dbg
  - Eterm
  - debuginfo-install
  - erts_printf
---
最近遇到一个 erlang VM 的 crash, 借这个机会熟悉了一下 gdb 的调试 erlang VM.

不管调试什么, 第一步要解决的是 symbols.

由于我们是自己编译的 VM, 甚至都没有 strip. 所以完全没有符号的问题. 当然, OS 的符号还是需要安装的. AWS EC2 的常用的 .so 的符号可以用下面的命令安装一下

	debuginfo-install expat-2.1.0-8.18.amzn1.x86_64 glibc-2.17-78.161.amzn1.x86_64 keyutils-libs-1.5.8-3.12.amzn1.x86_64 krb5-libs-1.12.2-14.43.amzn1.x86_64 libcom_err-1.42.12-4.40.amzn1.x86_64 libselinux-2.1.10-3.22.amzn1.x86_64 libyaml-0.1.6-6.7.amzn1.x86_64 ncurses-libs-5.7-3.20090208.13.amzn1.x86_64 openssl-1.0.1k-10.87.amzn1.x86_64 zlib-1.2.8-7.18.amzn1.x86_64
	
一开始分析 core 文件时用的 `-c` 的参数调用 gdb

	gdb -c core.12345
	
发现所有模块的 symbol 都没有被加载. 用命令 [symbol-file](http://www.delorie.com/gnu/docs/gdb/gdb_125.html) 貌似可以加载单个 .so 的 symbol. 但 `info sh` 查看还是显示没有 symbol 没有.

几次尝试之后, 发现同时指定 executable 和 core 就没有问题了

	gdb /usr/local/lib/erlang/erts-7.2.1/bin/beam.smp core.12345
	
	(gdb) info sh
	From                To                  Syms Read   Shared Object Library
	0x00007f6c83e44f10  0x00007f6c83e45804  Yes         /lib64/libutil.so.1
	0x00007f6c83c40ed0  0x00007f6c83c419d0  Yes         /lib64/libdl.so.2
	0x00007f6c839434b0  0x00007f6c839ada08  Yes         /lib64/libm.so.6
	0x00007f6c83729880  0x00007f6c83733038  Yes         /lib64/libtinfo.so.5
	0x00007f6c83509200  0x00007f6c83515798  Yes         /lib64/libz.so.1
	0x00007f6c832f08a0  0x00007f6c832fb544  Yes         /lib64/libpthread.so.0
	0x00007f6c830e52c0  0x00007f6c830e80bc  Yes         /lib64/librt.so.1
	0x00007f6c82d403c0  0x00007f6c82e84510  Yes         /lib64/libc.so.6
	...
	
使用 `set dir` 指定源码的目录

	set dir  ~/.kerl/builds/r1821-hipe/otp_src_18.2.1/erts/emulator/
	
使用 `p` 命令, 可以打印参数, 变量的值

	(gdb) fr 5
	#5  erts_send_message (sender=sender@entry=0x7f6c0462a210, receiver=receiver@entry=0x7f6c1c58e6c8, receiver_locks=receiver_locks@entry=0x7f6c643fdba0, message=message@entry=140101919687994, 
	    flags=flags@entry=0) at beam/erl_message.c:1033
	1033            hp = erts_alloc_message_heap_state(msize,
	(gdb) p sender->common
	$5 = {id = 118919054547027, refc = {atmc = {counter = 1}, sint = 1}, tracer_proc = 18446744073709551611, trace_flags = 0, timer = {counter = 0}, u = {alive = {started_interval = 2, reg = 0x0, 
	      links = 0x7f6c0462a578, monitors = 0x0}, release = {later = 2, func = 0x0, data = 0x7f6c0462a578, next = 0x0}}}

`p` 命令还可以执行 debuggee 里的函数, 比如我想加 `sender->common->id` 转换成 erlang pid, 可以执行:

	p erts_printf("%T\n", sender->common->id)
	
但是, gdb 有一个限制, 就是必须 live debugging 才能执行函数. 所以我就单独起一个 erl 进程, gdb attach 过去. 在执行这条命令就可以了. 所以还是尽可能直接调试进程, 而不是 core.

`erts_printf()` 这个函数在调试 erlang VM 时非常有用. 它可以打印任何 [Eterm](http://www.cnblogs.com/zhengsyao/p/erlang_eterm_implementation_2.html) 数据. 

erlang 中有很多函数是 bif. 比如 `erlang:self()` 对应的就是 self_1() 这个 c 函数, 所以, 你可以在 gdb 中放断点, 然后在 erlang shell 调用 self(), 这边断点就到了:

	(gdb) b self_0
	Breakpoint 1 at 0x4b6550: file beam/bif.c, line 3575.
	(gdb) info br
	Num     Type           Disp Enb Address            What
	1       breakpoint     keep y   0x00000000004b6550 in self_0 at beam/bif.c:3575
	(gdb) c
	Continuing.
	[Switching to Thread 0x7f6e46ceb700 (LWP 4744)]
	
	Breakpoint 1, self_0 (A__p=0x7f6e620c0d48, BIF__ARGS=0x7f6e5a9f2900) at beam/bif.c:3575
	3575         BIF_RET(BIF_P->common.id);
	(gdb) bt
	#0  self_0 (A__p=0x7f6e620c0d48, BIF__ARGS=0x7f6e5a9f2900) at beam/bif.c:3575
	#1  0x0000000000443ede in process_main () at beam/beam_emu.c:3690
	#2  0x00000000004cfd8a in sched_thread_func (vesdp=0x7f6e5aacbcc0) at beam/erl_process.c:8021
	#3  0x000000000061fe6b in thr_wrapper (vtwd=<optimized out>) at pthread/ethread.c:114
	#4  0x00007f6e68be0df5 in start_thread (arg=0x7f6e46ceb700) at pthread_create.c:308
	#5  0x00007f6e68705bfd in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:113
	
反汇编 pc, 或者一个函数:

	(gdb) disas 
	Dump of assembler code for function self_0:
	=> 0x00000000004b6550 <+0>:     mov    (%rdi),%rax
	   0x00000000004b6553 <+3>:     retq   
	End of assembler dump.
	
	(gdb) disas nodes_1
	Dump of assembler code for function nodes_1:
	   0x000000000050ef30 <+0>:     push   %r14
	   0x000000000050ef32 <+2>:     push   %r13
	   0x000000000050ef34 <+4>:     mov    %rdi,%r13
	   0x000000000050ef37 <+7>:     push   %r12
	   0x000000000050ef39 <+9>:     push   %rbp
	   0x000000000050ef3a <+10>:    push   %rbx
	   0x000000000050ef3b <+11>:    sub    $0x10,%rsp
	   0x000000000050ef3f <+15>:    mov    (%rsi),%rax
	   0x000000000050ef42 <+18>:    mov    %rax,%rdx
		...
		
使用 `info fun` 列出所有的函数 (列表会很长)

	(gdb) info fun
	All defined functions:
	
	File sys/unix/erl_main.c:
	int main(int, char **);
	
	File beam/beam_load.c:
	Eterm code_get_chunk_2(Process *, Eterm *);
	Eterm code_module_md5_1(Process *, Eterm *);
	Binary *erts_alloc_loader_state(void);
	Eterm *erts_build_mfa_item(FunctionInfo *, Eterm *, Eterm, Eterm *);
	Eterm erts_finish_loading(Binary *, Process *, ErtsProcLocks, Eterm *);
	int erts_is_module_native(BeamInstr *);
	...

进一步google/研究之后, 发现原来 erlang VM 的开发者早就考虑到这个需求, 在源码目录中有一个 etp-commands.in. 在 gdb 中导入之后就有了一堆可用的命令:

	(gdb) source .kerl/builds/r1821-hipe/otp_src_18.2.1/erts/etc/unix/etp-commands.in
	%---------------------------------------------------------------------------
	% Use etp-help for a command overview and general help.
	%
	% To use the Erlang support module, the environment variable ROOTDIR
	% must be set to the toplevel installation directory of Erlang/OTP,
	% so the etp-commands file becomes:
	%     $ROOTDIR/erts/etc/unix/etp-commands
	% Also, erl and erlc must be in the path.
	---Type <return> to continue, or q <return> to quit---
	%---------------------------------------------------------------------------
	etp-set-max-depth 20
	etp-set-max-string-length 100
	--------------- System Information ---------------
	OTP release: 18
	ERTS version: 7.2.1
	Compile date: Fri Jan 15 08:45:35 2016
	Arch: x86_64-unknown-linux-gnu
	Endianness: Little
	Word size: 64-bit
	Halfword: no
	HiPE support: yes
	SMP support: yes
	Thread support: yes
	Kernel poll: Supported and used
	Debug compiled: no
	Lock checking: no
	Lock counting: no
	Node name: 'examplexmpp@ip-10-0-0-129'
	Number of schedulers: 32
	Number of async-threads: 10
	--------------------------------------------------
	(gdb) etp-process-info sender
	  Pid: <0.8048.5>
	  State: running | active | prq-prio-normal | usr-prio-normal | act-prio-normal
	  I: #Cp<gen:do_call/4+0x1c0>
	  Heap size: 4185
	  Old-heap size: 28690
	  Mbuf size: 0
	  Msgq len: 0 (inner=0, outer=0)
	  Parent: <0.7981.5>
	  Pointer: (Process *) 0x7f3f7e5e4cc0
	(gdb) etpf message
	<etpf-boxed 0x7f3e0eadf022>.
	(gdb) etpf-boxed 0x7f3e0eadf022
	{route,<etpf-boxed 0x7f3eb28063da>,<etpf-boxed 0x7f3eb280641a>,<etpf-boxed 0x7f3e0eade632>}.
	(gdb) etp 0x7f3eb28063da
	{jid,#HeapBinary<0x16,0x61767939317a6573,0x6f6861627a756c73,0x71695f697230>,#RefcBinary<0xc,0x7f3eb2806728,0x7f3ea1394240,0x7f3ea1394258,0>,#HeapBinary<0>,#RefcBinary<0x16,0x7f3eb28065d0,0x7f3e0bd9b628,0x7f3e0bd9b640,0>,#RefcBinary<0xc,0x7f3eb2806610,0x7f3e0bd9b668,0x7f3e0bd9b680,0>,#RefcBinary<0,0x7f3eb2806640,0x7f3e0bd9b7e0,0x7f3e0bd9b7f8,0>}.
	
不过这个脚本看起来还是在 32位 时代写的. 里面的 `printf` 打印使用的都是 `%#x`. 所以指针都显示不全. 在 etp-command.in 替换所有的 `%#x` 为 `%#lx` 就显示正常了.

另外, 这个脚本没有打印 HeapBinary, RefcBinary. 我[改进](https://gist.github.com/nicoster/335487384baa45ab7167/revisions)了一下. 现在可以直接显示 HeapBinary, RefcBinary 了:

	(gdb) etp message
	{route,{jid,<<"sez19yvasluzbaho0ri_iq">>,<<"xmpp.example.com">>,<<"">>,<<"sez19yvasluzbaho0ri_iq">>,<<"xmpp.example.com">>,<<"">>},{jid,<<"sez19yvasluzbaho0ri_iq">>,<<"xmpp.example.com">>,<<"exampleChat_pc">>,<<"sez19yvasluzbaho0ri_iq">>,<<"xmpp.example.com">>,<<"exampleChat_pc">>},{xmlel,<<"iq">>,[{<<"id">>,<<"{B580111F-1BEA-4202-8AC6-79442144B714}">>},{<<"type">>,<<"result">>}],[{xmlel,<<"example">>,[{<<"xmlns">>,<<"example:iq:ext">>},{<<"type">>,<<"roster">>}],[{xmlel,<<"default">>,[{<<"version">>,<<"0">>}],[]},{xmlel,<<"group">>,[{<<"id">>,<<"example.com">>},{<<"version">>,<<"83">>},{<<"name">>,<<"example.com##2##example.com">>}],[{xmlel,<<"item">>,[{<<"jid">>,<<"z4bvtrnrtdck_1edr0ljsq@xmpp.example.com">>},{<<"phoneno">>,<<"">>},{<<"lname">>,<<"Weist">>},{<<"fname">>,<<"Matthew">>},{<<"nickname">>,<<"Matthew Weist">>},{<<"name">>,...}],[]},{xmlel,<<"item">>,[{<<"jid">>,<<"yvwdhlassbgxthlskovzeq@xmpp.example.com">>},{<<"phoneno">>,<<"">>},{<<"lname">>,<<"Kazmierczak">>},{<<"fname">>,<<"Patricia">>},{<<"nickname">>,...},{...}],[]},{xmlel,<<"item">>,[{<<"jid">>,<<"yri-vu4itg2r_h7j0y86og@xmpp.example.com">>},{<<"phoneno">>,<<"">>},{<<"lname">>,<<"
	...
	
## References
* [Erlang数据类型的表示和实现（5）——binary](http://www.cnblogs.com/zhengsyao/p/erlang_eterm_implementation_5_binary.html)
* [An example analysis of a BEAM process core dump](https://gist.github.com/studzien/773baa48a432e70021c2)