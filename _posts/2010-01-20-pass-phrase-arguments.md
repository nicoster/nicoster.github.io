---
id: 105
title: pass phrase arguments
date: 2010-01-20T11:11:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=105
permalink: /pass-phrase-arguments/
tags:
  - compile
---
<h1><a name="PASS_PHRASE_ARGUMENTS">PASS PHRASE ARGUMENTS</a></h1>
Several commands accept password arguments, typically using -passin and -passout for input and output passwords respectively. These allow the password to be obtained from a variety of sources. Both of these options take a single argument whose format is described below. If no password argument is given and a password is required then the user is prompted to enter one: this will typically be read from the current terminal with echoing turned off. 
<dl>
<dt><a name="item_pass">pass:password</a> </dt>
<dd>
the actual password is password. Since the password is visible to utilities (like &#8216;ps' under Unix) this form should only be used where security is not important. 
</dd>
<dt><a name="item_env">env:var</a> </dt>
<dd>
obtain the password from the environment variable var. Since the environment of other processes is visible on certain platforms (e.g. ps under certain Unix OSes) this option should be used with caution. 
</dd>
<dt><a name="item_file">file:pathname</a> </dt>
<dd>
the first line of pathname is the password. If the same pathname argument is supplied to -passin and -passout arguments then the first line will be used for the input password and the next line for the output password. pathname need not refer to a regular file: it could for example refer to a device or named pipe. 
</dd>
<dt><a name="item_fd">fd:number</a> </dt>
<dd>
read the password from the file descriptor number. This can be used to send the data via a pipe for example. 
</dd>
<dt><a name="item_stdin">stdin</a> </dt>
<dd>
read the password from standard input. 
</dd>
</dl>
