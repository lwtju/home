---
layout: post
title: 记一次Jenkins部署错误解决过程
key: 20161110
tags: Jenkins
---

## 问题表现

提示错误：RPC failed: result=18, HTTP code = 200

此外还有一个表象，有的job能工作，有的job不能工作

![jenkins_error](/home/assets/images/jenkins_error.png "问题表现截图")


## 解决过程

1、最初以为是授信问题，尝试将https用户名密码登录方式改为ssh方式，无效，除了上面截图，未能在jenkins上找到其他日志

2、然后查看git服务，开始被unicorn.stderr.log 的一个日志误导了，以为是内存不够

![uncorn_stderr_1](/home/assets/images/uncorn_stderr_1.png "uncorn_stderr_1.log")

然而这个错误并不是致命错误，然后又返回查看jenkins

3、在jenkins的屏幕输出中发现了这条 fatal: The remote end hung up unexpectedly 这是远程断开了，应该先从宏观上查问题，jenkins和gitlab之间还有一层nginx，发现nginx里有个log

![jenkins_nginx_error_1](/home/assets/images/jenkins_nginx_error_1.png "jenkins_nginx_error_1.log")

upstream closed, 说明git服务多半超时了

3、再次查看unicorn.stderr.log 发现了这条

![uncorn_stderr_2](/home/assets/images/uncorn_stderr_2.png "uncorn_stderr_2.log")

基本是配置问题了，打开unicorn.rb发现了这个时间设置 timeout 60
改成120，还是有错，改成240，成功执行

## 总结

一开始解决问题很容易钻入某个错误细节，把数据走向经过的结构搞清，从宏观上查看中间log，更容易顺藤摸瓜
