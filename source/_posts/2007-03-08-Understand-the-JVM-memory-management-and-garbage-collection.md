---
layout: post
title: "了解JVM的内存管理与垃圾回收"
comments: true
date: 2007-03-08 04:31
updated: 2022-08-16
tags:
- SoftwareDev
---

v2 - 2022-8-16
v1 - 2007-3-8

Java语言具备GC(垃圾回收)的能力，内存管理不需要应用程序去过问，这很方便。但是，GC是怎么进行的，JVM的内存参数应该怎么调整，如何优化，往往我们不是太清楚。看过一些资料后，对Sun JVM的内存管理以及垃圾回收的机制大概有了一个概念，这里将这些资料归纳和翻译出来。本文初稿内容主要基于Sun JVM 1.3.1，在后续版本中有不少优化措施，但是这些基本概念还是不变的。

## GC

当JVM进行GC的时候，是要消耗CPU资源和需要一定时间的，这会影响到程序的正常运行，因此需要尽可能减少GC消耗的时间。Java程序运行过程中，对象的生命周期有长有短，其中相当大部分是都是比较短命的，例如局部的对象一用完就可以回收了。在大多数情况下，只要能够及时回收这些短命对象的内存，就能够确保JVM有足够内存来分配给新的对象。因此JVM采用一种分代回收(*generational collection*) 的策略，用较高的频率对年轻的对象(young generation)进行扫描和回收，这种叫做minor collection，而对老对象(old generation)的检查回收频率要低很多，称为major collection。这样就不需要每次GC都将内存中所有对象都检查一遍。

内存收集方式的最基础方式是以下几种：

一种称为copying或scavenge，将所有仍然生存的对象搬到另外一块内存后，整块内存就可回收。这种方法有效率，但需要有一定的空闲内存，拷贝也有开销。这种方法用于minor collection。

另外一种称为mark-sweep，从GC root开始根据引用遍历对象，将活着的对象标记出来，扫描一遍后，没有被标记的内存就可以回收了。这种方法不需要占用额外的空间，但会形成内存碎片。这种方法用于major collection.

在mark-sweep基础上发展出来一种方法称为mark-compact，将活着的对象标记出来后，搬迁到一起连成大块的内存，然后回收其他内存。这种方法不需要占用额外的空间，但迁移带来额外开销，速度相对慢一些，用于major collection.

在以上几种基础的内存收集方法之上，有各种改进，例如：增量式回收，每次只处理一小部分；多线程平行执行；通过各种方法缩短stop the world（普通GC在执行过程中要暂停应用的执行）时间。

目前版本 JVM 支持多种GC实现方式，通过 `-XX:+UseXyzGC` （Xyz为GC方式名字，具体查看文档）来指定：

- Serial GC - 串行使用单核CPU的mark-compact
- Parallel GC - 使用多线程进行，也称为 Throughput Collector
- CMS（已经被G1替代） - Concurrent Mark Sweep，分阶段并行的进行mark和sweep，目标是缩短GC停顿时间，只在某些阶段需要暂停应用执行
- G1 - Garbage First，并行的mark-compact，可以增量进行
- ZGC（在Java 15起提供）- 使用颜色指针标记，并行处理所有高开销操作，暂停应用短于10ms

## Heap Memory

JVM管理的内存，通常叫做堆(heap)，可以用下面的图来描述。

JVM启动后，保留一段地址空间，这个空间的大小由-Xmx指定。这块空间的大小就是heap可能的最大值，但一开始不一定全都分配了物理内存，初始分配的heap大小由-Xms指定，如果-Xms小于-Xmx，剩余部分是virtual的，当需要的时候，再向OS申请。

Young generation 内存由一块Eden(伊甸园，有意思)和两块Survivor Space构成。新创建的对象的内存都分配自eden。两块Survivor Space总有会一块是空闲的，用作copying collection的目标空间。Minor collection的过程就是将eden和在用survivor space中的活对象copy到空闲survivor space中。所谓survivor，也就是大部分对象在伊甸园出生后，根本活不过一次GC。对象在young generation里经历了一定次数的minor collection后，年纪大了，就会被移到old generation中，称为tenuring。

剩余内存空间不足会触发GC，如eden空间不够了就要进行minor collection，old generation空间不够要进行major collection。

很多参数会影响里面各部分空间的分配。-XX:MinHeapFreeRatio与-XX:MaxHeapFreeRatio设定空闲内存占总内存的比例范围，这两个参数会影响GC的频率和单次GC的耗时。-XX:NewRatio决定young与old generation的比例。Young generation空间越大，minor collection频率越低，但是old generation空间小了，又可能导致major collection频率增加。-XX:NewSize和-XX:MaxNewSize直接指定了young generation的缺省大小和最大大小。

{%img /attachments/2007/3/java_memory.jpg %}

-Xmx
set maximum Java heap size

-Xms
set initial Java heap size

-XX:NewRatio=2
Ratio of old/young generation sizes.

-XX:NewSize=2.125m
Default size of new generation (in bytes)

-XX:MaxNewSize=
Maximum size of new generation (in bytes). Since 1.4, MaxNewSize is computed as a function of NewRatio.

-XX:SurvivorRatio=
Ratio of eden/survivor space size. For example, when SurvivorRatio=2, one survivor space size is 1/4 of young generation.

(上面给出的缺省值不一定准确，不同JVM版本和不同OS环境下会有不同)

## Non-Heap Memory

除了heap，JVM还有多种内存使用，下面列出主要的：

### Metapace

Java 8 之前的版本有permanent generation，属于heap space的一个特殊部分，是JVM用来保存class object和meta data，大小由-XX:PermSize和-XX:MaxPermSize指定。大量动态生成(编译)和加载class会增加这部分内存的耗用。但从Java 8起，不再有permanent generation，而是改成了放在native memory中的 metaspace，大小会自动增长，也可以由 `MetaspaceSize` and `MaxMetaspaceSize` 限制。

### Code cache

用于存放JIT编译的代码和native代码

### Thread

每个线程都有一个栈，栈的大小由 `-Xss` 控制。

### Direct Buffer Memory

可以在Java应用中申请使用的堆外内存，经常用于NIO，也可以在JVM之间共享。大小缺省与heap相同，也可以由 `-XX:MaxDirectMemorySize` 限制。

## Reference

以上给出的只是基本的介绍，下面reference中的文章都很不错，对进一步了解或者查找性能优化参数都有帮助。

1. [Guide to the Most Important JVM Parameters](https://www.baeldung.com/jvm-parameters)
2. [Memory footprint of a Java process (Youtube)](https://www.youtube.com/watch?v=c755fFv1Rnk)
3. <http://java.sun.com/javase/technologies/hotspot/vmoptions.jsp>
