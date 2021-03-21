---
title: 如何加速Github访问速度
author: Hank
categories: 技巧
tags:
  - GitHub
  - 加速
abbrlink: d480
date: 2020-07-22 15:46:00
---

最近比较烦时常抽风的GitHub，搞得我不能愉快的~~克隆~~借鉴代码，再加上之前一直在用的鸡场场长跑路了，所以尝试改hosts来实现。

## 首先把hosts放出来

路径是

```
C:\Windows\System32\drivers\etc\hosts
```

你可以把它复制到桌面上，增加三行，再覆盖原文件。

```
140.82.112.3                github.com
185.199.108.153             assets-cdn.github.com
199.232.69.194              github.global.ssl.fastly.net
```

当然这只是我的配置，你可以拿去用但是我不知道能不能奏效。

如果效果不好，你可以自己在[ipaddress](https://www.ipaddress.com/)上查找对应域名的ip然后进行更改。

值得注意的是第二个加速域名有多个ip，任选一个即可。

## 到这里还没完

打开CMD，输入

```
ipconfig /flushdns
```

回车后执行刷新本地DNS缓存数据。

试试看访问我的[个人主页](https://github.com/Cliccker)，如果能快速加载出来就说明奏效了。

## 但是

访问速度是快了不少，下载还是龟速怎么办？

一种方法是在码云上新建一个仓库，然后把Repo搬过来下载，个人觉得这样做有点麻烦。

另一种方法比较推荐，就是利用文件代下载服务，如https://shrill-pond-3e81.hunsh.workers.dev/

![加速下载](https://my-picbed.oss-cn-hangzhou.aliyuncs.com/img/20200722162047.png)

感谢热衷于分享的程序⚪们！