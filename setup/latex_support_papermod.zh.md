---
title: "为 papermod 主题添加 Latex 支持"
date: 2022-09-26T11:42:46+08:00
lastmod: 2022-09-26T11:42:46+08:00
tags: ["hugo", "geek", "tips"]
categories: ["setup"]
description: "" #描述
weight: # 输入 1 可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: false #是否展示评论
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: false #顶部显示当前路径
---

## steps to follow
在`themes/PaperMod/layouts/partials`目录下创建`math.html`文件，文件内容如下
```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.2/dist/katex.min.css" integrity="sha384-bYdxxUwYipFNohQlHt0bjN/LCpueqWz13HufFEV1SUatKs1cm4L6fFgCi1jT643X" crossorigin="anonymous">
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.2/dist/katex.min.js" integrity="sha384-Qsn9KnoKISj6dI8g7p1HBlNpVx0I8p1SvlwOldgi3IorMle61nQy4zEahWYtljaz" crossorigin="anonymous"></script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.2/dist/contrib/auto-render.min.js" integrity="sha384-+VBxd3r6XgURycqtZ117nYw44OOcIax56Z4dCRWbxyPt0Koah1uHoK0o4+/RRE05" crossorigin="anonymous"></script>
<script>
    document.addEventListener("DOMContentLoaded", function() {
        renderMathInElement(document.body, {
          // customised options
          // • auto-render specific keys, e.g.:
          delimiters: [
              {left: '$$', right: '$$', display: true},
              {left: '$', right: '$', display: false},
              {left: '\\(', right: '\\)', display: false},
              {left: '\\[', right: '\\]', display: true}
          ],
          // • rendering keys, e.g.:
          throwOnError : false
        });
    });
</script>
```

编辑`themes/PaperMod/layouts/partials`目录下的`extend_head.html`文件，添加如下内容：
```html
{{ if or .Params.math .Site.Params.math }}
{{ partial "math.html" . }}
{{ end }}
```

编辑`config.toml`，添加如下内容
```toml
[params]
        math = true # 添加 latex 支持
```

## Reference
[katex-hugo-papermod](https://rkilingr.me/posts/katex-hugo/)
[math-typesetting](https://adityatelange.github.io/hugo-PaperMod/posts/math-typesetting/)


