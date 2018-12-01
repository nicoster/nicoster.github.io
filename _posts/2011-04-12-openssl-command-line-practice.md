---
id: 425
title: OpenSSL Command-Line Practice
date: 2011-04-12T18:23:17+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=425
permalink: /openssl-command-line-practice/
tags:
  - openssl
  - rsa
  - genrsa
---
<h4 id="section-OpenSSL+Command-Line+Practice-RSA_E7_9A_84_E7_9B_B8_E5_85_B3_E6_93_8D_E4_BD_9C">RSA 的相关操作</h4>
<ul>
<li>创建密钥</li>
</ul>
<pre>D:\Temp>openssl genrsa -out pri.pem 1024
Loading 'screen' into random state - done
Generating RSA private key, 1024 bit long modulus
..................++++++
...........................++++++
e is 65537 (0x10001)

D:\Temp>type pri.pem
-----BEGIN RSA PRIVATE KEY-----
MIICXAIBAAKBgQDK7TIBRai7Xq6dwU9IxbmvIh47nt6VPZ+OwSJv32xTieGNS5TS
FPF6q5piSkuqCalJPoyWn39+AzvgKRFLi+rCgeByLu8ijPX/VgujLkKp/VPCaXx5
Okf36HxxUbu6ODw0H8z8ARlnoXz+L2dT5XdhC/7PDsodawCFCohbTjzmEwIDAQAB
AoGAA03HUaP7sklBWIosK0gk1Mgea+QTRaTCM0XLtLyTe+yzwmQnoR/8Kn4evljt
UHBl1C5zhYRFRBzzXZvtjyhRAyBBLMI+Jwmju0oPPLlXJzURQZxP2+5ivwxwUu/I
iCSoWCbr9IAcUOPJHGSG275kJ9IQCa9lSWlCiUjmCyZn1MECQQDxPZpzPHEx184J
xxqaXtykZz1ju98i/I4zcl1sxJ6mAnvGzBZ2fmyW5kzOWWDisstesO7PzEuJCGAG
DWClBBQhAkEA11eAx7cvIGREisMGo8EfG98pPBpGidDya4htmcGfPp1VrLYtvju6
9P7HDy4IyLrv1iAcXP0lfEDzfRV6rFhzswJAaysMxAij2JqgI2PaA54EstxSP04k
sGw119EEg99NAz6zMftUN0uufdLNaBX4nn0DL4u2a4W8QKIB1m528pe/QQJBAKt5
GCjwK2ylqxa7uZvH+ledWh5r5eN0KLWMC4o17fJUIpbG8qHaukLAZg4mYARHJxfg
tfUt9x18MudVpTt7q5UCQH53YfdJ1OON7RnFYVrlXKY5CK+PqJ/E1zykWSzRE12j
IkngPoZV3MkAuA9UU+vsUn6b9YYvqyT2ETKM1orRp+k=
-----END RSA PRIVATE KEY-----</pre>
1024 是密钥长度. 可以指定其他 (最小是 32). 一般都使用 1024bit. 从 pri.pem 的字面上看, 它保存的是私钥. 其实应该是密钥. 因为其中也包含了公钥. 用下面的命令可以将公钥提取出来:
<pre>D:\Temp>D:\Temp>openssl rsa -in pri.pem -pubout -out pub.pem
writing RSA key

