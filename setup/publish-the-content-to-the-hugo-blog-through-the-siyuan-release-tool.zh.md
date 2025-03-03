---
title: 通过思源发布工具将内容发布到 Hugo
slug: publish-the-content-to-the-hugo-blog-through-the-siyuan-release-tool.zh
date: '2024-11-21 23:32:04+08:00'
lastmod: '2025-03-03 23:23:04+08:00'
tags:
  - hugo
  - siyuan
categories:
  - setup
keywords: hugo,siyuan
authors: zwyyy456
comments: true
draft: false
showToc: true
hidemeta: false
disableShare: true
showbreadcrumbs: false
---





## 前期准备

首先，安装好思源笔记与思源发布工具插件，以及安装 picgo 插件并配置好图床（此项为建议，非必须）。

## 发布设置

点击 `发布工具->通用设置->发布设置`​，在插件商店中，选择 `Github`​ 下的 `Hugo`​，之后会弹出一个 `添加 Hugo 平台`​ 的页面，点击提交即可。然后点击 `发布工具->通用设置->发布设置`​ 下的发布配置，能看到会有一个已启用的 Hugo 的配置项，点击旁边的设置按钮，就进入了如下的设置页面：

其中，仓库名是你的 GitHub 中的对应仓库的仓库名，我这里是 `posts`​，作为 submodule 添加到我的 hugo 博客的仓库中，`存储目录`​ 这里其实不太关键，只有在点击一键发布的时候，才会被用到，我这里会默认选择 `tech`​，与个人设置有关，如果是发布到 Hugo 博客，那么这个发布工具做的实质上就是将本篇文章转换成 `md`​ 文件，添加到 GitHub 的对应的仓库的对应目录下，`文件规则`​ 对应的是生成 `.md`​ 文件的文件名的规则。支持设置为 `[filename].md`​ 以及 `[slug].md`​ 等，其中 `filename`​ 表示的是显示在思源左侧的文档树中的文件名，`slug`​ 则是本工具会根据文件名的英文翻译加上随机字符串自动生成的别名。

![image](https://pic-upyun.zwyyy456.tech/siyuan20250226233348.pngnull)

文章预览规则和预览规则可以先不管，暂时保持默认即可，发布目录也保持 `tech`​。

> 根据测试结果，`.md`​ 文件在选择一键发布时，会被直接发布到 `存储目录`​ 对应的目录，而 `发布目录`​ 默认与存储目录保持一致，这里是一个无效的选项。

鉴权 `token`​ 是点击 `token`​ 生成地址去生成的 api token，点击生成 token 的时候选择 `classic`​ 的那一个（这样可以选择 token 的有效期为永久，权限只要把 repo 那一栏都勾选上即可）。

取消勾选 `yaml 永久链接`​ 后，Front Matter 中不会自动生成 `url`​ 对应键值对，`yaml 预设配置`​ 的 json 键值对一一对应默认生成的 yaml 格式的 Front Matter 的内容，但是 `slug`​ 对应键值对还是会默认生成（源码中写死的），`date`​ 与 `lastmod`​ 这两个键值对也还是会默认生成。

```json
{
    "authors": "zwyyy456",
    "comments": true,
    "draft": false,
    "showToc": true,
    "hidemeta": false,
    "disableShare": true,
    "showbreadcrumbs": false
}
```

## 发布工具偏好设置

我的发布工具的 `偏好设置`​ 的设置如下：

![image](https://pic-upyun.zwyyy456.tech/siyuan20250226233920.pngnull)

建议勾选 `去除正文 h1`​，否则生成的文章中，会生成一个文章名对应的一级标题，其中勾选 `显示文章管理菜单`​，则点击发布工具的按钮时，会有 `仪表盘`​ 选项，其余选项是否勾选建议自行决定。

点击发布工具的一键发布后，文章对应的 `md`​ 会被上传到对应的目录中，这时，我们在思源中，查看该文件的 `更多 -> 属性`​，在自定义中，可以看到有一个 `github-Hugo-yaml`​ 文件，内容如下，插件在执行一键发布时，会自动在 `.md`​ 文件的内容头部添加以下内容，这里的 `slug`​ 与前面的文件规则中的 `[slug].md`​ 是对应的，即生成的 `.md`​ 文件的文件名是 `publish-xxx.md`​ 。

‍

点击发布工具的 `常规发布`​，点击我们的 `Hugo`​ 项，在 `偏好设置`​ 勾选了 `允许修改别名`​ 的情况下，在 `详细发布`​ 中可以修改别名，即 `slug`​，也可以选择要将文件发布到哪个目录，在 `源码模式`​，我们可以修改 hugo 的 yaml 格式的 Front Matter 的内容，还可以修改 `.md`​ 文件的内容，而 `同步修改到思源笔记`​ 按钮可以将我们的修改同步到思源笔记的文章中。

> 在源码模式下，修改了 Front Matter 的内容，则在详细发布中，设置文章的标签以及分类将不再影响 Front Matter，需要完全自己维护。

## 将思源中的文档与 hugo 中已发布的文章绑定

点击 `偏好设置`​ -> `文章绑定`​，对于已经通过思源发布工具发布的文章，其内容如下图所示：

![image](https://pic-upyun.zwyyy456.tech/siyuan20250303232357.pngnull)

因此，如果想将思源中的文档与 hugo 中已经发布的文章绑定，则在思源发布工具中打开该文档，点击 `偏好设置 -> 文章绑定`​，`Hugo`​ 这一栏，填写 md 文件在仓库的完整路径。

‍
