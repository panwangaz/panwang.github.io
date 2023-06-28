---
layout:     post
title:      DVWA靶机练习之Command Injection
subtitle:   
date:       2019-12-09
author:     kevin
header-img: img/green-bg.jpg
catalog: true
tags:
    - security
    - web
    - php
    - ctf
---



## preface



这是 DVWA 靶场练习系列的第二篇，这次的内容是命令执行注入，常见于需要调用系统命令的输入框，因为编程语言一般都有调用系统命令的接口，因此如果用户输入了一些恶意代码可能就能获取到系统的敏感信息



## 低级



这里有一个输入框，根据题目意思就是输入一个 ip 地址，来测试主机与该地址是否能够进行通信，说实话，这种一看就是命令注入的，因为 `ping` 就是一个系统命令，因此，如果没做保护很容易通过命令的连接执行其他的命令



![ping.jpg](https://i.loli.net/2019/12/14/athNcYFRolDwVbj.jpg)



这里我们就 ping 一下本地环回，然后在后面加上 `&&` 连接注入的命令 `whoami` ，这就造成了一次命令注入攻击

```
127.0.0.1 && whoami
```



![low.jpg](https://i.loli.net/2019/12/14/vOM3U1PaBhk4uZL.jpg)



我们看一下后端的代码，可以说，这是直接获取用户输入并没有做任何过滤，所以非常危险



![low-code.jpg](https://i.loli.net/2019/12/14/hmdIOJFBAQMu9V4.jpg)



## 中级



中级的例子我们还是用低级的 payload 测试，就不能通过了，这下肯定是有了过滤的



![error](https://i.loli.net/2019/12/14/lvS4ugzhyDctFPW.jpg)



查看一下后端的代码，对 `&&` 和 `;` 进行了过滤，也就是说我们不能用这两个符号来连接命令，不过还是没有过滤完全，还是有很多的方法可以绕过



![median-code](https://i.loli.net/2019/12/14/b4Pmao2ExUMsufF.jpg)



比如我们还可以用 `&` 来绕过，这在 Linux 下表示在后台执行命令，在 Windows 下还是表示将两个命令顺序执行。也可以用管道命令 `|` 将前一个命令的标准输出当作后面一个命令的标准输入。还可以针对程序将这两个东西过滤成空格这一特点来制造新的 payload，比如 `&&&` 和 `&;&` 被过滤之后都是 `&&`，因此有很多种构造的方法



## 高级



高级的有点狠啊，中级的那些绕过的全部都不能用了，试了好些都不可以，然后看看源码做了什么过滤



![high-code.jpg](https://i.loli.net/2019/12/15/l9oXByuSc1KUF2C.jpg)



这整个就把很多命令都给过滤了，不能用管道，不能用命令拼接，真的不知道该怎么绕过，去看了网上的解答，太坑了！这里过滤的是 `'| '` ，后面有个空格，所以我们可以在左边加上空格，这样就可以绕过，不得不说，搞 ctf 真的要看眼力，这样都行，我服了，所以最终的 payload 就是下面这个，想到的话还是挺简单的

```
127.0.0.1 |whoami
```



## 不可能



![impossible-code.jpg](https://i.loli.net/2019/12/15/9cCwltN2nVhYbIL.jpg)



这里他验证了 token 来过滤请求，并且直接限制了输入必须是以 `.` 相隔开的 ip 地址，这样就没办法注入命令了，因此，在过滤的时候最好就是使用白名单，不要使用黑名单