D:\Temp>type pub.pem
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDK7TIBRai7Xq6dwU9IxbmvIh47
nt6VPZ+OwSJv32xTieGNS5TSFPF6q5piSkuqCalJPoyWn39+AzvgKRFLi+rCgeBy
Lu8ijPX/VgujLkKp/VPCaXx5Okf36HxxUbu6ODw0H8z8ARlnoXz+L2dT5XdhC/7P
DsodawCFCohbTjzmEwIDAQAB
-----END PUBLIC KEY-----</pre>
-pubout 的意思是输出公钥.
openssl rsa 还有一些参数可能要用到, 比如读取公钥中的 modulus
<pre>D:\Temp>openssl rsa -in pub.pem -pubin -modulus
Modulus=CAED320145A8BB5EAE9DC14F48C5B9AF221E3B9EDE953D9F8EC1226FDF6C5389E18D4B94
D214F17AAB9A624A4BAA09A9493E8C969F7F7E033BE029114B8BEAC281E0722EEF228CF5FF560BA3
2E42A9FD53C2697C793A47F7E87C7151BBBA383C341FCCFC011967A17CFE2F6753E577610BFECF0E
CA1D6B00850A885B4E3CE613
writing RSA key
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDK7TIBRai7Xq6dwU9IxbmvIh47
nt6VPZ+OwSJv32xTieGNS5TSFPF6q5piSkuqCalJPoyWn39+AzvgKRFLi+rCgeBy
Lu8ijPX/VgujLkKp/VPCaXx5Okf36HxxUbu6ODw0H8z8ARlnoXz+L2dT5XdhC/7P
DsodawCFCohbTjzmEwIDAQAB
-----END PUBLIC KEY-----</pre>
-pubin 是说 -in 参数后面的是公钥.
既然密钥对已经有了, 就可以加解密了
<pre>D:\Temp>echo openssl brief introduction by nicoster at gmail.com>src.txt
D:\Temp>openssl rsautl -inkey pub.pem -pubin -encrypt -in src.txt -out out.bin
D:\Temp>hexdump out.bin
HexDump - DiamondCS Freeware Console Tools (www.diamondcs.com.au)
---
Target: out (128 bytes)
00000000: 8D 02 A8 F1 BF CB DE C8 EB 39 D4 CB BE F8 5A F4  ?克奕?运绝Z?
00000010: 76 6B 5F 1A 1D 85 77 81 64 04 D6 D1 99 1B 8D AD  vk_..厀乨.盅?嵀
00000020: 51 F0 51 97 87 30 22 ED D3 9B 9B 7B DA 83 55 BF  Q餛棁0"碛洓{趦U?
00000030: DB 94 87 DA CF 21 5E 4C 41 F9 6A 7B CA 5D 2C 70  蹟囑?^LA鵭{蔧,p
00000040: 4B 7B 87 B9 02 BF 86 F3 01 CC 2F 7D 93 3B E8 37  K{嚬.繂??}??
00000050: 5C 8B 0F 59 66 A6 B3 80 36 74 E0 7F B3 E3 2E 5B  \?YfΤ€6t?炽.[
00000060: 59 9D 67 C6 AA 19 36 E4 52 7F 57 DF AC 9F 05 B2  Y漡篇.6銻W攥??
00000070: DB 53 ED 3F 63 09 F8 0E 39 08 63 C7 BF 43 68 F8  跾?c.?9.c强Ch</pre>
RSA 加密时明文的长度不能超过密钥长度, 例子中是 1024bit, 即 128 字节. 如果不要 padding, 最多也就是加密 128 字节. 如果要 padding 的话, 长度还要减少.
再试试解密:
<pre>D:\Temp>openssl rsautl -inkey pri.pem -decrypt -in out.bin
Loading 'screen' into random state - done
openssl brief introduction by nicoster at gmail.com</pre>
RSA 的签发/验证和加解密类似. 其实所谓的签发/验证就是用私钥加密/公钥解密
<pre>D:\Temp>openssl rsautl -inkey pri.pem -sign -in src.txt -out out.bin
Loading 'screen' into random state - done

D:\Temp>hexdump out.bin
HexDump - DiamondCS Freeware Console Tools (www.diamondcs.com.au)
---
Target: out.bin (128 bytes)
00000000: 37 85 7C 75 B2 BE 8B 94 41 03 08 C3 83 D6 7D CF  7厊u簿嫈A..脙謢?
00000010: 4B B2 54 01 5F D1 6A 27 96 15 90 F2 17 B8 A7 F3  K睺._裫'?愹.抚?
00000020: 23 22 1B 75 4F F4 FC F9 09 A6 78 AC 40 D9 A8 56  #".uO酎?珸侉V
00000030: 5A 81 55 36 DF 72 9C 73 66 D9 90 57 E0 92 7F 54  Z乁6遰渟f賽W鄴T
00000040: 21 7A A2 2B 31 05 71 30 62 3D 91 EF 9F 28 3E 3B  !z?1.q0b=戯?>;
00000050: 37 26 D5 C0 6F 86 48 58 33 F6 9E DD DE D3 7A 34  7&绽o咹X3鰹蒉觶4
00000060: 58 14 44 93 AC E0 7D 71 10 EA 74 A3 35 B6 5C 22  X.D摤鄛q.阾?禱"
00000070: DA B2 93 56 47 5D 76 F7 24 4D 43 AF 18 1A 94 02  诓揤G]v?MC?.?

D:\Temp>openssl rsautl -inkey pub.pem -pubin -verify -in out.bin
Loading 'screen' into random state - done
openssl brief introduction by nicoster at gmail.com</pre>
<h4 id="section-OpenSSL+Command-Line+Practice-AES_E7_9A_84_E7_9B_B8_E5_85_B3_E6_93_8D_E4_BD_9C">AES 的相关操作</h4>
openssl enc 命令涵盖了很多对称加密算法. 这里我只拿 AES 做例子.
<pre>D:\Temp>openssl enc -K 12345678901234567890123456789012 -iv 12345678901234567890
123456789012 -in src.txt -e -aes-128-cbc -out out.bin

D:\Temp>hexdump out.bin
HexDump - DiamondCS Freeware Console Tools (www.diamondcs.com.au)
---
Target: out.bin (64 bytes)
00000000: 3A 77 71 12 9E 51 2F A0 C4 D0 FF C5 B1 8C F8 75  :wq.濹/犇?疟岠u
00000010: 01 32 8C 7D B3 F1 C4 B1 4F FB AC 23 64 CD 88 B7  .2寎绸谋O#d蛨?
00000020: 24 36 8C 3F B7 18 EC AC CA D1 6D E3 66 68 35 37  $6??飕恃m鉬h57
00000030: 45 9A A2 29 8E BB 9D CD 89 EE 79 4A A1 D8 5F C8  E殺)幓澩夘yJ∝_

D:\Temp>openssl enc -K 12345678901234567890123456789012 -iv 12345678901234567890
123456789012 -d -aes-128-cbc -in out.bin
openssl brief introduction by nicoster at gmail.com</pre>
<h4 id="section-OpenSSL+Command-Line+Practice-MISC">MISC</h4>
base64 操作
<pre>D:\Temp>openssl base64 -in src.txt
b3BlbnNzbCBicmllZiBpbnRyb2R1Y3Rpb24gYnkgbmljb3N0ZXIgYXQgZ21haWwu
Y29tDQo=

D:\Temp>openssl base64 -in src.txt |openssl base64 -d
openssl brief introduction by nicoster at gmail.com</pre>
sha256/sha1/md5
<pre>caroot$openssl dgst -sha256 src.txt
SHA256(src.txt)= 158a4355d98b57889569ab9eb266489f399d1eca070a2f74eea5123532708556
caroot$openssl dgst -sha1 src.txt
SHA1(src.txt)= 660c1af9339e3b685bf03d1efd2fc77bfb9ba3b2
caroot$openssl dgst -md5 src.txt
MD5(src.txt)= 0968fea2e8ea89d4da5675b514fb7d7e</pre>
 
