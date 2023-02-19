---
layout:     post
title:      "RHEL中yum无法启用repo的解决方案"
subtitle:   "白嫖系统"
date:       2022-2-19
author:     "tofucurd"
header-img: "img/post-bg-1.jpg"
catalog: true
mathjax: true
tags: [Linux, yum, RHEL]
---

安装完RHEL 9.1后，使用``sudo yum update``时发现``There is no enabled repos in "/etc/yum.repos.d"``，下面是解决方案：

网上搜到的几片Blog都是用的CentOS的repo替换，其实现在RHEL已经可以对个人开发者免费了，所以第一步时注册Red Hat账号并申请通过Red Hat开发者订阅（即白嫖），并使用如下命令注册：
```sh
sudo subscription-manager register
sudo subscription-manager refresh
```
过程中发现
```sh
System certificates corrupted. Please reregister
```
``refresh``失败，搜索后发现是``/etc/kpi/product``中文件缺失，从``https://access.redhat.com/labs/rhpc/``上下载``Red_Hat_Product_Certificates.sh``并执行即可成功注册刷新，之后可以正常使用yum。