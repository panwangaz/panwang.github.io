---
layout:     post
title:      ubuntu 配置 OpenCV 和 OpenCV_contrib
subtitle:   TODO
date:       2019-11-04
author:     kevin
header-img: img/green-bg.jpg
catalog: true
tags:
    - linux
    - OpenCV
---



## preface



最近的学习涉及到 KCF 追踪算法，然而在我的 OpenCV 中找不到 KCF 的头文件，查阅资料发现还需要安装 OpenCV_contrib 这个模块，但又不想重装我的 OpenCV，于是就在我的 WSL(ubuntu18.04) 里面重新装一个 OpenCV，顺便记录一下坑，以防再掉进去



## 下载 OpenCV



我之前一直用的是 OpenCV3.4.4 版本，本想尝尝 OpenCV4.1.2 ，但是网速不太好，不想下载了，所以还是用老版本的，至于 OpenCV_conrib ，也是直接去官网 git clone 下来，也才 80+Mb 所以很快，然后我们需要将 OpenCV_contrib 的版本切换成跟我们的 OpenCV 一样，用 `git checkout` 命令



![opencv-1.jpg](https://i.loli.net/2019/11/04/mBFkd2uOZgJnacY.jpg)



## 安装依赖



首先要装一堆依赖，不然的话之后的 cmake 过程中会报一堆奇怪的错误，更狗的是，可能 OpenCV 编译好了，用的时候发现有些模块用不了，这就是缺少依赖库的原因，所以，一定要在编译之前下载好依赖库，由于我的 WSL 基本只用来做 C++/Python ，所以要下载好多依赖，大概花了十分钟吧



```bash
$ sudo apt-get install build-essential
$ sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
$ (optional)sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev
```



## cmake 编译



我们先进入 OpenCV 的目录，新建一个文件夹名叫 build (其实叫啥都行)，这个文件夹是我们用来装 cmake 编译文件的，目的就是不跟源文件掺杂，然后我们进入这个 build 文件夹



```bash
$ cd opencv-3.4.4
$ mkdir build
$ cd build
```



下面就到了 cmake 编译环节，这里加了很多编译选项，官网里面都有[解释](https://docs.opencv.org/3.4.4/d7/d9f/tutorial_linux_install.html)，之前的话我就直接用 `cmake ..` 也是可以的，这里重要的就是第五个编译选项要找到 OpenCV_contrib 中 module 的路径，并且这里也对 python 环境做了编译。命令行输入这段代码后就等待 `Makefile` 生成了!



```cmake
cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr/local \
-D INSTALL_PYTHON_EXAMPLES=ON \
-D INSTALL_C_EXAMPLES=ON \
-D OPENCV_EXTRA_MODULES_PATH=../opencv_contrib/modules \
-D PYTHON_EXECUTABLE=/usr/bin/python3 \
-D BUILD_EXAMPLES=ON ..
```



当看到这段文字的时候就说明 cmake 编译通过，已经生成了 Makefile 文件



![opencv-2.jpg](https://i.loli.net/2019/11/04/3jWl4gVi15YtF7L.jpg)



## make



cmake 完毕之后，我们就按照 Makefile 中的规则进行编译，执行 make 操作，-j 选项使用电脑所有的线程进行编译



```bash
$ make -j
```



？？？去他 md，这 cmake 占用我 CPU 100% ，内存最后也没了，导致我电脑重启，再来 make 一次，过了几十分钟直接报错了，再见！等爷换个牛逼的电脑再来更新教程！



![opencv-3.jpg](https://i.loli.net/2019/11/04/tcJ7KsVA9Ilyojn.jpg)



勿念！！



![opencv-4.jpg](https://i.loli.net/2019/11/04/Z1ztcnBC42AWliY.jpg)



> 2020.05.16 我又回来了，假设上面 make 的步骤已经成功了，接下来就要将 OpenCV 安装到系统里面了



## make install



make 之后，生成了可执行的文件，如果源码编译没有问题的话，便将程序安装至系统预设的可执行文件存放路径，在 Makefile 里面指定。用下面这个命令进行安装（要有 sudo 权限，因为这是向系统里面写文件）



```bash
$ sudo make install
```



## 配置链接库



程序运行时加载动态链接库可以通过 `ldconfig` 来执行，这玩意是什么东西呢？程序运行的时候可能需要动态的链接库，主要是在默认搜寻目录 `/lib` 和 `/usr/lib` 以及动态库配置文件 `/etc/ld.so.conf` 内所列的目录下, 
搜索出可共享的动态链接库(.so 后缀文件)。因此一般的做法就是在 `/etc/ld.so.conf` 下新建一个文件叫做 `opencv.conf`，往里面写上动态链接库的路径，再通过 `ldconfig` 命令使配置的路径生效。



```bash
$ sudo vim /etc/ld.so.conf.d/opencv.conf
```



往里面写上 `/usr/local/lib ` ，表示去 `/usr/local/lib ` 这个目录寻找 OpenCV 的动态链接库（make install 时将动态链接库安装在此处）



最后执行 `ldconfig` 命令使修改生效，下次程序运行时，会自动在 `/usr.local/lib` 目录中搜索动态库

```bash
$ sudo ldconfig
```



到此，ubuntu 下的 OpenCV 库的配置就结束了，开始调包吧