---
id: 161
title: 转一篇候捷的 Inside C++ Object Model 译本的译序
date: 2006-05-11T08:59:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=161
permalink: /%e8%bd%ac%e4%b8%80%e7%af%87%e5%80%99%e6%8d%b7%e7%9a%84-inside-c-object-model-%e8%af%91%e6%9c%ac%e7%9a%84%e8%af%91%e5%ba%8f/
categories:
  - Uncategorized
---
<em>最近在读 Inside C++ Object Model; Essential COM</em>
<em>转一篇候捷的 Inside C++ Object Model 译本的译序</em>
对於传统的循序性（sequential）语言，我们向来没有太多的疑惑，虽然在函式呼叫的背後，也有着堆叠建制、叁数排列、回返位址、堆叠清除等等幕後机制，但函式呼叫是那麽地自然而明显，好像只是夹带着一个包裹，从程式的某一个地点跳到另一个地点去执行。    但是对於物件导向（Object Oriented）语言，我们的疑惑就多了。究其因，这种语言的编译器为我们（程式员）做了太多的服务：建构式、解构式、虚拟函式、继承、多型&#8230;。有时候它为我们合成出一些额外的函式（或运算子），有时候它又扩张我们所写的函式内容，放进更多的动作。有时候它还会为我们的 objects 加油添醋，放进一些奇妙的东西，使你面对 sizeof 的结果大惊失色。    存在我心里头一直有个疑惑：电脑程式最基础的形式，总是脱离不了一行一行的循序执行模式，为什麽OO（物件导向）语言却能够「自动完成」这麽多事情呢？另一个疑惑是，威力强大的 polymorphism（多型），其底层机制究竟如何？     如果不了解编译器对我们所写的 C++ 码做了什麽手脚，这些困惑永远解不开。     这本书解决了过去令我百思不解的诸多疑惑。我要向所有已具备 C++ 多年程式设计经验的同好们大力推荐这本书。    这本书同时也是跃向元件软体（component-ware）基本精神的跳板。不管你想学习 COM（Component Object Model）或CORBA（Common Object Request Broker Architecture）或是 SOM（System Object Model），了解 C++ Object Model，将使你更清楚软体元件（components）设计上的难点与运应之道。不但我自己在学习 COM 的道路上有此强烈的感受，Essential COM（COM 本质论，侯俊杰译， 峰 1998）的作者 Don Box 也在他的书中推崇 Lippman 的这一本卓越的书籍。 ?    是的，这当然不会是一本轻松的书籍。某些章节（例如３、４两章）可能给你立即的享受 &#8212; 享受於面对底层机制有所体会与掌控的快乐；某些章节（例如５、６、７三章）可能带给你短暂的痛苦 &#8212; 痛苦於艰难深涩难以吞咽的内容。这些快乐与痛苦，其实就是我翻译此书时的心情写照。无论如何，我希望透过我的译笔，把这本难得的好书带到更多人面前，引领大家见识C++ 底层建设的技术之美。 
