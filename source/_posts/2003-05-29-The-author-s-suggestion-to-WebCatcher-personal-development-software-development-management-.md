---
layout: post
title: "给WebCatcher作者的建议（个人开发软件的开发管理问题）"
comments: true
date: 2003-05-29 03:00
tags:
- SoftwareDev
---
[WebCatcher](http://www.wizissoft.com/cn)是我非常喜欢的一个软件，作者很热情的在论坛中解答问题、了解意见。我觉得个人开发软件同样存在着软件开发管理的问题，这是我写给他的一个建议： 

* * *

估计这段时间老魏也够烦的：）

问题一个接一个，用户意见也不少。。。

我对这个软件还是很欣赏的，作者的技术水平也不错。

不过这次在版本升级的过程中暴露出了在开发管理中的问题，呵呵，一个人写程序也是要有开发管理的，因为随着软件规模增大、复杂性增加，遇到的问题也会越来越多。

不知道作者有没有用版本管理软件来管理源代码，用版本管理软件的分支－合并功能可以做到同时维护旧版本的bug和增加新版本的功能，另可避免同样的bug重复出现。已发布版本可以不断出bug fix版本，但不包括新功能；开发中版本包含新功能，可以定时发布供测试。大多少用户还是希望软件把工作稳定放在第一位的，而不是当测试者。

软件的发布的细节也不够细致，例如供下载的测试版本没有区分版本号，没有详细change log。无论修改得多还是少，只要是公开/半公开发布的，都必须要有不同的版本标识，小改动的发布可以只更改子版本号或者build序列号，但一定要能跟旧版本区分。change log是让用户（也是让开发者自己）知道各版本的区别的，也不应该省略。

上次提过这个意见： [http://211.92.136.88/~wsj/BBS/viewtopic.php?t=578](http://211.92.136.88/~wsj/BBS/viewtopic.php?t=578)  
其实并不是说要让用户自己去添加bug信息，这个还是应该由作者（或者找一个专人）做的，将用户反馈收集整理后添加。用不用专门软件也不是很重要，专门软件用起来方便些，不过开始的安装配置和学习使用也麻烦，重要的是有这样的流程，并且能给用户反馈。例如，用户发现了问题，一查就可以知道这个问题是否意见被别人发现了，在哪个版本已经修正，或者是有什么临时的解决方法。

作者与用户沟通的态度很令人赞赏，回答问题、接受建议都很认真诚恳。不过有些细节上还可以更进一步，例如用户发现一个bug，作者马上说“谢谢，我会尽快改进”，用户会很高兴，因为作者重视他的意见。可是，这个bug什么时候修改了，在哪个版本中？没有了下文。用户只好一个个版本下载试用，然后发现自己提到的bug还是存在，而且还多了一些新的问题，长久下去可能就会厌烦而放弃这个软件了。作为用户，关心的是软件是否能符合自己的需求，如果他某一天在新版本change log中看到自己提出的问题意见解决了，才是真正的心满意足。个人做软件比起大公司的优势也在于可以跟最终用户更加贴近，实际上，好的个人软件的用户忠诚度往往比大公司的软件更高。

另外给个建议，作者的免费终生升级策略以后也许要考虑改改，否则你只能靠发展新用户来获得收入，而随着用户增加，支持的压力却越来越大。用适当的金钱来换取良好的服务、功能升级，对于用户来说也是很合理的。老实说，如果作者因为经济原因不能继续软件的开发维护，对于已有用户来说是更大的损失。