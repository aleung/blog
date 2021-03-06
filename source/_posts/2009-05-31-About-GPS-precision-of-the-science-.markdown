---
layout: post
title: "关于GPS精度的“科普”"
comments: true
date: 2009-05-31 21:36
tags:
- GPS_GIS
---
对于GPS精度好多人有误解，我要在这里"科普"一下。  
  
**说法**：_GPS有两级精度权限，民用和军用。_  
**评论**：正确。简单的说，GPS有多种信号频率和不同编码，可以提供不同的精度。民用的L1载波的C/A码是低精度的。军用的精度是米级以内，民用的精度是10米级别。  
  
**说法**：_现在民用GPS的精度只能到100米。_  
**评论**：错误。早期GPS民用信号加入了SA（Selective Availability）干扰，精度只能到100米。但自从2000年5月开始，SA就被取消了，所以现在民用GPS精度是能达到10米级别的。  
  
**说法**：_手机的A-GPS功能是为了提高精度。_  
**评论**：不完全正确。A-GPS主要的效果是提高定位速度，而不是精度。  
普通GPS启动时需要搜索卫星信号，接收星历，才能定位与跟踪。卫星导航电文以30秒周期重复，GPS需要收齐4颗卫星的数据才能定位，因此在信号不好（断续）的情况下，就需要很长时间才能得到首次定位，甚至无法定位。  
A-GPS（Assisted GPS）是指利用移动网络来辅助GPS定位，具体技术又分为几种，常用的方案是通过移动网络发送GPS星历等辅助数据，GPS无需从卫星下载数据，使手机GPS在弱卫星信号的情况下都能快速定位。

**说法**：_Google地图的My Location显示我的位置偏移了几百米，那是因为民用GPS精度限制造成的。_  
**评论**：错误。这种大尺度（几百米）的位置偏移目前只发现在中国大陆地区地图出现，这不是GPS定位的误差，而是地图本身就是偏移，不准确的。
