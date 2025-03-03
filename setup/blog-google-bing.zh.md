---
title: 使博客被搜索引擎收录
slug: blog-google-bing.zh
date: '2025-03-03 23:28:12+08:00'
lastmod: '2025-03-03 23:28:13+08:00'
tags:
  - hugo
  - blog
categories:
  - setup
keywords: hugo,blog
authors: zwyyy456
comments: true
draft: false
showToc: true
hidemeta: false
disableShare: true
showbreadcrumbs: false
---





## 前言

在网站没有被搜索引擎收录之前，网站内容是无法通过搜索引擎搜索到的，因此我们手动让网站被 Google 和 Bing 收录。

## Google

进入 Google Search Console，点击 `添加资源`，资源类型选择 `网址前缀`，这里我的网址前缀为 `https://blog.zwyyy456.tech/`，验证方法选择通过 `HTML 文件`，按照提示，下载一个 google 开头的 html 文件，将这个文件放在 Hugo 博客文件目录的 `static` 目录下，这是因为 `static` 目录中的文件与目录在编译时，会被复制到 `public` 目录，即网站的根目录。

完成验证之后，下一步则是添加网站地图，如下图所示：

![PIpv3c](https://pic-upyun.zwyyy456.tech/uPic/PIpv3c.png)

Hugo 会自动为我们生成 `sitemap.xml` 文件，即站点地图，由于我的网站支持中英双语，因此站点地图的位置为 `https://blog.zwyyy456.tech/zh/sitemap.xml` 和 `https://blog.zwyyy456.tech/en/sitemap.xml`，如果是单语言的 Hugo 博客网站，则是 `https://HugoExample.com/sitemap.xml`。添加好站点地图之后等待谷歌处理数据即可。

## Bing

进入 Bing Webmaster Tools，可以直接从 Google Search Console 导入网站，提交站点地图的方法是类似的。
