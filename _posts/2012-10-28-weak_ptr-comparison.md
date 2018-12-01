---
id: 666
title: weak_ptr comparison
date: 2012-10-28T20:36:12+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=666
permalink: /weak_ptr-comparison/
tags:
  - boost
  - weak_ptr
  - c++11
  - shared_ptr
  - owner_less
---

	set<weak_ptr<int> > ints;

这行代码在 c++11 上编译出现错误. 而使用 boost, tr1 中的 `weak_ptr` 则编译通过. 原因在于 c++11 中的 `weak_ptr` 不支持 std::less. 而另外两个则支持. c++11 提供了 `std::owner_less` 解决这个问题. 所以


	std<weak_ptr<int>, owner_less<weak_ptr<int>>> ints;

就可以在 c++11 正常工作了. 
这里需要讨论一下 `weak_ptr` 的比较问题. 其实已经有一些文章讨论这个, 下面是其中之一.
<http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2004/n1590.html>

smart ptr 的比较没有看起来的那么简单. 先看一下 `shared_ptr`/`weak_ptr` 的通常的内部实现:

![shared_ptr_internal.png]({{site.url}}/attachments/2012/10/shared_ptr_internal.png)

图片来自 <a href="http://view.officeapps.live.com/op/view.aspx?src=http%3a%2f%2fecn.channel9.msdn.com%2fevents%2fGoingNative12%2fGN12STL11.pptx" title="这里">这里</a>

它有两个指针, `_Ptr` 指向真正的对象, `_Rep` 指向所谓的 control block, 其中记录着 `shared_count`, `weak_count`. 对于 `shared_ptr` 来说, 有两种方式来比较大小. value-based, owner-based. 也就是分别比较 `_Ptr` 或者 `_Rep` 的值. 而 `weak_ptr` 则不能简单的比较 `_Ptr`, 因为 `weak_ptr` 随时可能 expire, `_Ptr` 的值从而改变(清零). 所以就只能比较 `_Rep`, 也就是 control block 的地址. 一般的, 为了和 `weak_ptr` 的实现一致, `shared_ptr` 也采用 owner-based 的机制.

上面提到的 `owner_less` 会调用 `shared_ptr`/`weak_ptr` 的成员函数 `owner_before()`, 其实现就是比较 control block 的地址.
