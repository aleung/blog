---
title: 在非GPL应用中使用OpenJDK的法律问题
layout: blog
date: 2017-01-06 22:56:07
tags:
- SoftwareDev
- OpenSource
- Java
---

大家都知道 Oracle JDK 在商业应用场合是需要购买 license 的，往往会选择 OpenJDK 来规避。但是，OpenJDK 的 license 是 GPL，GPL 是一种 “传染性” 的协议，那么 OpenJDK 是否可以在非 GPL 应用中使用呢？

在 [StackOverflow](http://stackoverflow.com/)、[Open Source Stack Exchange](http://opensource.stackexchange.com/) 中搜索到一些相关的讨论，大部分答案是认为可以使用，应用无需因此用 GPL 发布，但也有人提出质疑。

在 OpenJDK 网站中的 [FAQ](http://openjdk.java.net/faq/)是这么写的：

> ### What open-source license is OpenJDK published under?
>
> GPL v2 for almost all of the virtual machine, and GPL v2 + the Classpath exception for the class libraries and those parts of the virtual machine that expose public APIs.
>
> ### How do I know which license applies to a given source code file in OpenJDK?
>
> Each source code file is individually licensed - look for the copyright header with the license information.

比较坑爹的是整个 OpenJDK 居然不是统一使用一种授权协议，每个源文件有自己独立的授权——要明确你使用的法律责任，得一个一个源文件看去！不过，按照 FAQ 里面的说法（如果它是准确的），类库和 JVM 的公共 API 都是用 GPL v2 ＋ Classpath exception (GPL v2 + CE) 发布的。

这个 classpath exception 是什么回事？GPL v2 要求：基于 GPL 软件派生出来的作品也需要继承使用 GPL 协议发布。一个程序动态链接或者静态链接了一个库，都视为此库的派生作品。按照这个定义，在 Java 程序里使用了（在classpath中包含了）某个类库，就是此类库的派生作品。然后，OpenJDK 的某些源代码的授权声明中，在 GPLv2 协议文本之后附加了一段 [classpath exception](http://openjdk.java.net/legal/gplv2+ce.html)，对于链接这种情况给予了豁免，可以自由选择授权协议。

我的理解，这是为了让 OpenJDK 可被商业应用（非 GPL 授权应用）使用。JVM (HotSpot) 的代码大部分都是用 GPL （没有 classpath exception）授权，但因为 Java 应用只是在 VM 上运行，并没有与 VM 链接，所以不受 GPL 的限制。

但是，没有那么简单。咨询公司开源部门的专业人士，他让我谨慎的评估一下。因为 OpenJDK 作为一个开源项目，并没有严格的控制提交代码采用的授权协议，很可能某个代码提交增加了一个新的源文件，其授权是不带 CE 的，或者将已有文件的授权的 CE 部分删除了，应用使用了这个源文件编译出来的库就中招了。

我下载了 OpenJDK7 的源代码，用脚本扫描了所有 `*.java, *.c*, *.h*` 文件，找出带有 GPL 但又不包含 Classpath exception 的文件。除去 HotSpot 部分和 example 代码，并不多：

```
./share/lib/security/BlacklistedCertsConverter.java
./aix/porting/porting_aix.*
./share/native/sun/security/ec/impl/*
```

第一个是个工具（带有main函数），不是库的一部分；第二个是 AIX 系统相关的；第三个是属于 [SunEC provider](http://docs.oracle.com/javase/7/docs/technotes/guides/security/SunProviders.html#SunEC)，实现椭圆曲线加密算法（ECC）的。

需要我们注意的只有第三个，对应着文件 `jre/lib/ext/sunec.jar`。也就是，Java应用确保不使用这个jar，就可以用 OpenJDK 而不需要用 GPL 发布。一旦使用了 `sunec.jar`，整个 Java 应用都必须用 GPLv2（或其兼容协议）发布，开放源代码。

但我上面的分析仅仅是基于当前版本的 OpenJDK7，OpenJDK 每次发布代码都会变化，故此每次要安装升级时都必须重新检查它的所有源代码的授权协议。对于商业软件来说，保险起见还是乖乖给 Oracle 付费使用他们家的 JDK 吧。

