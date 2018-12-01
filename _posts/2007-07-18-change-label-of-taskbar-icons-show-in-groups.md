---
id: 126
title: Change Label of Taskbar Icons Show In Groups
date: 2007-07-18T20:27:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=126
permalink: /change-label-of-taskbar-icons-show-in-groups/
tags:
  - taskbar
  - xp
  - FileDescription
  - explorer
---
xp 的一个新特性. taskbar icons show in groups. 任务栏图标分组显示. 不知大家注意到没有. 分组后图标的 label 有可能变化. 如图, IE 的 label 本来是 about:blank : Microsoft Internet Explorer. 分组之后显示的是 Internet Explorer. 前者很明白. 自然就是 window title. 那么后者呢? 手头有这么个问题. 
<img alt="" src="http://p.blog.csdn.net/images/p_blog_csdn_net/ArCoolGG/b.PNG" />_<img alt="" src="http://p.blog.csdn.net/images/p_blog_csdn_net/ArCoolGG/a.PNG" />
一开始就是满世界 google. 没有结果. 找到最多的就是教你如何禁用 balloon tooltip. 然后有试了试 spy++, 鼠标一放上去. 一大堆消息. 最多的就是 HITTEST. 也没有收获. 又想试试 windbg. 可惜无处下手啊. 想设断点都不知道该找那个 API. ShowToolTip()? 我想的太天真了. 最后. 我想到了一个办法. 写一个简单的程序. 把默认的字符串都带上标记. 比如 DemoProgram 的 Mainframe 就改成 DemoProgramMainFrame, AppTitle 就改成 DemoProgramAppTitle 等等. 改了之后编译. 没有变化? 奇怪. 折腾了一会儿. 又 rebuild all 了一把. 嗯, 变化了. 分组后的图标 label 变成了 DemoProgramFileDesc. 原来是版本信息中的文件描述. 又改成别的试了试. 嗯? 居然不变化了? 无奈, 搜索注册表. 哈哈. 被我找到了这么一个键值:
 HKEY_CURRENT_USER\Software\Microsoft\Windows\Shell\NoRoam\MUICache 这个键下有这样的值/数据:d:\demo\program.exe=DemoProgramFileDesc
原来被资源管理器缓存了. 直接修改这个键值. 重启程序就可以看到生效了. 又试了试如果 FileDescription 如果为空会怎么样? 发现 Explorer 会用 exe 的文件名作为名称. 
试了这么多办法. 还是土办法管用. 其实还可以试试著名的 Process Monitor. 不过在这个例子中可能也不能奏效.  
