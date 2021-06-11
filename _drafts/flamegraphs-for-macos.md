---
layout: post
title: DTrace, FlameGraphs for macOS
---

相信大家都知道火焰图了, 不过一般都是在 linux 上使用 systemtap 来抓取调用栈, 然后生成火焰图. 很少有文档说 mac 上怎么生成火焰图. 下面提供了几种方法

## 使用 DTrace

```shell
DURATION=${2:-35}
LOGFILE=/tmp/$DURATION-`date +%Y%m%d-%H%M%S`

sudo dtrace -x ustackframes=100 -n 'profile-997 /pid == $target/ { @[ustack()] = count(); } tick-'${DURATION}'s { exit(0); }' -p $1 -o $LOGFILE.log && \
flamegraph.pl > $LOGFILE.svg && \
open $LOGFILE.svg
```

上面的命令使用 profile 这个 provider, profile-997 表示每秒采样 997 次. `/pid == $target/` 是 predicate, 表示只关注 $target 这个进程, 也就是在命令行用 -p 指定的进程 ID. action 是 `{ @[ustack()] = count(); }`, 意思是按调用栈汇总. 

这样就能生成这么一样火焰图

{figure 1}


## 使用 sample

macOS 自带了一个 /usr/bin/sample 命令. Activity Monitor 可以对一个进程进行采样, 就是用的这个命令.

{figure 2}

```shell
DURATION=${2:-35}
LOGFILE=/tmp/$DURATION-`date +%Y%m%d-%H%M%S`

sample $1 $DURATION -f $LOGFILE.log && stackcollapse-sample.awk $LOGFILE.log | \
flamegraph.pl > $LOGFILE.svg && \
open $LOGFILE.svg
```

脚本比较简单. 不过里面的 stackcollapse-sample 做了一点修改. 

## Reference
* https://www.joyent.com/blog/bruning-questions-debugging
* https://www.oracle.com/technetwork/server-storage/solaris/dtrace-cc-138561.html
