---
layout: post
title: php问题汇总
key: 20150810
tags: php
---

1. 用php执行exec要在结尾加上 2>&1   否则错误信息可能打不出来 redirect the error msg to standard output  
2. php-fpm.conf内有控制系统环境变量的参数 clear_env ，此参数在 5.4.27之后默认开启，会导致php命令行执行脚本获取不到系统的环境变量
3. 