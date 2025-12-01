---
title: "使用 vim 开发 iOS/macOS 应用"
date: 2025-10-14T23:34:14+08:00
lastmod: 2025-10-14T23:34:14+08:00 #更新时间
authors: ["zwyyy456"] #作者
categories: ["setup"]
tags: ["vim", "swift"]
series: []
description: "" #描述
weight: # 输入 1 可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true #是否展示评论
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: false #顶部显示当前路径
---
## 前言
Vim 真是令人上瘾，习惯了 Vim 之后，总想把一切快捷键都改成 Vim 模式，Xcode 最一开始也有 XVim 这个比较完善的插件，但是后来由于苹果的限制，Xcode 已经不能再安装插件了，尽管苹果给 Xcode 添加了 Vim 模式，但是非常残废，无法自定义快捷键，连 `:w` 保存都不能用，加之 Xcode 又大又重，也很不符合我的使用习惯，因此，我就开始寻找一个既能使用 Swift 开发 iOS/macOS 原生应用，又能尽量减少对 Xcode 的依赖的开发方案，得益于微软推出的 LSP 协议与 VsCode 的蓬勃发展，还真有一个使用 VsCode 开发 iOS/macOS 原生应用的可行方案，并且 Xcode 的 View 的实时渲染也能通过 InjectionIII 实现，具体方案可以参考 [在 Cursor 打造高效 iOS 开发环境](https://blog.imjp.uk/fxxk-xcode)。

受韦一笑老师的启发，加之只有只有 Vim 的最新版还能支持 Win7（neovim 早已不支持），我最近一直在捣鼓如何将 Vim 打造成完全符合自己需求的代码编辑器，干脆一不做、二不休，尝试使用 Vim 来写 Swift、开发 iOS 应用，于是就有了这篇文章。

## 插件安装

利用 Vim 写 Swift，实际上必装的插件就是 [yegappan/lsp](https://github.com/yegappan/lsp), 该插件是使用 Vim9 Script 开发的，相比 Coc 与 vim-lsp 更轻巧、性能更好，自带补全功能，无需额外安装插件。通过 Plug 安装该插件很简单，在 `.vimrc` 中添加 `Plug 'yegappan/lsp'` 即可，然后执行 `:PlugInstall` 安装。

`yegappan/lsp` 插件的的配置方案我是参考的韦一笑老师的 [yegappan 配置](https://github.com/skywind3000/vim/blob/master/site/bundle/yegappan.vim)，在 `~/.vimrc` 中需要 source 该文件以及一个写了 lsp server 配置的文件，事实上我的 vim 配置文件也基本是参考（照抄）他的，也就改了部分快捷键，这里要注意的就是如果希望修改自动补全的方案，需要 `LspSetup` 事件触发后再修改，否则会被 `yegappan/lsp` 插件覆盖，如下所示：

```vim
augroup LspSetup
  au!
  au User LspAttached set completeopt-=noselect
augroup END
```

至于 lsp server 的配置，可以参考 yegappan/lsp 的 [wiki](https://github.com/yegappan/lsp/wiki)，里面有 sourcekit-lsp（swift 的 lsp server）的配置，我最一开始参考的 nvim-lspconfig 的 sourcekit.lua，然而不行，nvim 的这个感觉是给 Windows 与 Linux 上单独写 swift 用的，mac 安装 Xcode 或者 Xcode 命令行工具附带的 sourcekit-lsp 并不在环境变量中，我的 sourcekit lsp server 的配置如下：

```vim
if has('mac') && executable('xcodebuild')
    " let g:sourcekit_lsp_path = trim(system('xcrun --find sourcekit-lsp'))
    let g:lsp_servers.sourcekit_lsp = #{
                \ filetype: ['swift', 'objc', 'objcpp'],
                \ path: 'xcrun',
                \ args: ['sourcekit-lsp'],
                \ root: ['.git', '.svn', 'buildServer.json', 'Package.swift']
                \ }
endif
```

详细配置可以参考我的 [dotfile](https://cnb.cool/zwyyy456/dotfile) 中的 Vim 部分。

还有一点要注意的是，如果先安装了命令行工具，后安装的 Xcode，xcode-select 指向的可能是命令行工具而非完整的 Xcode App，因此需要执行 `sudo xcode-select -s /Applications/Xcode.app/Contents/Developer` 将 xcode-select 指向完整的 Xcode App。

## 生成 Swift 配置数据库

就像 clangd 需要 `compile_commands.json`，sourcekit-lsp 也需要 `buildServer.json` 文件来理解项目，项目根目录下根据项目类型执行以下其中一条命令可以生成 `buildServer.json` 文件：

```bash
xcode-build-server config -workspace <xxx>.xcworkspace -scheme <XXX> 
xcode-build-server config -project <xxx>.xcodeproj -scheme <XXX>
```

`xcode-build-server` 可以使用 homebrew 安装。具体有哪些 scheme 可以在 Xcode 中查看。

对于 swift package，xcode build server 并不能很好的支持，反而 swift 本身已经支持得很好了，直接 swift build 即可。 

## 语法高亮

Vim 的语法高亮引擎已经十分陈旧了，对 Swift 的解析支持非常差劲，这里我们使用 LSP 的语义高亮来替代，注意前面 `yegappan.vim` 中要设置 `\   semanticHighlight: v:true,` 来开启语义高亮，在你的 vim 配置文件中，再添加以下内容：

```vim
" yegappan/lsp highlight support
hi! link LspSemanticNamespace SublimePink
hi! link LspSemanticType SublimeAqua
hi! link LspSemanticClass SublimeAqua
hi! link LspSemanticEnum SublimeAqua
hi! link LspSemanticInterface SublimeGreen
hi! link LspSemanticStruct SublimeAqua
hi! link LspSemanticTypeParameter SublimeAqua
hi! link LspSemanticParameter SublimeOrange
hi! link LspSemanticVariable SublimeWhite
hi! link LspSemanticProperty SublimeWhite
hi! link LspSemanticEnumMember SublimePurple
hi! link LspSemanticEvent SublmeGreen
hi! link LspSemanticFunction SublimeGreen
hi! link LspSemanticMethod SublimeGreen
hi! link LspSemanticMacro SublimeGreen
hi! link LspSemanticKeyword SublimePink
hi! link LspSemanticModifier SublimePink
hi! link LspSemanticComment SublimeGrey
hi! link LspSemanticString SublimeYellow
hi! link LspSemanticNumber SublimePurple
hi! link LspSemanticRegexp SublimeYellow
hi! link LspSemanticOperator SublimePink
hi! link LspSemanticDecorator SublimePink
```

即可实现基于 LSP 的语义高亮，我用的是 Monokai 主题，有自己喜欢的主题可以自定义颜色，也可以使用支持 LSP 语义高亮的主题。

## 通过 InjectionNext 实现 View 的实时渲染

升级到 macOS 26 之后，原先的 InjectionIII 已经不能用了，但是作者又推出了 InjectionNext 来实现实时渲染。具体方案如下：

1. 下载并安装 App，然后 进入 `Build Settings -> Linking -> Other Linker Flags`。为 Debug 配置添加两个标志（注意：分两行写）：
```sh
-Xlinker
-interposable
```
2. 完全退出 Xcode App，运行 InjectionNext，从菜单选择 Lauch Xcode 以启动 Xcode；
3. 在 Xcode 项目中，通过 `File > Add Package Dependencies...` 添加 InjectionNext 的 Swift 包，添加 HotSwiftUI 和 InjectionNext 这两个包，注意都需要先 clone 到本地，再通过 Add local package 安装；
4. 在 InjectionNext 的菜单项中，选择 `Preapre SwiftUI -> Entire Project`，然后在 Xcode 中运行项目，修改 View 的内容，然后 <D-s> 保存，就能实时看到修改了；
5. 如果希望在外部编辑器例如 Vim 中修改了代码并保存后，也能实时更新 View，需要在 InjectionNext 的菜单中改为 watch project，project directory 设置为项目的根目录。

> 最好先在 Xcode 中右键项目，在 Group 选项处把 group 转换为 folder。

## 结语

这样一番配置好之后，创建一个 xcode 项目，用 Vim 编辑对应的 swift 文件，高亮、定义跳转、引用查找、自动补全等功能都正常了，可以愉快的用 Swift 来写 iOS 应用了。

## 参考

- [在 Cursor 打造高效 iOS 开发环境](https://blog.imjp.uk/fxxk-xcode)
- [sweetpad](https://sweetpad.hyzyla.dev/docs/autocomplete)
- [InjectionIII](https://github.com/johnno1962/InjectionIII)

