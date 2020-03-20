---
title: Hackintosh Installation
date: 2016-04-11
tags:
- Software
- Hardware
---

装了一台黑苹果在公司里用，幸福指数大大提升。

# 硬件

硬件选型参考tonymacx86的[CustoMac Mini](http://www.tonymacx86.com/building-customac-buyers-guide-march-2016.html#CustoMac_Mini))，算下来主机不足4000元。要保证安装顺利、功能正常和稳定，最重要的是选择推荐的主板型号，可以找到详细的安装配置教程，各种可能的坑都有人给出解决方案了。安装过程一律依据tonymacx86的guide进行，没有任何问题。SSD和机械硬盘组成fusion drive，在较低的成本上达到不错的性能和容量的兼顾。唯一问题是用了fusion driver之后hibernate就不能用了，因为Clover认不出fusion driver，找不到上面的sleep image。

# 软件

以下内容均针对 El Capitan。

## App安装途径

安装应用途径，按照优先顺序：
- [Homebrew Cask](https://caskroom.github.io/)
- [Homebrew](http://brew.sh/)
- App Store
- 官网下载

### 升级已安装的所有brews

``` sh
brew update
brew upgrade
```

### 升级已安装的所有casks

From: [http://apple.stackexchange.com/a/231020/13455](http://apple.stackexchange.com/a/231020/13455)

``` sh
curl -s https://gist.githubusercontent.com/atais/5d3ec79e67f221cb55b7/raw/f6e6b647e6f90939c016ba88e591529d143cb33d/cash_upgrade.sh | bash /dev/stdin
```

## 强大的命令行

- 用iTerm2替换系统自带的terminal
  - 安装 Solarized Dracula theme （不要选Solaris dark，好多工具的输出会看不见）
  - 更改字体
- 用zsh替换缺省的bash
  - 使用 Oh-my-zsh 进行zsh的配置，以上步骤参考这里 [https://gist.github.com/kevin-smets/8568070](https://gist.github.com/kevin-smets/8568070)
  - 设置为超酷的theme: [agnoster](https://gist.github.com/agnoster/3712874)
- 安装 [fasd](https://github.com/clvv/fasd)


## Finder增强

### 快速切换是否显示隐藏文件

按照[这里](http://apple.stackexchange.com/questions/177132/how-to-set-h-to-enable-show-hidden-files/177138#177138)的方法创建一个Automator Service。

### 增强 Finder 预览文件的能力

安装quick look plugins： [https://github.com/sindresorhus/quick-look-plugins](https://github.com/sindresorhus/quick-look-plugins)

## 增强剪贴板

安装 ClipMenu，提供剪贴板历史功能

## 本地化Web应用

安装 [epichrome](https://github.com/dmarmor/epichrome) 将web应用（例如trello）包装为本地应用，方便使用 Mission Control 和全屏。它实际上是每个web应用绑定一个独立的chrome profile，故此还可以安装使用chrome插件。

## 一套键盘鼠标控制两台电脑

Mac是台式机，键盘鼠标连接到Mac上，安装SynergyKM，运行为server；Win笔记本安装Synergy（网上寻找第三方编译版本，open source，但官网下载binary要收费），运行为client。

使用Windows键盘，需要重新映射按键。首先是在 Preference - Keyboard - Modifier Keys 中对调 Option 和 Command 键，这样左下角三个键的位置就跟Mac的键位相同了。然后安装[Karabiner](https://pqrs.org/osx/karabiner/)，一个功能超强的键盘映射工具，在profile中选择enable需要的映射规则。

因为用 Synergy 同时控制Win电脑，这时重映射过的键盘在 Windows 里面就很难用了。Karabiner 支持多个profile，但目前还没有根据所在屏幕自动切换的方案，解决方法是创建一个NoRemap的profile，里面不打开任何remap规则，用F6键做profile切换。

Profile切换需要在Karabiner中增加自定义规则，在`private.xml`中加入下面的内容：

``` xml
<vkopenurldef>
    <name>KeyCode::VK_OPEN_URL_SHELL_switchprofile_default</name>
    <url type="shell">
        <![CDATA[    /Applications/Karabiner.app/Contents/Library/bin/karabiner select_by_name Default    ]]>
    </url>
</vkopenurldef>
<vkopenurldef>
    <name>KeyCode::VK_OPEN_URL_SHELL_switchprofile_noremap</name>
    <url type="shell">
        <![CDATA[    /Applications/Karabiner.app/Contents/Library/bin/karabiner select_by_name NoRemap    ]]>
    </url>
</vkopenurldef>
<item>
    <name>Switch Profile to Default with F6</name>
    <identifier>private.switch1</identifier>
    <autogen>
        __KeyToKey__
        KeyCode::F6,
        KeyCode::VK_OPEN_URL_SHELL_switchprofile_default
    </autogen>
</item>
<item>
    <name>Switch Profile to NoRemap with F6</name>
    <identifier>private.switch2</identifier>
    <autogen>
        __KeyToKey__
        KeyCode::F6,
        KeyCode::VK_OPEN_URL_SHELL_switchprofile_noremap
    </autogen>
</item>
```

参考自：[http://apple.stackexchange.com/a/143064/13455](http://apple.stackexchange.com/a/143064/13455)