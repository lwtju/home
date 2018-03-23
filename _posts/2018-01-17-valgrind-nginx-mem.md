---
layout: post
title: 用valgrind查找nginx内C代码写越界问题
key: 20180117
tags: valgrind nginx memory
---

## 问题表现

公司内的一个打点服务用的是nginx+lua实现的，其中解密代码用C实现。测试发现nginx会自动重启worker进程，并在重启一定次数后会直接不再重启。也会有一些进程不退出，但无法接受新的链接。


## 解决过程

1、观察error log能发现worker进程会退出,调试模式定位到了一个解密so的问题

2、valgrind安装就不再赘述了，安装好之后执行命令   
valgrind --tool=memcheck --leak-check=full --track-origins=yes --show-reachable=yes /usr/local/nginx/sbin/nginx -c /home/q/system/nginx.conf

![valgrind_nginx_mem_1](/home/assets/images/valgrind_nginx_mem_1.jpg "valgrind_nginx_mem_1.jpg")

3、检查源代码也会有相应的发现

![valgrind_nginx_mem_2](/home/assets/images/valgrind_nginx_mem_2.png "valgrind_nginx_mem_2.png")