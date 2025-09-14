---
layout:     post
title:      "记一次网络troubleshooting"
subtitle:   "有被自己糖到"
date:       2025-9-14
author:     "tofucurd"
header-img: "img/post-bg-1.jpg"
catalog: true
mathjax: true
tags: [macOS, network]
---

## 问题

忽然发现mac连接``eduroam``或者有线时均无法访问任何zju网站，但是其他网站一切正常。

## 解决过程

关闭所有VPN，未解决。

怀疑zju网站是不是炸了，很快通过iphone排除。

尝试使用iphone热点功能绕开校园网，发现可以访问。

使用``ping``及``traceroute``跟包，发现无法到达目标服务器，包在中途``hop=5``时超时。

此时怀疑DNS在解析网址``learn.intl.zju.edu.cn``出错，查看DNS servers list, 仅有``8.8.8.8``。

使用``ping``得到DNS解析结果为``210.32.4.165``，同时google该网址的ip，发现吻合，同时得知该ip位于玉泉校区。

因此问题出在DNS server错误地将网址解析为了玉泉校区的机房，而非海宁，故手动配置DNS server为router ip，解决。

打开CFW，发现又不行了，将``*.zju.*``加入CFW的bypass名单中，依旧不行。

查看CFW的log发现即使是direct mode，``learn.intl.zju.edu.cn``依然被解析成了上述的``210.32.4.165``，连忙查看NDS servers list，发现已被篡改。

查看CFW文档，发现Mixin与TUN mode选项均会改变DNS servers list，故关闭上述选项，重新写入router ip，解决。

此时使用``traceroute``跟包，发现已经正确解析为了海宁校区内网ip，输出如下。

```
$ traceroute learn.intl.zju.edu.cn
traceroute to learn.intl.zju.edu.cn (10.105.7.35), 64 hops max, 40 byte packets
 1  rt-be57-0754 (192.168.2.1)  3.799 ms  2.470 ms  2.236 ms # 这个是我房间里自己的router，使用网线连接校园网
 2  10.106.93.1 (10.106.93.1)  3.997 ms  17.770 ms  4.683 ms
 3  10.105.7.35 (10.105.7.35)  6.075 ms  3.605 ms  4.228 ms
```

## 后记

这么点破烂事我花了一下午时间，从4点到6点半，看来本人水平有待提高啊（笑）。