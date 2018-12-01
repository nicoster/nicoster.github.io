---
id: 159
title: 小议 COM 线程模型
date: 2006-07-21T16:20:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=159
permalink: /%e5%b0%8f%e8%ae%ae-com-%e7%ba%bf%e7%a8%8b%e6%a8%a1%e5%9e%8b/
categories:
  - Uncategorized
---
最开始的时候 COM 没有多线程的概念. inproc 组件也没有 InprocServer32 这样的键值.所有的组件都认为自己在唯一的线程中被创建. 使用. 不用同步访问任何数据. 因为一个进程中只有一个线程.所有的组件都挤在 main STA 执行. 等一个组件的函数调用执行完成, 另一个组件的函数调用才可能继续.
然后多线程出现了. inproc 组件开始支持 InprocServer32=apartment, 组件可以在不同的线程里被创建. apartment 组件要做的是同步全局数据. 比如, 两个线程创建了同一个组件的两个实例. 如果组件有一些独立于实例的全局数据. 那么这些数据需要同步访问. 情况好转了一些. 一个组件可以在一个单独的线程执行. 在一个线程内部访问该线程创建的组件. 不用调度, 可以直接访问.
但问题依然存在, 跨线程访问一个组件时. 需要调度. 不论是否真的需要同步访问, 每次都需要等待组件的一个函数调用完成, 才能继续另一个函数调用. 毕竟只有一个线程. 然后就出现了 InprocServer32=free 的组件. 一个进程中多个线程可以加入到一个 MTA 中. 这个 MTA 中的线程可以自由的访问这个 MTA 中的组件的方法. 要不要同步. 全由程序员来决定了.对于 COM runtime 来说. 它可以不用再管组件的同步. 所有的工作都交给了 COM 程序员. 而在 single 和 apartment 中.COM runtime 还为 COM 组件创建了一个隐藏的窗口来实现函数调用的同步. 
一个 STA 客户访问一个 free 的组件. 那么还是需要调度. 影响效率. 所以. 出现了 InprocServer32=both 的组件. 当 STA 客户访问时. 它就是 STA 组件. 可以直接调用. 当 MTA 客户访问时. 它就是 MTA 组件.  也可以直接调用.  
COM 的线程模型其实也用来解决 COM 在多线程中遇到的问题. 它用于减轻程序员的编程负担.  Not a burden, but a helper.
