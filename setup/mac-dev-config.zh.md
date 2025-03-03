---
title: Mac 开发环境配置
slug: mac-dev-config
date: '2025-05-04 18:06:12+08:00'
lastmod: '2025-05-04 18:06:12+08:00'
authors: zwyyy456
comments: true
draft: false
showToc: true
hidemeta: false
disableShare: true
showbreadcrumbs: false
categories:
  - setup
tags:
  - geek
  - tips
---





## 前言

本文主要记录新到手 mac 之后，如何配置 mac 以满足自己的需求，以便后续新 mac 到手或者重装系统再配置 Mac。

> m4 芯片的 Mac mini 到手后，估计两年内不会再买新 Mac 了。

## 安装 Homebrew

执行 `/bin/bash -c "$(curl -fsSL https://github.com/Homebrew/install/raw/HEAD/install.sh)"`​ 即可，如果因为网络问题，无法下载安装脚本，可以执行 `/bin/bash -c "$(curl -fsSL https://mirrors.ustc.edu.cn/misc/brew-install.sh)"`​。

## 配置环境变量

mac 自带了 git，因此执行 `rm -rf ~/.config && git clone https://github.com/zwyyy456/dotfile.git ~/.config`​，然后执行 `echo 'ZDOTDIR=$HOME/.config/zsh' > $HOME/.zshenv`​ 再重新打开终端即可。

在 `~/.config/zsh/rc.d/04-env.sh`​ 中，已经通过设置 homebrew 环境变量，使其使用中科大源了：

```sh
  export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.ustc.edu.cn/brew.git"
  export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.ustc.edu.cn/homebrew-core.git"
```

## 通过 Homebrew 安装软件

大名鼎鼎的 Homebrew 无需多言，执行以下命令以安装我所需要的软件：

```zsh
brew install fish neovim tree lazygit cmake ninja fzf tmux lsd
brew install trae wezterm visual-studio-code neovide orbstack 
brew install appcleaner font-lxgw-wenkai mac-mouse-fix keepassxc raycast
brew install floorp microsoft-edge siyuan zotero betterdisplay
brew install iina localsend miniconda
```

总的原则大概就是能用 homebrew 安装的都是使用 homebrew 安装，例如 Edge、VScode 等。

要注意的是，Homebrew 安装软件时，将软件分成了 Formulae 和 Casks 两大类，简单来说就是，有 gui 的就是 Casks，没有的就是 Formulae，安装带 GUI 的软件，例如 Neovide 时，完整安装指令为 `brew install neovide --cask`​。

> 现在 homebrew 已经不需要添加 `--cask`​ 也能安装有 gui 的软件了。

Homebrew 还可以用于安装字体，例如执行 `brew install font-lxgw-wenkai`​ 即可安装霞鹜文楷字体。

## 软件配置

上述通过 homebrew 安装的软件中，trae 是字节出的类 cursor 软件，打开时可以选择从 vscode 导入配置，而 vscode 的设置可以在登陆微软帐号之后自动同步。注意 trae 不支持 pylance，可以使用 pyright 替代。需要执行以下命令，使得安装 vscode.neovim 后，按住 j、k 等键，光标会持续跟随移动。

```sh
defaults write com.trae.app ApplePressAndHoldEnabled -bool false
defaults write com.microsoft.VSCode ApplePressAndHoldEnabled -bool false
```

raycast 我主要使用剪贴板和窗口管理功能，可以给剪贴板设置别名为 `cb`​，并设置快捷键为 `<C-v>`​。

![image](https://pic-upyun.zwyyy456.tech/siyuan20250305122658.pngnull)

思源笔记则是登录帐号，从 iCloud 导入 s3 的 json 配置文件，然后启用插件，将日间与夜间模式的主题都设置为 `savor`​ ，并将编辑器字体设置为霞鹜文楷即可。

neovim 在 `~/.config`​ 文件中早已配置好 LazyVim，执行 `nvim`​ 就会自动下载安装相关插件（注意网络环境问题）。详见 [Neovim 的配置与使用](https://blog.zwyyy456.tech/zh/posts/blog/neovim_tutorial/)。

miniconda 在 `~/.config`​ 中也已经配置好环境变量，注意执行 `conda config --set changeps1 False`​，否则 fish 右侧会显示两次当前 conda 环境。

betterdisplay 可以让 2k 屏也支持 hidpi，可以选择一个合适的支持 hidpi 的分辨率，对于 2560undefined 分辨率的屏幕，可以直接设置缩放到 1920undefined。

在 `.config`​ 的配置文件中，fish 的大多数配置已经完成了，插件也自动安装了，可以再设置一下 fish 的高亮颜色，执行 `fish_config`​ 会打开一个网页，可以设置 fish 的颜色高亮，这里选择 `Base16 Eighties`​，然后点击 set theme，最后终端中输入回车即可。

mac 还需要创建 ssh 密钥对（当然，复制其他电脑的密钥对过来也不是不行，省的再往 GitHub 去添加了），执行 `ssh-keygen -t ed25519 -C "zwyyy456@hotmail.com"`​，然后一路 Enter 即可，可以选择输入密码，这样的话，需要执行 `ssh-add ~/.ssh/id_ed25519`​ 来将私钥添加到 session 缓存，这样终端的当前 tab，或者说 session 中，后面需要用到 ssh 的密钥的操作就不会再需要输入密码了。

tmux 需要创建 `~/.tmux.conf`​，往其中写入 `source-file ~/.config/tmux/tmux.conf`​，`~/.config/tmux/tmux.conf`​ 中内容如下：

```sh
set-environment -gu LESS
set -s set-clipboard external
set -sa terminal-overrides ",alacritty*:Tc,xterm*:Tc,gnome*:Tc,screen*:Tc"
```

，从而使得 tmux 启动时，先清空 `LESS`​ 环境变量，再加载 zsh 的配置文件。

mac 在通过 homebrew 安装了 gcc 或者 llvm 之后，`clang++`​ 编译时，会找不到 `cstdio`​ 头文件，需要移除 homebrew 安装的 gcc 后重新执行 `xcode-select --install`​。

> 也有可能是之前的 build 文件夹中的 CMake 的 cache 导致的。

[macOS 找不到 stdio.h 头文件](https://stackoverflow.com/questions/51761599/cannot-find-stdio-h)
