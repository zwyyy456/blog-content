---
title: 借助 cnb.cool 进行远程开发
slug: remote-development-with-cnbcool
date: '2025-02-24 22:34:26+08:00'
lastmod: '2025-02-26 22:42:14+08:00'
authors: zwyyy456
comments: true
draft: false
showToc: true
hidemeta: false
disableShare: true
showbreadcrumbs: false
---





## 前言：为什么要远程开发

当我读研的时候，买了性能在当时还是挺强的 m1pro 的 MacBook，放实验室用，宿舍则是但由于我开发的程序一般在 Linux 上运行，因此，我一般都是在 mac 上通过 vscode remote 远程连接到实验室的服务器上进行开发，这样有一个很大的好处——无需同步代码。

vscode 的 remote 插件在我看来，简直是 vscode 的杀手级特性，它让远程连接到服务器进行开发变得丝滑且简便。远程开发的好处是显而易见的，本地机器性能要求低，能正常跑 vscode 就行，并且无需操心代码同步问题。我读研的时候，一般是远程连接到实验室的服务器上进行开发。

> 另一种丝滑但极客的方案是直接在终端中 ssh 连接到远程机器并使用配置好的 neovim。

## 远程开发方案的探索

当我毕业后，自然是无法再连接到实验室的服务器进行远程开发以及编译程序了。事实上，我设想过很多备用方案，例如买一台云服务器来进行远程开发，但这样的问题在于，便宜、稳定低延迟、高性能，属于一个不可能三角；后面准备采用的解决方案是自己组一个高性能的 nas，配合 tailscale 或者 frp 即可实现远程开发。事实上，我确实这么干了，但当我买了后，我才发现，在有 115 会员的前提下，我并不需要一台大容量的 nas，之前买的小主机搭配已有的 4T 的移动硬盘对我来说绰绰有余了，加之不想操心 nas 的维护问题（甚至懒得给 nas 装 TrueNAS 系统），后面还是把 nas 给卖了，好在没亏钱。之后就是不需要性能的，就连接到 2c2g 的腾讯轻量云进行远程开发，需要性能的，就直接 mac 本地开发，需要性能且需要在 Linux 上开发的，则是 mac 通过 orbstack 创建一个 Debian 的 Docker 容器，再通过 vscode remote 连接上去进行远程开发，代码同步则通过 git 与 GitHub 进行手动维护（偶尔会忘记同步）。

> 事实上试过通过脚本与 git 进行自动的代码同步，但总觉得有潜在的问题。

## cnb.cool：一个全新且几乎完美的解决方案

