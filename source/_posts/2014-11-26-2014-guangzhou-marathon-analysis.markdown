---
layout: post
title: "2014广马数据分析"
date: 2014-11-26 07:38:06 +0800
comments: true
tags: 
- Running
---
*Updated 2014-11-27: 之前漏了一些号码段，重新补上更新了。*

2014广州马拉松在网上可以根据号码查询成绩和名次了，但是没有开放完整成绩册的数据下载。出于好奇，想知道成绩的分布：跑330以内的有多少人？400呢？有多少人最后关门一刻才冲线的？于是写了个程序把数据抓下来分析。

{%img right /attachments/2014/11/groups.png %}

数据不包含号码以0开头的专业选手（大概几十人）。由于不知道确切号码段，没完赛选手的数据有可能有误。

首先是各组的人数，男女比例全马约8:1，半马约3.6:1。跑步的妹子相对还是比较少，女子只要完赛了，总能够拿到个几百名的名次的 :)

然后看成绩分布。

半程组里有15%男选手和22%女选手未能完成比赛，而全程组不能完赛的男女选手比例分别是14%和21%。不一定是女子毅力不行，是相同的关门时间对女子更困难些吧。但有意思的是虽然全程更痛苦，但完成率跟半程是基本上一样的。

半马组，一半的男选手在2:30内跑完，一半的女选手在2:45内跑完。全马组，这个数字是男子5:00，女子5:20。有能力5小时内跑完全程的，已经强过一半的人了。

有几十人是过了关门时间才冲线的，依然有成绩，看来组委会还不是那么不近人情。

{%img /attachments/2014/11/half-marathon.png %}

{%img /attachments/2014/11/marathon.png %}

能跑330是什么水平？400呢？看看下面的表格，比例是表示有多少人在这个时间内完赛。

- 男子全程

| 时间 | 比例 |
|----:|----:|
|3:30 | 4.0%|
|4:00 | 15.6%|
|4:30 | 32.1%|
|5:00 | 52.0%|
|5:30 | 67.2%|
|6:00 | 85.5%| 

- 女子全程

| 时间 | 比例 |
|----:|----:|
|3:30 | 1.3%|
|4:00 | 5.7%|
|4:30 | 17.1%|
|5:00 | 38.6%|
|5:30 | 58.0%|
|6:00 | 78.2%| 