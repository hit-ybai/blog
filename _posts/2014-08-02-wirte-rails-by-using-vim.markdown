---
layout: post
title: "Rails开发常用Vim插件"
date: 2014-08-02 08:21:31 +0800
comments: true
categories: 开发工具
tags:
- vim
- rails
---
看[RailsCast China](http://Railscasts-china.com)的时候，收集了一些常用来写Ruby on Rails的Vim插件:

* [commandT](http://www.vim.org/scripts/script.php?script_id=3025)
功能: 与Sublime的commandT快捷键类似，可以实现当前项目文件查找
* [CtrlP](http://kien.github.com/ctrlp.vim/)
功能: 实现文件的查找，并快速切换；
“CommandT”和”CtrlP”的功能基本一致，顾名思义，通过快捷键”command+T”和”Ctrl+P”，呼叫插件。

* [Rails.vim](http://www.vim.org/scripts/script.php?script_id=1567)
常用功能:
“:Rfind/:1R/:find”: 可以用来查找目录中的文件
“:Rfunctionaltest”: 跳转到单元测试
“:RController”: 跳转到Controller
“:RVfunctionaltest”: (横屏分屏)打开单元测试
“gf”: 代码上线文相关跳转
详见文档

* [snipMate](http://www.vim.org/scripts/script.php?script_id=2540)
功能: 输入简写展开
例: ‘bt’+Tab->‘belongs_to’, ‘hm’+Tab-‘has_many’, ‘rp’+Tab->“render :partial => ‘item’”, …
ruby.snippets文件中有各个简写展开形式的记录，也可以通过改写这个文件，自定义展开形式

* [ack.vim](https://github.com/mileszs/ack.vim)
功能: 根据文件内容查找文件，并切换文件
例: “:Ack ‘pattern’”

* [The NERD tree](http://www.vim.org/scripts/script.php?script_id=1658)
功能: 显示文件目录;


##Reference:
Rails with Vim : <http://Railscasts-china.com/episodes/Rails-with-vim>