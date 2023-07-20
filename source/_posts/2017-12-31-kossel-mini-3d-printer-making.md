---
title: Kossel Mini 3D打印机制作记录 (2)
date: 2017-12-31 14:41:46
tags:
- Maker
---

{%img /attachments/2017/12/kossel.jpg %}

[上一篇](/blog/2015/10/05/kossel-mini-3d-printer-making/)是2015年国庆写的，当时是滑轨质量不行需要更换。后来重新买了滑轨，装配起来，测试过电路板、电机正常，然后就停工了。一放两年，这两个星期终于重新拿出来，继续制作。

<!-- more -->

两年过去，Kossel打印机已经不是DIY热门了，搜索到的新信息很少。另一方面，淘宝上配件的价格普遍下降了不少。

# 装配

其实主要部分在2015年底已经装好了，就差滑车限位开关和挤出机没装上。装起来后就是上面照片的样子啦。

电线虽然已经理过了还是有些乱，等能打印后设计个外壳把电路板罩上。

# 调试

装配还好，调试才是累人的活。

升级到最新的 Merlin 固件，delta结构打印机的配置参数减少了一些，网上的调试教程普遍是两三年前的，针对旧版本固件，已经不适用了。

## 基础知识

Delta 结构打印机的坐标原点在打印床正中心，三根立柱依次编号为 alpha / beta / gamma，但也有称为 X, Y, Z 柱，不能跟坐标系的三个轴混淆。三根立柱和xy平面的位置关系如下图，圆形为可打印半径。需要先计算好A, B, C 三点的xy坐标，后面调试要用。

{%img /attachments/2017/12/delta_columns.jpg %}

打印头的移动可以分解为：

* z轴上的运动：三个柱的滑车同时上下移动相同距离；
* xy平面上的运动：三个滑车上下移动不等距离，造成斜杆倾斜角度变化而改变斜杆在平面上的投影长度；

需要给出的参数就是以下几个：

* `DELTA_DIAGONAC_ROD` - 斜杆长度
* `DELTA_RADIUS` - 打印头处于平面正中央时，斜杆在平面上的投影长度
* `DELTA_HEIGHT` - 打印头可以到达的最高点的z坐标
* `DELTA_PRINTABLE_RADIUS` - 可打印半径
* `DEFAULT_AXIS_STEPS_PER_UNIT` - 各电机（包括三个柱，和挤出机）走1mm需要的步进脉冲数量

打印机怎么知道滑车的位置呢？滑车由步进电机驱动，根据`DEFAULT_AXIS_STEPS_PER_UNIT`可以推算出相对运动距离。在轨道最高点（endstop）安装有感应开关，滑车复位时上行至最高点直到感应开关触发，此处的位置对应着z轴的`DELTA_HEIGHT`值。

{%img /attachments/2017/12/delta_radius.jpg %}

## 参数调整

### 初始设定

`DEFAULT_AXIS_STEPS_PER_UNIT` 这个参数的调准是第一步。我用的步进电机基本步距1.8°，RAMPS驱动板采用1/16微步（microstepping），也就是转一圈需要3200个脉冲。XYZ柱电机安装GT2同步轮，齿距2mm，16齿，也就是转一圈滑车移动32mm，由此算出 100 steps/mm。对于挤出机(E)，根据挤出轮直径12mm，算出周长37.7mm，得到 84.88 steps/mm。

尽可能精确的测量斜杆长度（球头中心点距离），填入`DELTA_DIAGONAC_ROD` 。

将三个滑车推到最高点，测量打印嘴离打印床的垂直高度，稍放大几毫米余量，填入`DELTA_HEIGHT`。

`DELTA_RADIUS` 的值很难测准，可以通过用细线挂重物的方式来找出垂线。两三毫米内的误差也是可以的。

### 调准高度和endstop偏移

三个立柱的 endstop 感应器安装位置会有偏差，不在一个水平面上，`DELTA_ENDSTOP_ADJ` 参数可以矫正这个误差（如果有EEPROM，也可以用 G-code `M666` 设定这些参数并用`M500`保存）。

