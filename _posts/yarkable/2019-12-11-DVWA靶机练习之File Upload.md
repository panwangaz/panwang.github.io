---
layout:     post
title:      DVWA靶机练习之File Upload
subtitle:   
date:       2019-12-11
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



这是 DVWA 靶场练习系列的第三篇，这次的内容是文件上传漏洞，这里是针对 php 的漏洞，文件上传漏洞就是指程序员没有对用户上传的文件做检查，导致用户上传了恶意脚本即 webshell，来获取敏感信息甚至提取服务器权限



>  文件上传漏洞的三个必要条件是：
>
> 1. 可上传木马文件
>
> 2. 文件能被执行
> 3. 知道上传文件路径

缺了一个都不能执行上传漏洞



## low



直接上传一个 php 一句话上去，回显出了文件路径



![php-shell](https://i.loli.net/2020/04/11/7RiPjEZY8xGzh2c.png)



拿蚁剑一连，OK，搞定

![ant-sword](https://i.loli.net/2020/04/11/UQLmVOhalWrvqPT.png)



## medium



中级关卡再上传 php 一句话就提示只能提交图片格式的文件了

```txt
Your image was not uploaded. We can only accept JPEG or PNG images
```

猜测是进行了 MIME 校验，于是上传一句话，然后用 Burp 抓包，修改 MIME 为 `image/jpeg`，成功上传，蚁剑连起来



![](https://i.loli.net/2020/04/11/OCY3ViAz4S6nZcR.png)



![php-shell](https://i.loli.net/2020/04/11/7RiPjEZY8xGzh2c.png)



### 第二种方法



可以用组合拳，因为这套靶机还有文件包含漏洞，因此可以用文件包含漏洞结合文件上传漏洞进行攻击，因为文件包含是将包含的文件都按照 php 来执行，所以我们这里可以将一句话的后缀改成 jpg，然后用文件包含漏洞去执行



![file-inclusion](https://i.loli.net/2020/04/11/gNxvmMryO9SUoiC.png)



在蚁剑 url 处填上文件包含的地址，由于是 medium 级别的文件包含，所以要双写 http 绕过

```txt
http://localhost/dvwa/vulnerabilities/fi/?page=..././..././hackable/uploads/php-one-word.jpg
```



**但是，由于 DVWA 靶机是需要登录的，所以需要 cookie 进行身份认证，否则会重定向到登陆页面，拿不到 shell，因此可以先保存网站数据再右键浏览网站，保存相应的 cookie，这样才能成功 getshell**

![sword](https://i.loli.net/2020/04/11/jzgV1vGPZYEtLHh.png)



## high



high 级别的上传漏洞，像 medium 一样修改了 MIME 之后还是说格式错误，只接受图片文件，所以猜想它应该是检查了图片的内容，所以直接在一张真正的图片后面加入我们的一句话木马，用十六进制编辑器来搞张图片马就行了



![hex](https://i.loli.net/2020/04/11/XKUVo6QFSh7luAZ.png)



可以直接上传，然后配合文件包含漏洞执行代码，同理也是要在蚁剑里面设置 cookie 才能连上去

```txt
http://localhost/dvwa/vulnerabilities/fi/?page=file:///F:/nginx-1.16.1/html/dvwa/hackable/uploads/4.jpg
```

关于文件包含漏洞看我之前发的 DVWA 的文件包含漏洞教程