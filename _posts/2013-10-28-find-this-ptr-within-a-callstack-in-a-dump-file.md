---
id: 709
title: 'find "this" ptr within a callstack'
date: 2013-10-28T21:36:29+00:00
author: nick
layout: post
permalink: /find-this-ptr-within-a-callstack/
tags:
  - dbg
  - esi
  - find_this
  - pykd
  - windbg
  - python
---

When you're inspecting a dump file using Windbg, you may want to know the states of an object, to be specific, the members of a C++ object in the context. In order to do that, you need to find out the location of the object ie. the 'this' ptr. but how?

Before we go any further, let's take a brief look of C++ ABI.
In the Microsoft implementation of x86 C++ ABI, usually register ECX serves as the 'this' ptr. So the following C++ code will be translated to something like:

```cpp
foo = get_foo();
foo.test()
```

```x86asm
lea ecx, [ebp-10h]     ; load the address of foo
call Foo::test
```
	
inside Foo::test(), when it in turn calls another function Bar::test2:

```cpp
Foo::test(){
		Bar bar = get_bar();
		bar.test2();
		...
}
```

ecx will be loaded with the address of bar and then make the call. As the 'this' ptr to foo will still be used later on, it's necessary to save ecx before loading bar.
The assembly code might be:

	mov esi, ecx     ; save ecx to esi
	lea ecx, [ebp-20h]     ; load the address of bar
	call Bar::test2
	
	
the same goes to test2 which needs to save registers before using them, so the assembly of which may look like:

	push esi     ; save to stack
	push edi
	push ebx
	...
	

As we can see in the following callstack, the address of foo was saved into ecx (in main), then copied to esi (in test), and then stored to the stack (in test2), where we're going to find the ‘this' ptr.

	1. Bar::test2
	2. Foo::test
	3. main

So the steps to locate the 'this' ptr to foo could be summarized as:

1. Unassemble the code of Foo::test, to find out which register holds the ‘this' ptr. (esi in previous example)
2. Unassemble the code of function in next frame, that is, Bar::test2, to find out the offset where the register (esi) gets stored in the stack
3. Once we get the offset, add it to the ChildEBP in frame 2, dereference the address. done.

The process seems straightforward if you know some assembly. But if you don't, or you need to deal with dump files everyday, it soon becomes a bit challenging and/or tedious. That's where the script findthis.py (<a href="https://gist.github.com/nicoster/7195565">https://gist.github.com/nicoster/7195565</a>) comes to rescue.

Basically this script does step 2 and 3 for you. As it hasn't got that far to figure out in which register the 'this' ptr stores, it enumerates all the registers stored in the stack for each frame. And using the DML hyperlinks, you can click the links to view them as specific objects. The script is written in python. An extension pykd(<a href="http://pykd.codeplex.com/">http://pykd.codeplex.com/</a>) needs to be installed and loaded before running the script in Windbg.

Some known issues with the script:

1. cannot handle function override.
2. the value might not be 100% correct.
3. may have duplicate registers. Could be enhanced.
4. apart from `_EH_prolog3`/`_EH_prolog3_GS`, doesn’t handle other prologs (as they're uncommon in user code). Could be enhanced.
5. ~~reversed frames in the output~~ Enhanced.

If you would like to improve the script, fork it on github.
Below is a screenshot of the output by the script:

![findthis.png]({{site.url}}/attachments/2013/10/findthis.png)
