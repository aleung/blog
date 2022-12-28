---
layout: post
title: "图像格式介绍: jpg, gif, bmp, png, ecw......"
comments: true
date: 2004-06-29 21:39
tags:
- ImageProcessing
---
_图像格式介绍在网上有很多资料, 随便google一下都能找到. 这是我在论坛上回复别人的帖子, 顺手也帖到这儿._

这几种格式都是光栅图像格式(除了光栅格式还有一种是矢量格式), 描述的是图像点阵的色彩数据.

BMP格式是最简单的格式，基本就是图像点阵的内存拷贝。没有任何压缩，也无法表示格外的数据。现在很少用BMP来做图像数据交换了。

GIF和PNG都是索引色彩, 也就是不直接描述象素的颜色, 只是说这个点是几号颜色, 另外有个索引表（调色板）给出颜色号对应的RGB颜色。有简单的压缩算法，但这种压缩仅对于连续色彩的象素（例如大块色块的图形）才有效，适用于颜色有限的图像(如商业图形/地图/漫画), 对真彩色图像(如照片)不太合适. GIF的LZW压缩算法是有专利限制的（好像是在去年已经过期失效）, 需要授权才能使用, 故此w3c组织另起炉灶订立PNG格式, 对于使用而言两者差不多.

JPG是有损压缩, 利用人眼对高频细节分辨不是很敏感的特性, 将数据量减少。当压缩率过大时, 会在图像色彩变化的边界出现马赛克的现象. 但是作有限度的压缩时, 图像质量损失并不明显, 往往不能察觉。它的压缩率是可以调节的，让制图者可以在图象质量与图像文件大小间取得一个平衡点，因此是很有用的一种格式, 基本上网上的照片都是用jpg格式.

相比起以上的各种格式，TIFF格式更多的是一个框架，它规定了文件的数据段结构，但是对图像数据的具体编码算法没有定义，是可扩展定义的，因此其下发展出很多的子格式。例如原始的TIFF格式是无压缩的，现在也有多种压缩算法的扩展；将地理数据也保存到数据段中，就是GeoTIF格式。因为子格式很多，不是所有图像处理软件都支持，一般都只是支持无压缩的基本格式，故此有时会出现无法打开文件，或者显示图像不正确的现象。TIFF格式在出版业得到比较广的应用，作为标准的图像交换格式。

在地理信息领域，还使用一些其他的有损压缩格式，例如基于小波算法的ecw和MrSID格式。但是这些格式目前还基本是用于专业领域，不如jpg通用，支持的图像处理软件也不多。

对于卫星影像的最终输出，我觉得还是采用有损压缩格式比较好，因为图像数据量实在太大了，如果不压缩或者压缩过小，对于存储、传送都不方便。采用合适的压缩率，图像质量基本不会受到影响，看看网上的漂亮mm照片，都是jpg有损压缩的:) 如果是在oziexplorer中观看，建议使用ecw格式，ecw的好处在于可以局部加载数据，减少内存占用，对于超大图像有好处；它不像其他格式，即使只显示图像其中一小快，也要将整个图像数据全部解码放入内存。但是能处理ecw的软件很少, 只能在最后输出最终结果时转换.

地图制作，看什么图，如果是色块为主的图，用gif/png比较合适，如果色彩都是过渡的，就还是用jpg或者ecw好。

图像压缩：[http://media.cs.tsinghua.edu.cn/~ahz/digitalimageprocess/chapter14/chapt14_ahz.htm](http://media.cs.tsinghua.edu.cn/~ahz/digitalimageprocess/chapter14/chapt14_ahz.htm)