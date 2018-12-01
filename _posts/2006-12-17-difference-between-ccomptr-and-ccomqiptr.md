---
id: 141
title: Difference between CComPtr and CComQIPtr
date: 2006-12-17T21:22:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=141
permalink: /difference-between-ccomptr-and-ccomqiptr/
categories:
  - Uncategorized
tags:
  - atl
  - CComPtr
  - CComQIPtr
---
可能每个人学习 ATL 的时候都要区分 CComPtr 和 CComQIPtr 的不同. CComQIPtr 是 CComPtr 的超集, 但 CComQIPtr 不能用于 IUnknown. 为什么不能用. 
#include <atlbase.h>void main(){CComQIPtr<IUnknown> spunk;}
果然, 编译错误. 原来错误的原因只是 CComQIPtr 定义了  
CComQIPtr(T* lp){..}CComQIPtr(IUnknown* lp){..}
两个函数, 当 T = IUnknown 的时候, 函数重定义了. 还有另两个函数有这样的冲突.查看 atlbase.h, 可以看到 CComQIPtr 还有一个特化的版本:
template<>class CComQIPtr<IUnknown, &IID_IUnknown>{..}
这个版本可以让这样的代码通过编译:
CComQIPtr <IUnknown, &IID_IUnknown> spunk;
但 
CComQIPtr<IUnknown> sp;
为什么不能通过编译呢? CComQIPtr<IUnknown> 取了默认的模板参数, 成为 CComQIPtr<IUnknown, &__uuidof(IUnknown)>. __uuidof 这个操作符返回一个类型:
struct __s_GUID _GUID_00000000_0000_0000_c000_000000000046
很显然, 编译器不认为它和 IID_IUnknown 相同.  
