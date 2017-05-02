---
layout: post
title: "从Octopress迁移到Hexo"
date: 2015-06-13 14:55:27 +0800
comments: true
categories: 技术日志
tags:
- hexo
- octopress
- gitcafe
---
前阵子有幸地参与到一个 Road Map Project 从无到有的开发过程，整个项目做下来，感触很多。随着项目进入收官阶段，可以自由支配的时间渐渐多一些，准备有规律地更新博文。

![难得一见的北京蓝](http://7xjra1.com1.z0.glb.clouddn.com/galaxy_soho_sky.jpg)

这次正式从`Octopress`迁移到`Hexo`，个人感觉`Hexo`更加的 Simple Stupid，从Setup到Deploy的过程相当流畅。

<del>关于Hexo的部署，国内朋友的最佳实践是同时发布在`Github`和`Gitcafe`上，之后通过`DNSpod`分散国内外的流量，保证访问速度。感兴趣的同学可以去读一下Reference中的两篇文章。</del>

<del>我个人比较懒，并没有通过DNS来分流，只是单纯地在`Github`上备份`source`文件夹，将其软链到`Hexo`中。然后在`Gitcafe`上托管`Blog Site`。</del>

<lable style ="color: red">更新于：2015.6.23</lable>
现在还是部署回Github，因为Gitcafe常常出现无法访问的情况。

## Reference
[Hexo Landscape主题的字体和JS库优化](http://kuangqi.me/tricks/hexo-optimizations-for-mainland-china/)
[Hexo 同时支持Github和Gitcafe](http://colobu.com/2014/10/13/hexo-supports-both-github-and-gitcafe/)