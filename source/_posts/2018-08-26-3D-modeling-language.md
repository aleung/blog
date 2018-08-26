---
title: 3D modeling language
date: 2018-08-26 13:28:32
tags: ['Maker']
---

做好3d打印机，需要制作些什么就自己建模了。大部分的CAD工具都是图形交互式的，也就是靠鼠标把模型”画“出来，但我更喜欢通过建模语言来把模型描述出来。在网上搜寻和试用了一些基于3D建模语言的CAD工具，将我了解的记录一下。

# CAD建模语言

#### OpenSCAD 

网站：<http://www.openscad.org/>

文档：<http://www.openscad.org/cheatsheet/index.html>

这个领域的领头项目，知名度最高，使用者最多。 

虽然不是一门通用语言，但是也有不少通用语言的特性，例如变量、函数、条件判断、循环。

#### Relativity SCAD 

文档：<https://github.com/davidson16807/relativity.scad/wiki>

不是独立语言，只是OpenSCAD的library，但是它提供了一套全新的API替换掉OpenSCAD原有的API的大部分功能，使用起来比OpenSCAD API简单很多。 

是一个个人项目，近两年基本上没有更新了，但看上去完成度很高，也许是没有什么需要更新了，可靠程度有待验证。 

特别之处： 

- 通过描述对象之间的相对位置，大大简化了OpenSCAD建模中繁琐的transformation 
- 通过selector对部分对象进行操作，减少了boolean operation的繁琐组合 

感觉相当完美的解决了OpenSCAD使用过程中的痛点，写出来的代码简洁很多。 

#### CraftML 

网站：<https://craftml.io/>， <https://github.com/craftml/craftml>

Markup language 风格的建模语言（类似HTML）。1.x版本已经被放弃，github repo都删除了；现在的2.0 beta是重写过的，但似乎也是个死项目。 

#### OpenJsCAD 

网站：<https://github.com/jscad/OpenJSCAD.org>

Online IDE: <https://openjscad.org/> 

使用 JavaScript 语言建模，支持类似于OpenSCAD函数风格的API，和CSG对象风格API（已经deprecated）。

优势是使用 JavaScript 有很强的表达能力和扩展能力，可用利用npm模块来扩展。

不足： 

- API混乱，1.x的API已经deprecated，2.0 API又未完善 
- 文档错误多 
- API使用错误时没有定位错误位置，难以debug 

#### SolidPython 

网站：<https://github.com/SolidCode/SolidPython>

Python frontend for OpenSCAD。用 python 来生成 OpenSCAD 代码，也是依托于通用语言的强大表达能力和扩展能力。

近一年开发不活跃，但项目应该依然处于有维护状态。 

特别之处： 

- first-class negative space (holes) 
- 直接用 + - * 运算符表示 object boolean operators 

不足： 

- Python多行表达式书写不方便，需要人工调整缩进 

#### ImplicitCAD 

网站：<http://www.implicitcad.org/>，<https://github.com/colah/ImplicitCAD>

Online IDE：<http://www.implicitcad.org/editor> 

类似于OpenSCAD的语言，按项目文档的说法是：ExtOpenScad，with many things missing, many added, and many just working differently。 

项目未完全完善，处于beta测试可用状态。 

工具本身是用Haskell实现的。 

特别之处： 

- Extrude支持函数参数，产生变化的挤出效果 
- 圆弧化object对接的边缘 

# 我的选择

最先是用 OpenSCAD 的，但是用 OpenSCAD 做稍复杂一些的模型，代码就要写得很繁琐，故此寻找有没有其他更好的方案，就有了上面一批工具的尝试。

使用了一段时间的 OpenJsCAD，因为是我熟悉的js语言，v1 API有丰富的参数让代码写起来简洁一些，一度认为是最佳的选择。但后来发现它正在进行的向 v2 API 的迁移太混乱了，用户很难预期到底应该使用哪种方式。另外，近来几个月的开发工作也停滞不前了，觉得项目当前的成熟度有点问题。

其他的几个工具也都是有很大的成熟度问题，于是只好又回到了 OpenSCAD。Relativity SCAD 是最近的发现的，只用来做过一个模型，目前的体验相当好，应该可以作为今后的主要工具。可靠性、完整度还有待在更多的使用中去验证。