为了调校方便，需要在printing host软件中先设定4个宏，分别让打印头移动到前面基础知识坐标平面图中的 A, B, C, O 点。宏包含以下 G-code，其中x,y为点的坐标：

```
G28
G0 X<x> Y<y> Z5 F5000
```

执行宏后，打印头会悬停在对应点上方的几毫米处。在打印床上放一张复印纸。在printing host软件中控制打印头z轴单步下降，接近后每次下降0.1mm，抽拉复印纸根据打印头有没有摩擦到纸张来判断打印头是否已经降到打印床表面，如果纸张被打印头压死抽不动就是已经太低了。记录下最后的z坐标（打印机液晶屏显示，或者`M114`指令获取）。

根据A, B, C点测量到的值，调整 [`M666`](http://reprap.org/wiki/G-code#M666:_Set_delta_endstop_adjustment) 的X, Y, Z参数，反复迭代测量、调整，直到A, B, C点打印头接触打印床时z坐标为零。这个步骤完成后，打印机的三个立柱上滑车的z高度就是准确和同步的了。

### 调准radius

上面调整结束后，O点数据不为零，也就是打印头在xy平面上运动，z坐标不变的时候，实际上的运动轨迹是一个碗形，可能向上凸也可能向下凹。要纠正碗形失真，需要精确设置 `DELTA_RADIUS` 参数。用 [`M665`](http://reprap.org/wiki/G-code#M665:_Set_delta_configuration) 也可以调整这个参数 (R)。

调试过程类似，也是迭代调整参数和测量，直到 A, B, C, O 点均测得零值。

### 调准斜杆长度

接下来的校准需要通过打印一个立方体来进行，用游标卡尺测量打印出来的真实 x, y 边长，根据与期望长度的比例来调整 `DELTA_DIAGONAC_ROD`（或`M665`的L参数）。

由于打印头问题，这个测试还没能开始。

### 调试数据记录

下表是几次迭代校准的过程，左边的是使用的参数，A／B／C／O为测量值。

| M666 X | M666 Y | M666 Z | M665 H | M665 L | M665 R | A    | B    | C    | O    | Comment   |
| ------ | ------ | ------ | ------ | ------ | ------ | ---- | ---- | ---- | ---- | --------- |
| -6     | -8     | -6     | 250    | 214    | 103    | 0    | 0.8  | 0.7  | 1.3  |           |
| -6.2   | -7.2   | -5.3   | 250    | 214    | 103    | 0    | 0    | 0    | 0.7  | 高度和水平已校准  |
| -5.5   | -6.5   | -4.6   | 250    | 214    | 101    | 0.1  | 0    | 0.1  | 0    |           |
| -5.5   | -6.6   | -4.6   | 250    | 214    | 101.2  | 0    | 0    | 0    | 0    | radius已校准 |

## 挤出头

运动部件的校准还说顺利，可是真准备首次打印的时候才出问题。

进丝一段之后就挤不进去了，步进电机前进一下就往回跳。喷嘴一点料都没有挤出来。尝试回抽，拔一小段后也卡住。拆开打印头，发现打印丝在离前端几毫米的地方形成了一个环，造成不能前进也不能后退。

散热风扇正常，升高降低温度都试过，四氟管已经确认插到头，都没发解决进丝一到喉管瓶颈为就堵塞的问题。从拔出来的堵塞物看，是丝进入喉管瓶颈位置后软化膨胀变粗，刮起表面一层，堆积成环状就卡死了。

{%img /attachments/2017/12/jam.jpg %}

这卷耗材还是两年前的，拆过封口，时间太长会对材料有影响。换了一卷没有拆封的耗材来测试，这次能过顺利进丝通过瓶颈位，喷嘴也出丝了。可是接下来加载模型打印，又不出料了。

上网查资料，发现现在的喉管有改进型的，里面带铁氟龙管内衬，或者是可以让进料铁氟龙管直通过去，一直到喷嘴处。在耗材可能软化的段都是铁氟龙管内衬，摩擦力很小而且没有接缝，就不容易出现堵塞。

{%img /attachments/2017/12/hotend.jpg %}

元旦店家都休息了，又一次陷入等待状态......