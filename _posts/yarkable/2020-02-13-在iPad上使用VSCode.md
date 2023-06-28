---
layout: post
title: 在iPad上使用 VSCode
subtitle: 
date: 2020-02-13
author: kevin
header-img: img/green-bg.jpg
catalog: true
tags:
    - linux
---



## preface



以往的几年，放寒假回家我都会把电脑带回家，今年不想背这么重的电脑回家，而且在回家之前给学校的服务器做了内网穿透，回家后可以用 iPad 进行远程访问，想着反正也就十几天的假期，应该不会做太多的事，就把电脑留在了学校，谁知道就赶上了武汉的肺炎，这下要四月才能到学校，本来是 2 月 1号去学校备赛的，现在也只能在家里云备赛，烦得很。主要敲不了代码很恶心，虽然我已经在学校服务器上部署了 jupyter notebook，还能够打打 python 。



![](https://i.loli.net/2020/03/15/irh2gDElqvTmRkd.jpg)



但是偶尔想复习下 JavaScript ，在 iPad 上硬是找不到合适的集成环境，顶多只有编辑器，而且还不带自动补全的那种，Google 搜索一下，发现可以在服务器上云部署类似 VSCode 的 code-server ，然后在浏览器中通过端口访问服务，听起来好像可以冲一波，刚好手上有台阿里云服务器可以折腾一下。



## 安装 code-server



code-server 的官方 GitHub 地址[在这里](https://github.com/cdr/code-server)，目前的 star 数有两万七千多，是个挺优质的项目，我们直接用下面命令将软件包下载到服务器上



```bash
$ wget https://github.com/cdr/code-server/releases/download/2.1698/code-server2.1698-vsc1.41.1-linux-x86_64.tar.gz
```



![](https://i.loli.net/2020/03/15/qveH7XOADR2Tgn1.jpg)




由于 iPad 上的代理不是全局的，所以下载 GitHub 的包速度特别慢，我就用浏览器下载到 iPad 本地再通过 ftp 软件上传到服务器上。之后，将压缩包解压：
```bash
$ tar zxvf code-server2.1698-vsc1.41.1-linux-x86_64.tar.gz
```


文件夹里面有个叫 `code-server` 的可执行文件，为了以防权限问题，我就将整个文件夹的权限都赋予 `755`

```bash
$ sudo chmod 755 code-server2.1698-vsc1.41.1-linux-x86_64
```



然后参照一下文档，我们输入下面命令来看看有哪些参数是可选的

```bash
$ ./code-server —help
```

![](https://i.loli.net/2020/03/15/QhWKSBeg6sxz5kV.jpg)



对我来说，主要的就是 —auth，—port 这两个参数，—auth 是认证密码，有两个选项，一个是 none，一个是 password ，其中 password 需要从服务器上导入环境变量，默认情况下会开启密码，密码是个随机字符串，可以在终端找到。—port 是端口，决定了用哪个程序开启端口，默认是 8080 端口，想要指定其他端口得去服务器的防火墙处开放端口，因此，基本的开启服务的命令就是下面这几句

```bash
$ export PASSWORD=xxxxxx
$ ./code-server —auth password —port xxxx
```



然后服务器上就完成运行了，下面只需要打开浏览器，输入服务器的 ip+port ，设置了密码的话就输入密码

![](https://i.loli.net/2020/03/15/gxom5Edyw4Bbs7e.jpg)

认证身份之后就可以访问 code-server 了，还是那么熟悉的界面，接下去的操作就跟 VSCode remote 一样的，code-server 可以访问到服务器上所有的文件，执行程序就是用服务器上的应用程序来运行我们自己写的代码，例如我想在服务器跑一个 js 文件，那么我只要在我的服务器上下载 nodejs 就可以了，跑其他代码也是同理。还可以新建终端，debug，简直就跟 VSCode 一模一样



![](https://i.loli.net/2020/03/15/GuPqA1tImEfWleR.jpg)



## 关闭 code-server



我运行的时候并没有加上 nobup 命令，但是退出了 ssh 连接之后服务也并没有被关闭，导致我重新执行的时候说我的端口被占用了，这里就记录一下怎样查询端口号对应的应用程序并将程序关闭。



我们可以用 `lsof -i:port` 来查看端口对应的进程，如果有端口占用的话就会在下面显示出其进程号（PID）然后用 `kill PID` 来杀死进程，但是这一回用 kill 还不行，杀不死，然后用 `kill -9 PID` 才成功杀死进程，总之一个不行的话就用另一个。



![](https://i.loli.net/2020/03/15/FaRKCtiwDGVs5f2.jpg)



## misc



插件市场可以找到 VSCode 拥有的插件，但是用 code-server 的话下载不下来，我已经科学上网了，还是不行，but，这玩意也像 VSCode 一样可以在官网下载相应的插件安装包然后直接本地安装，由于我只是轻量使用，并不需要太多插件，而且插件太多也会造成卡顿现象，所以我就没有安装插件了。



如果退出 ssh 连接之后 code-server 服务关闭的话，建议直接用 nohup 来后台运行 code-server，或者也可以用 systemd 将 code-server 写成一个服务来启动，这个我就不折腾了，有兴趣的可以参考 reference 中的链接



最后展示下 coding 效果图以表达 VSCode 真香 ：）

![](https://i.loli.net/2020/03/15/yquJhEVOsZWi2gF.jpg)



## reference 

https://blog.lopygo.com/2019/p-vscode-code-server-test/
