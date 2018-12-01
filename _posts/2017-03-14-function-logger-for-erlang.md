---
id: 725
title: Function Logger for Erlang
date: 2017-03-14T00:10:02+00:00
author: nickr
layout: post
tags:
    - function logger
    - parse_transform
    - decorater
---

调试 erlang 的代码, 基本上还是靠打 log. 在一个函数进入或退出时打印参数和返回值, 是一个有效的调试手段. 如果是用 c++, 实现这样一个 function logger 可以做的很简单. 但 erlang 里没有对象, 构造, 析构这样的东西, 得另想办法. 还好我们有 parse_transform. 

parse_transform 说到底, 就是 erlang 编译器将源码转换成 parse tree 的时候, 给用户一个机会修改它. 这个 parse tree 是这样的形式:

```erlang
[{attribute,1,file,
     {"/workspace/erl-decorator-pt/_build/default/deps/decorator_pt/src/main.erl",
      1}},
 {attribute,9,module,main},
 {attribute,10,author,"nickx"},
 {function,24,foo,6,
     [{clause,24,
          [{var,24,'Name'},
           {var,24,'A'},
           {var,24,'B'},
           {var,24,'C'},
           {var,24,'E'},
           {var,24,'N'}],
          [],
          [{'case',25,
               {op,25,'>',{var,25,'N'},{integer,25,0}},
               [{clause,26,
                    [{atom,26,true}],
                    [],
                    [{call,27,
                         {atom,27,foo},
                         [{var,27,'Name'},
                          {var,27,'A'},
                          {var,27,'B'},
                          {var,27,'C'},
                          {var,27,'E'},
                          {op,27,'-',{var,27,'N'},{integer,27,1}}]}]},
                {clause,28,[{var,28,'_'}],[],[{atom,28,ok}]}]}]}]},
 {function,32,bar,0,[{clause,32,[],[],[{atom,32,ok}]}]},
 {eof,33}]
```

它对应的源文件为:

```erlang
-file("/workspace/erl-decorator-pt/_build/default/deps/decorator_pt/src/main.erl",
      1).
-module(main).
-author("nickx").
foo(Name, A, B, C, E, N) ->
    case N > 0 of
        true ->
            foo(Name, A, B, C, E, N - 1);
        _ ->
            ok
    end.
bar() ->
    ok.
```

所有的 directives 比如 -module, -author 在 parse tree 中都有一个对应 的 {attribute, ..} 的 tuple. 甚至你自己写一个 -foo, 都会有对应的 {attribute, LINE, foo, ..}, 只是编译器的 backend 无法识别这个 attribute 最终会报错. 正是利用这个 attribute, 让我们可以实现 function logger.

function logger 的思路很简单. 就是 hook 一下感兴趣的函数, 先打印参数, 然后执行原函数, 再把返回值打印出来即可. 转念一想, hook 的作用不仅于此, 还可以有其他的应用. 所以, 用 parse_transform 实现一个通用的 hook 机制就更有价值. 联想到 python 的 decorator, 可以考虑实现一个 erlang 的 decorator. google 之, 早就有人有同样想法. 其中的一个: [erl-decorator-pt](https://github.com/spilgames/erl-decorator-pt) . 基于这个实现稍作修改, 加上 function logger 的代码就完成了. [查看源码](https://github.com/nicoster/erl-decorator-pt)

使用起来很简单, 如下所示:

```erlang
-include("func_logger.hrl").

?funclog("2:5").
foo(Name, A, B, C, E, N) ->
    case N > 0 of
        true ->
            foo(Name, A, B, C, E, N - 1);
        _ -> ok
    end.
```

其中的 "2:5" 表示只打印 B, C, E, N 这 4 个参数. 如果打印全部参数则指定 ":" 即可. 要解析这个字符串, 可以在每次打印 log 的时候做, 不过这样的话, 就势必引入不必要的 runtime penalty, 这里想到了可以用 erlang 大神 Ulf Wiger 实现的 ct_expand. 它可以在编译时执行函数将其结果嵌入到 parse tree. 

parse_transform 功能非常强大, 比如上面的 git repo 里还实现了一个 -export_func, 它的作用是, 你可以在要导出的函数头部写上这个 -export_func, 这个函数就导出了. 不需要加入到文件头部的 export list 中. 