其实我是先了解的腾讯的 CODING，他在像 GitHub 那样提供代码托管服务的同时，还提供了云原生构建与开发功能，而 [cnb.cool](https://cnb.cool) 则属于是 CODING 的下一代产品，可以基于 Dockerfile 声明开发环境，和代码一起管理。

在 [cnb.cool](https://cnb.cool) 中，进行远程开发很简单，新建一个仓库，创建好之后，直接点击的 `云原生开发`​ 按钮，即可创建一个开发环境。如果是已经创建好的仓库，点击右上角的 `云原生开发`​ 即可。

![image](https://pic-upyun.zwyyy456.tech/siyuanimage-20250224234822-6sbw4ur.pngnull)

创建好开发环境之后，可以选择使用 WebIDE 或者 VSCode 或者 Cursor 打开，后两者都是基于 vscode 的 remote 插件实现的，WebIDE 则是基于 code-server 实现的网页版 vscode。

![image](https://pic-upyun.zwyyy456.tech/siyuanimage-20250224234844-w27ei9y.pngnull)

构建开发环境，除了第一次耗时可能较久之外，只要不改动 `.ide/Dockerfile`​ 文件，后面创建开发环境都很快，只需要十几秒就可以完成了。其中，默认使用 `cnbcool/default-dev-env:lastest`​ 作为开发环境基础镜像，具体使用啥镜像，可以在 Dockerfile 中声明。

云原生开发核时为 1600，比 CODING 多了 320 核时，公测阶段额外提供 16000 核时。并且，CNB 是按照顶级组织而非用户来计费的，而一个用户可以创建多个组织，因此，理论上无需担心核时不够用。

[cnb.cool](https://cnb.cool) 作为代码托管平台，相关的功能也比较齐全，由于我比较关心的是远程开发相关功能，基本也只是个人业余开发，故在此不多赘述。

## 自定义 cnb.cool 的远程开发环境

前面提到，[cnb.cool](https://cnb.cool) 是基于 `.ide/Dockerfile`​ 来声明开发环境的，因此，可以修改这个 Dockerfile，通过 `From node:20`​ 选定基础的 docker 镜像，通过 `RUN code-server --install extension redhat.vscode-yaml`​ 来安装所需的 vscode 插件，值得注意的是，其插件市场并非 vscode 官方的插件市场，有些插件可能会找不到；同时可以通过 `RUN apt install clangd clang`​ 等来安装对应语言的开发环境。

当然，对我来说，每个仓库都要自行定义一遍 `.ide/Dockerfile`​ 还是有点麻烦，由于我目前主要写 python、go 和 cpp，因此各个代码仓库所需要的开发环境是基本一样的，因此这里我选择直接将对应的开发环境打包成 docker image 传到 [cnb.cool](https://cnb.cool) 即可，这就使用到了它的云原生构建功能，使用起来也很简单：

首先，新建一个代码仓库，我这里命名为 `cnb-dev-env`​，然后在仓库根目录下创建 `.cnb.yml`​ 文件，内容设置为如下：

```yaml
main:
  push:
    - services:
        - docker
      stages:
        - name: docker login
          script: docker login -u ${CNB_TOKEN_USER_NAME} -p "${CNB_TOKEN}" ${CNB_DOCKER_REGISTRY}
        - name: docker build
          script: docker build -f Dockerfile -t ${CNB_DOCKER_REGISTRY}/${CNB_REPO_SLUG_LOWERCASE}:latest .
        - name: docker push
          script: docker push ${CNB_DOCKER_REGISTRY}/${CNB_REPO_SLUG_LOWERCASE}:latest
```

其中，`${}`​ 中的变量都是 [cnb.cool](https://cnb.cool) 默认的内置环境变量，具体可参见 [默认环境变量](https://docs.cnb.cool/zh/build-in-env.html)，其实就是根据当前目录下的 `Dockerfile`​ 来构建镜像，并在每次 push 到远程仓库时，自动构建 Docker 镜像并上传到制品库的 Docker 源地址，例如我这里就是上传到了 `docker.cnb.cool/zwyyy456/cnb-dev-env:latest`​。之后的其他代码仓库中，例如 [double-entry-generator](https://cnb.cool/zwyyy456/double-entry-generator) 只需要将 `.ide/Dockerfile`​ 中内容设置为 `FROM docker.cnb.cool/zwyyy456/cnb-dev-env:latest`​ 即可。

这里要注意的是，这样做之后，第二次点击云原生开发，那么使用的就是使用 `.ide/Dockerfile`​ 构建出来的 dockerfile-cache 缓存镜像，在 `.ide/Dockerfile`​ 的内容不变的情况下，修改 `docker.cnb.cool/zwyyy456/cnb-dev-env:latest`​ 镜像，使用的远程开发环境还是旧的，必须要修改 `.ide/Dockerfile`​ 才会去重新拉取最新的镜像。还有一种方案，则是在代码仓库的根目录下创建 `.cnb.yml`​ 文件，修改其内容为如下，则每次会去拉取新的镜像去创建。

```yaml
# .cnb.yml
$:
  # vscode 事件：专供页面中启动远程开发用
  vscode:
    - docker:
        # 自定义镜像作为开发环境
        image: docker.cnb.cool/zwyyy456/cnb-dev-env:latest
      services:
        - vscode
        - docker
      stages:
        - name: ls
          script: ls -al
```

‍

‍

‍

‍

‍

‍
