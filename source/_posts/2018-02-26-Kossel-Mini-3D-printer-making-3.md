---
title: Kossel Mini 3D 打印机制作记录 (3)
date: 2018-02-26 22:00:36
tags:
- Maker
---

前篇：[(1)](/blog/2015/10/05/kossel-mini-3d-printer-making/),[(2)](/blog/2017/12/31/kossel-mini-3d-printer-making/)

# 继续调试

换上了淘宝上新买的喉管、铁氟龙送料管和送料管接头，出丝就正常了。另外，才知道铁氟龙管还有透明和乳白色两种，材质有点不同的，据说乳白色的更好。

第一次打印一个小方块出来，兴奋得不得了。

虽然能打印东西出来了，但实际上调试的磨难才刚刚开始。

打印校准立方体，发现问题 X、Y 尺寸有百分之几的偏差，直角不够垂直，最重要的问题是底部刚开始的几层错位严重，每层都比下面一层错位一两毫米。

## 出料堆积

观察发现打印路径上有很多短边和转折的时候，会在端点上有出料堆积，堆多了造成打上一层时打印头刮碰。以为是这个原因造成的错位。到百度“3d打印”贴吧上问人，别人指出了几个可能问题：

- 堆积材料因为冷却不够
- 打印头漏料
- 错位是因为步进电机丢步

一检查，果然打印头有漏料。因为加热铝块和铜喷嘴的热膨胀系数不同，常温下拧紧的喷嘴高温下变得很松动，需要加热后再拧紧。

又加了小风扇在旁边吹打印模型，加速降温。但这个不是主要原因。在切片软件中降低挤出系数减少出料量，就没有堆积现象了。

电机丢步的评论我没有重视。（那时我还不知道怎样的现象是丢步，后来才发现其实这个才是真正原因。贴吧的人气很低，问问题很多天都没什么回应，但上面有些人确实有经验。）

可是，问题依然存在。

## 框架偏差

仔细观察打印机框架，感觉立柱有一点点倾斜。用垂线测量了一下，确实是往一个方向出现了剪切位移。

框架用很多颗螺丝拧紧，型材插进角件中，有一点裕度，螺丝一压紧，难以控制它不倾斜。感觉要调正很困难，把解决的方向放在打印床上，想把玻璃打印床的固定螺丝换成弹簧可调节的，通过调整平面去让它与立柱垂直。

本来都要买调平螺丝了，晚上尝试了一下把打印机横放，底座的螺丝拧松一半，一个个直角来调整：先把螺丝拧到半紧，但用力还能稍微掰得动的程度，用直角尺量着立柱和平台的角度慢慢调整，直到把两根铝型材调整到正交，调好一处上紧一处的螺丝，一共6组。居然最后给调准了，用垂线测量基本上没有误差。

可是，这并不是导致层错位的原因。

## 自动调平

打印小立方体时，打印头只在打印平面的中心附近移动。尝试打大面积的校准模型，发现打印头在某些边缘位置会刮到打印床玻璃上，觉得是调平还有问题。

手工调平效率太低了，靠抽纸的方式来判断也不够准确。本来打算能自己打零件，才打印一个自动调平开关的安装座的，现在打印机的误差太大根本不能打零件。

于是买了一个压力传感器，无需安装，需要用时套到喷嘴上就行，当喷嘴下压到打印床上，传感器上下两片触片就会闭合。事实证明，这种传感器虽然贵一点，但绝对值得：

- 使用方便，套上去就行
- 与喷嘴没有高度差，没有水平偏差，不需要测量和在固件中设置位移
- 不需要在效应器上额外安装东西，不会增加重量

{%img /attachments/2018/2/1.jpg %}

另外也怀疑endstop的机械微动开关触发不准确，买了光电阻挡触发开关，不过后来也没有装上。

有了自动调平功能，看着打印头哒哒哒的敲打着玻璃板，十几个测量点反复测多轮，手工根本做不到，效率天上地下之差。

可是，自动调平之后，打印出来的立方体变形得更加厉害了，从底到顶都是歪的。

## 电机失步

有了压力传感器，尝试用 G30 指令做单次Z probe，发现连测几次结果都不一样，似乎是打印头接触玻璃板后没有立即停下来，造成错位了。修改固件，降低 Z probe 速度，慢动作之下就看出来了：打印头接触玻璃板后，水平滑动了一段距离才抬起——本来应该一探到玻璃板，开关触发，就马上抬起来的。以为是压力传感器不够敏感，试了用 M119 指令查看状态，手指头轻轻一压传感器，状态就变化了，也不是传感器问题。

而且，观察到这个滑行现象在探测点靠近B柱的时候特别明显，而在其他位置就不会发生。

