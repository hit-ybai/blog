---
layout: post
title: "千万不要sudo brew install"
date: 2015-01-09 09:34:22 +0800
comments: true
categories: 开发工具
tags:
- activemq
- homebrew
---
近些天来一直受到一个问题的困扰，在执行`activemq start`后，能够看到进程已经启动，然而根本没有什么卵用，完全连不上`localhost:8161/admin/`。改成`sudo activemq start`，问题就解决了。

今天终于发现了问题的原因：当初我在用`brew`安装`activemq`的时候就是用的是：
```bash
$ sudo brew activemq
```
而正确的安装方式是：
```bash
$ brew activemq
```
解决方案：
```bash
$ brew reinstall activemq
```
如果再这么做的话，这个世界，就清静了：
```bash
$ brew reinstall `brew list`
```