---
layout: post
title: "终于找到了最合适的文件同步/备份工具"
comments: true
date: 2007-05-28 06:06
tags:
- Software
---
很长时间以来，都为怎么在多台电脑之间同步文件而头痛。使用过不少的同步软件，都有或多或少的不足之处。因为两边的文件都可能被编辑，同步软件必须要足够自动化，能够自动判断同步的方向，只进行增量同步，并且要能够识别出文件的删除，别删了之后一同步又回来了。 

现在终于找到了一个满足要求的软件：[Unison](http://www.cis.upenn.edu/~bcpierce/unison/)，经过差不多三个月的使用，感觉相当满意，可以介绍出来了。 

在我看来，Unison的优点有这些： 

  * 除了本机磁盘同步外，还支持网络同步，可以通过socket通讯方式，而不是靠windows文件共享。我的laptop通过共享copy文件时，偶然会出现出错。Unison还支持ssh，在unix上可以用安全程度较高的ssh。 
  * 有命令行模式，可以通过脚本自动运行。其实Unison本来就是命令行程序，只是提供了GTK的GUI。GTK的界面虽然不太漂亮，但还是很好用的，在GUI模式下，会先把各个文件的同步方向显出出来，可以修改，点击go才执行。 
  * 自动化程度比较高，每次同步完成后会将文件状态记录下来，下次同步的时候就能自动进行增量同步了。只要不是两边同时发生了修改，基本不需要人工干预。 
  * 配置比较灵活，可以按照通配符来忽略某些文件/目录。因为Unison原来是unix平台的软件，所以配置也是文件方式的，只习惯于Windows图形界面的人也许会不喜欢。 
  * 支持跨平台同步，包括windows，各种unix，OS-X。这点我用不着，但对于同时使用不同操作系统的人会很有用。 
  * 免费，开源(GPL)。

我的工作环境有一台desktop和一台laptop，desktop的性能好些，所以一般都会在desktop上干活，但不在座位上时就只能用laptop，故此两台电脑都要有相同的文件，并且保持同步。现在保持同步的方法是：在desktop上以socket server模式运行unison，一直保持运行；laptop上通过cron在后台定时调用命令行的Unison，一天可以运行多次；如果我需要立即进行同步，就手工运行Unison GUI。 

一些tips： 

  * 第一次同步会耗很多时间，因为unison要先对所有文件内容进行一次比较。以后的同步，如果在fastcheck模式下就会很快了。一定要记得在配置文件中加上配置：fastcheck = yes，否则每次都还是会做全文比较。非windows平台就没有这样的问题，具体看软件文档。 
  * 作为socket server运行：unison -socket <port>
  * 命令行模式自动运行(无需人工干预，如果文件有冲突就skip)：unison -batch <profile_name>

顺便推荐一下这个Windows平台下的[cron](http://aleung.blogbus.com/files/11892516220.zip)，一个几十k的小程序，使用起来跟unix的几乎一模一样，很好用。（2007-9-8 补充: 网上cron的页面已经没有了，我已将链接改为本地下载） 