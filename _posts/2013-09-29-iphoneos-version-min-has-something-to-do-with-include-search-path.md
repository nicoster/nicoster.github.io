---
id: 699
title: iphoneos-version-min has something to do with include search path
date: 2013-09-29T16:07:57+00:00
author: nick
layout: post
permalink: /iphoneos-version-min-has-something-to-do-with-include-search-path/
tags:
  - ios
  - osx
  - include search path
  - iphoneos-ver-min
  - xcode5
---
After I upgrade to Xcode 5, some projects failed with the following error:

	fatal error: 'tr1/functional' file not found
	#include <tr1/functional>
	
Then I checked the iOS SDK 7.0 that was shipped with the Xcode 5 and I found that there’s actually the tr1 folder in the header directory:

	/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/usr/include/c++/4.2.1/tr1  

Issue the same command which failed in Xcode, but plus -v to let clang show the verbose output:

	$ clang -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk -v -arch armv7 -miphoneos-version-min=7.0 ...
	Apple LLVM version 5.0 (clang-500.2.76) (based on LLVM 3.3svn)
	Target: arm-apple-darwin13.0.0
	Thread model: posix
	 ...
	clang -cc1 version 5.0 based upon LLVM 3.3svn default target x86_64-apple-darwin13.0.0
	ignoring nonexistent directory "/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/usr/include/c++/v1"
	ignoring nonexistent directory "/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/usr/local/include"
	ignoring nonexistent directory "/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/Library/Frameworks"
	#include "..." search starts here:
	#include <...> search starts here:
	 ...
	 /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../lib/c++/v1
	 /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../lib/clang/5.0/include
	 /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/include
	 /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/usr/include
	 /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/System/Library/Frameworks (framework directory)
	End of search list.
	…
	
We can see the `iPhoneOS7.0.sdk/usr/include/c++/4.2.1` hasn’t been searched. After several attempts I found it’s the `-miphoneos-version-min=7.0` makes the difference. When I changed it to 4.3, the c++ header directory has been searched and it builds okay.

	$ clang -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk -v -arch armv7 -miphoneos-version-min=4.3 ...
	...
	#include "..." search starts here:
	#include <...> search starts here:
	 /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/usr/include/c++/4.2.1
	 /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/usr/include/c++/4.2.1/backward
	 /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../lib/clang/5.0/include
	 /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/include
	 /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/usr/include
	 /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/System/Library/Frameworks (framework directory)
	End of search list.
	... 
