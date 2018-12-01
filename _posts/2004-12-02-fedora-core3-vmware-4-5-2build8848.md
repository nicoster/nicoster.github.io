---
id: 169
title: Fedora Core3 @ vmware 4.5.2Build8848
date: 2004-12-02T20:01:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=169
permalink: /fedora-core3-vmware-4-5-2build8848/
categories:
  - Uncategorized
---
今天在 vmware 上整了一个 FC3. 感觉不错阿.
创建了一个 4G 的硬盘. 第一次装的时候没有注意安装大小. 居然提示空间不足. 没办法, 只好重来. 再次选的时候, 把一些奢侈的东东就免了. 安装停顺利的.
装完后重起, 到了登录界面. 直接输入用户名,密码.居然没有见到 KDE!奇怪. 我明明选了的阿. 原来在登录的时候有个关于 '会话'的选项.在哪里可以选择 KDE.然后登录就可以进入KDE了. 
然后就发现分辨率只有800x 600. 上网一查, 发现原来是没有安装 vmware-tools. 按照说明一步一步的安装之后. 完成配置. 发现进不了 X 了! 继续找. 的确有人也遇到这个问题. 接着就找到了解决方案.见:
http://kerneltrap.org/node/view/4030
这个老外还提供了一个 vmware-tools的一个补丁. 按照他的说明把补丁打上. 顺便学他的去掉了一些没用的服务, reboot. 可以了. 速度也快多了. 
这段文字就是在 fedora 下写出来的. 它自带的 智能拼音 虽然比不上紫光. 但已经很出色了. nice..