百思不得其解，在百度贴吧和 [3dprinting.stackexchange.com](https://3dprinting.stackexchange.com/q/5292/162) 上提问，都没有人回答。（贴吧在半个月后有高人指出问题，可那时我已经解决了）

后来试试用手阻挡滑块的运动，感受一下电机的力度。电机的力度蛮大的，我才明白为什么刚开始出料堆积造成打印头刮碰时，别人都没当这是原因——材料被打印头刮碰时已经被加热软化，所产生的阻力对电机来说算不上什么。而试到B柱时，电机的力度不足，手稍微用力阻挡就会打滑——原来这个就叫做电机的失步！

没有亲自尝试体会，没法完全理解别人的经验。

更换了步进电机，重新自动调平，4点测量后就已经准确校准了，后面的25点、100点测量都是一个迭代就收敛。到此为止，打印功能完全正常。

# 完善

接下来，是一些完善的工作。

## 模型散热

E3D v6 挤出头配了散热风扇，只是给挤出头散热的，不吹打印工件，所以我用个充电宝带个小米风扇在旁边吹。这样不是太方便，而且风扇距离太远也无法做到材料一挤出喷嘴就能立即冷却固化。看别人的设计，有些是在打印头旁边加挂一个风扇吹工件，看起来臃肿占地方，也增加重量。

在 Thingiverse 上见到一个二合一的风扇风道[设计](https://www.thingiverse.com/thing:839620)，一个风扇分出两个风道，分别给挤出头散热和吹向喷嘴周边。

{%img /attachments/2018/2/duct1.jpg %}

制作了一个，感觉不够理想：

- 风扇凸出太多，打印头运动到边缘时会碰撞立柱，减少了可打印范围半径
- PLA材料弹性不够，夹在挤出头上可能会不够牢固

想到改进的方向：

- 外形紧凑，尽可能少凸出于效应器投影面：减少风扇倾角，尽量垂直放置；缩短风道长度
- 重用原来 E3D v6 配套的塑料风扇座，那个比较牢固

重新设计了一个，也发布到 Thingiverse 上了：https://www.thingiverse.com/thing:2781523

{%img /attachments/2018/2/duct2.jpg %}

## 挤出机导管

用的是 MK8 远程挤出机，铝合金材质很结实，回抽的效果也不差，但设计还说不上很完美：

- 虽然进料孔、出料孔和齿轮边缘在一条直线上，但由于挤丝的一边是平的挤丝齿轮一边是带槽的堕轮，丝受压就歪向带槽的一方了，造成很难将丝插入出料孔。在打印前穿丝还可以用小螺丝刀拨着方向慢慢试，在打印过程中想接丝就没可能做到了。
- 换丝时即使按下杠杠松开压轮，丝线还是会摩擦挤丝齿轮。但这也不算什么问题了。


看到有人用铁氟龙管替换堕轮，让整个进丝通道都在管中，我也试了一下。可是打印出来的工件很疏松，送料不足的样子，就装回堕轮了。后来再想，也许将挤压丝线的压力调大能有改善。

后来在出料孔前加了一小截铁氟龙导管，虽然还是做不到不靠手工辅助就能准确插入管中，但起码好操作一些了，打印过程中也能够接上另一段丝线。

{%img /attachments/2018/2/extruder.jpg %}

## 电路板外壳

电路板裸露会比较脆弱，得做个外壳装起来。

一个LCD控制器的外壳，一个 RAMPS＋Arduino的外壳，搞了好几天才弄好。最大的感受是：设计不难，建模不难，最麻烦的事情是怎么把尺寸测量准确。知道为什么说 3D 打印机是工业设计的好辅助工具了，电脑做的模型还是要做个真实样品出来才能验证，根据实物再做修改调整。如果没有快速成型工具，直接出模具，很可能就废了

做的各种零件，基本上都要打印两三次甚至更多才能最终确定设计。这种FDM 3D打印的速度其实很慢，半边外壳就要打印一个多小时。节省时间的方法是对于设计不确定、可能出问题的复杂的部分，先打印一块局部出来。

各种各样的测试品一堆：

{%img /attachments/2018/2/2.jpg %}

Kossel Mini 原来的设计里，电路板是放在框架的上方的。Delta结构打印机由于悬臂，上部空间浪费很多。于是我将LCD、Arduino和挤出机都改成挂在框架横梁下面，减少了整机高度，另外处于铝合金框架的保护之中，搬动、摆放也会安全一些。

{%img /attachments/2018/2/3.jpg %}

RAMPS＋Arduino的外壳的设计放在 Thingiverse上：https://www.thingiverse.com/thing:2806140 。LCD控制器我的做得不如别人的好，就没有放上去。

---

这个3D打印机设计全过程涉及的资料，包括安装手册、参考资料、物料清单（BOM）和淘宝购买链接、打印模型源文件、照片都存放在 https://github.com/aleung/kosselprinter


