---
id: 495
title: ssh login without passwd
date: 2011-06-20T19:55:56+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=495
permalink: /ssh-login-without-passwd/
tags:
  - ssh
  - keygen
  - scp
---
先看这篇帖子:
http://www.linuxproblem.org/art_9.html
其中的最下面的 note 要注意:
Depending on your version of SSH you might also have to do the following changes:
<ul>
<li>Put the public key in <tt>.ssh/authorized_keys2</tt></li>
<li>Change the permissions of <tt>.ssh</tt> to <tt>700</tt></li>
<li>Change the permissions of <tt>.ssh/authorized_keys2</tt> to <tt>640</tt></li>
</ul>
如果还是提示要密码. 尝试看看 log
<blockquote>sudo <span style="font-family: Consolas, Monaco, 'Courier New', Courier, monospace; font-size: 12px; line-height: 18px; white-space: pre;">grep -ir ssh /var/log/*</span></blockquote>
我的机器上遇到这个 error:
<pre>/var/log/secure:Jun 20 19:07:54 CT53-64-BASE sshd[28890]: \
Authentication refused: bad ownership or modes for directory /disk1/home/nick/
</pre>
可以这样来 fix:
<blockquote>
<pre>chmod go-w ~/
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys</pre>
</blockquote>
 
<h3>为什么有 authorized_keys 和 authorized_keys2?</h3>
In OpenSSH prior to version 3, the sshd man page used to say:
The $HOME/.ssh/authorized_keys file lists the RSA keys that are permitted for RSA authentication in SSH protocols 1.3 and 1.5 Similarly, the $HOME/.ssh/authorized_keys2 file lists the DSA and RSA keys that are permitted for public key authentication (PubkeyAuthentication) in SSH protocol 2.0.The release announcement for version 3 states that authorized_keys2 is deprecated and all keys should be put in the authorized_keys file.
