---
id: 724
title: Remove Sophos without Password
date: 2016-10-18T00:10:02+00:00
author: nick
layout: post
tags:
    - unintall
    - sophos
    - passwords
---


Do this at your own risk. It works for me and I hope it works for you too.

using `kextstat|grep sophos` you will find 3 sophos kexts.
using `kextfind` you could find these kexts are located at:

```
/Library/Extensions/SophosFileProtection.kext
/Library/Extensions/SophosPortInterceptor.kext
/Library/Extensions/SophosWebProtection.kext
```

now mount the file system as read-write

`$ sudo mount -uw /`

remove them all

```
$ sudo rm -rf /Library/Extensions/SophosFileProtection.kext
$ sudo rm -rf /Library/Extensions/SophosPortInterceptor.kext
$ sudo rm -rf /Library/Extensions/SophosWebProtection.kext
```

reboot your macOS, during the shutdown I met a kernel panic. 

now remove the application files (maybe this step could be performed before the first reboot)

```
$ sudo mount -uw /
$ sudo rm -rf /Library/Sophos\ Anti-Virus
```

reboot again, during the restart, I got a panic once again, reboot again, everything's fine. 

bye sophos.




 