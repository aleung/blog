---
title: Microservice development environment make easy with Docker
date: 2017-09-07 13:37:23
tags:
  - SoftwareDev
---

正在负责的产品线已经全面转为基于 node.js 技术栈微服务架构，为了提高开发效率，我希望能创建一个标准的、容易创建的开发环境，以改善开发环境之间不一致、可能不完备、创建配置麻烦的现状。

对这个开发环境的要求是 lightweight，reproducible 和 portable。

<!-- more -->

开始尝试的是 [Vagrant](https://www.vagrantup.com/)，但感觉缺点还是不少：provision 过程麻烦了些；如果要同时进行不同的开发就要运行多个VM，占用资源比较多；而且有个关键问题没找到好的解决方法：Windows 的文件系统不支持符号链接 (symbolic link)，而 npm install 时会依赖于符号链接，VM 使用 Windows host 的共享目录时在里面执行 npm install 会出错。

这时 docker 进入了视野，容器的轻量、自包含一切的特性很满足我对标准化开发环境的要求。经过几天折腾，搞出了这样一套环境，初步试用还比较满意。

这个基于 docker 的开发环境做到了这些：

* 可以运行于任何操作系统、任意主机上，只需要安装了 docker。我们用到的环境包括每个开发者的 PC、远程的公共开发机和测试 lab 里的主机，操作系统包括 Win7、Linux 和 macOS。
* 在满足要求的机器上（装好 docker 和其他必须的软件），敲一个命令，在一分钟内就可以从无到有创建出一个开发环境。
* 开发环境中所有东西都是预定义和存储于公共仓库（包括 git，docker镜像和 npm 模块仓库）中，每个人在任何机器上创建出来的环境都会是一摸一样的。
* 定义出一套标准的、可重用的环境，统一各个微服务项目的开发体验。

这个开发环境覆盖了一个微服务的日常开发流程，包括编码、构建、质量检查、单元测试等等。但不含括要求跨服务集成的测试，例如系统测试、端到端测试。

环境包括预先创建的用于开发、测试的多个docker镜像。其中最主要的是 node.js 环境的镜像，其中包括了基本的 Linux shell 环境和 node.js 开发运行用到的软件。其他镜像是单元测试中需要依赖到的工具。

{%img /attachments/2017/9/images.png %}

每个微服务有自己独立的开发环境，通过 `docker-compose` 启动起来。被开发的服务在 node.js 容器中构建和测试，源代码放在host文件系统中，通过 volume与容器共享，这样开发者可以在 host 上随意使用喜欢的编辑器修改代码，也绕开容器没法运行图形界面应用的问题。根据需求，将单元测试要用到的容器 link 到 node.js 容器，构造出测试环境。

{%img /attachments/2017/9/microservice-dev.png %}

于微服务开发，是不需要暴露任何端口出来的，因为测试过程都在容器中。对于 web 应用开发，要从容器中映射出 http 端口供 host 的浏览器访问，如果需要做 web debug，还可以将 debug 端口暴露出来。

一个 host 上可以同时进行多个项目的开发，环境完全独立，相互并不干扰。

{%img /attachments/2017/9/isolated-env.png %}

在调试这个环境的过程中，最麻烦的是 Win7 上运行 docker。在 Win10 之前的环境中运行 docker，当某些特殊字符在控制台上显示时，有可能引发 docker 出现 "New state of 'nil' is invalid" 的错误。详细情况在 docker issues [#21323](https://github.com/moby/moby/issues/21323) 中 [#22345](https://github.com/moby/moby/issues/22345) 有描述。后来找到变通方案：安装  [ConEmu](http://conemu.github.io/en/Downloads.html) 或者 [Cmder](http://cmder.net/)，并且将 ConEmu 设置为缺省终端，替换掉 Windows 自带的终端，就可以避免错误的发生。

Windows 环境里还有一个限制：只有 `C:/Users/` 下面的目录才能被挂载成 docker volume，所以源代码目录必须放在 `C:/Users/{your_id}/` 之下.

一个开发环境需要的容器以及相关配置都定义在 `docker-compose.yml` 文件中了，以下为一个例子：

```yaml
version: "3"

services:
  target:
    hostname: foo-service    # set it to your project name
    image:  internal.docker.hub/dev/nodejs
    volumes:
      - ${PWD}:/project
      - ~/.cache/yarn:/usr/local/share/.cache/yarn
    depends_on:
      - postgres
    environment:
      PgDatabaseAddress: postgres

  postgres:
    image: internal.docker.hub/dev/postgres
    environment:
      POSTGRES_PASSWORD: test
      POSTGRES_USER: test
```

另外，对于 node.js 项目，我们也定义了统一的 `package.json` 模版，里面包含了开发过程各种操作的 npm scripts，包括 TypeScript 编译、代码格式化、tslint检查、单元测试、文档生成等等。

要在一台机器上开始某个项目的开发，要做的事情只需要：

1. 将源代码 git clone 的本地目录，里面包括了定义docker环境的  `docker-compose.yml` 和定义 node.js 项目的 `package.json`。
2. 运行 `docker-compose up --rm target`，启动 docker 环境。启动时会自动安装所有 npm 依赖。
3. 这时终端会进入到容器的 bash 中，在这儿就可以像在本地环境一样进行开发操作了，例如：`yarn build`, `yarn test`。

