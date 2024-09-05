---
title: "Neovim 的配置与使用"
date: 2023-03-17T15:08:52+08:00
lastmod: 2024-04-15T15:08:52+08:00 #更新时间
authors: ["zwyyy456"] #作者
categories: ["setup"]
tags: ["vim", "neovim", "geek", "tips"]
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
## 安装 LazyVim
参考 Lazyvim 的官方安装教程即可安装，要求系统已经安装好了 `npm`。

实际上就是 clone [folke](https://github.com/folke) 的适用于 LazyVim 的初始 [配置文件](https://github.com/LazyVim/starter) 到 Neovim 的配置文件所处的目录，Linux 和 macOS 都是 `~/.config`，Windows 比较特殊，位于 `~/AppData/Local/`。

由于我对初始配置文件做了一定的修改，因此我这里直接 clone 我自己的 [配置文件](https://github.com/zwyyy456/nvim_config.git)

> It is recommended to run `:checkhealth` after installation

值得注意的是，LazyVim 会安装 `nvim-treesitter` 插件，而 `nvim-treesitter` 插件会自动编译安装 C/C++ 等语言的解析器，而编译安装是需要 C/C++ 的运行环境的，对安装了 Command Line Tool 的 Mac 或者 Linux 而言，这一步一般不会存在问题，Windows 则是容易出现问题，我在重装 Windows 系统后尝试过只通过 scoop 安装了 `llvm`，尽管命令行中执行 `clang --version` 是有正常输出的，即 C/C++ 运行环境已经正常安装好了，但是 `nvim-treesitter` 始终无法正常编译解释器，不得已，我又通过 `scoop install mingw` 安装了 `mingw`（gcc），安装好 `mingw` 之后，解释器就能正常被编译安装了，此后执行 `scoop uninstall mingw` 卸载掉 `mingw` 也还是能正常使用。

## keymap 配置

**Insert**模式下按下`jk`退出**Insert**模式，将`nvim/lua/config/keymap.lua`中文件内容修改为：

```lua
local keymap = vim.keymap
keymap.set("i", "jk", "<ESC>")
```

## 同步系统剪贴板

neovim 自带的拷贝文字功能被称为 yank，yank 的结果存放在 neovim 自己的缓冲区，要让这个缓冲区与操作系统的剪贴板同步，即 yank 的结果会存放在操作系统剪贴板，而 paste 的时候会从操作系统剪贴板粘贴，那么需要修改 `nvim/lua/config/options.lua`，追加内容： 

```lua
local opt = vim.opt
opt.clipboard = "unnamedplus" -- Sync with system clipboard
```

并执行 `sudo apt install xclip` 以安装 `xclip` 从而让系统拥有剪贴板功能。

## 利用 lazy.nvim 安装并管理插件

nvim 的配置文件目录结构如下：

```txt
├── comment.lua
├── init.lua
├── lazy-lock.json
├── lazyvim.json
├── LICENSE
├── lua
│   ├── config
│   │   ├── autocmds.lua
│   │   ├── keymaps.lua
│   │   ├── lazy.lua
│   │   └── options.lua
│   ├── plugins
│   │   ├── colorscheme.lua
│   │   ├── disabled.lua
│   │   └── example.lua
│   └── vscode
│       ├── config
│       │   ├── keymaps.lua
│       │   ├── lazy.lua
│       │   └── options.lua
│       └── plugins
│           ├── comment.lua
│           ├── disabled.lua
│           └── mini.lua
├── README.md
└── stylua.toml
```

以 [nvim-osc52](https://github.com/ojroques/nvim-osc52) 插件为例，我们首先在 `nvim/lua/plugins/` 目录下创建文件 `osc52.lua`，内容修改为

```lua
return {
  {
    "ojroques/nvim-osc52",
    lazy = false,
    opts = {tmux_passthrough = true},
  },
}
```

由于 `nvim/lua/config/lazy.lua` 中内容如下

```lua
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  -- bootstrap lazy.nvim
  -- stylua: ignore
  vim.fn.system({ "git", "clone", "--filter=blob:none", "https://github.com/folke/lazy.nvim.git", "--branch=stable", lazypath })
end
vim.opt.rtp:prepend(vim.env.LAZY or lazypath)

require("lazy").setup({
  spec = {
    -- add LazyVim and import its plugins
    { "LazyVim/LazyVim", import = "lazyvim.plugins" },
    -- import any extras modules here
    -- { import = "lazyvim.plugins.extras.lang.typescript" },
    -- { import = "lazyvim.plugins.extras.lang.json" },
    -- { import = "lazyvim.plugins.extras.ui.mini-animate" },
    -- import/override with your plugins
    { import = "plugins" },
  },
  defaults = {
    -- By default, only LazyVim plugins will be lazy-loaded. Your custom plugins will load during startup.
    -- If you know what you're doing, you can set this to `true` to have all your custom plugins lazy-loaded by default.
    lazy = false,
    -- It's recommended to leave version=false for now, since a lot the plugin that support versioning,
    -- have outdated releases, which may break your Neovim install.
    version = false, -- always use the latest git commit
    -- version = "*", -- try installing the latest stable version for plugins that support semver
  },
  install = { colorscheme = { "tokyonight", "habamax" } },
  checker = { enabled = true }, -- automatically check for plugin updates
  performance = {
    rtp = {
      -- disable some rtp plugins
      disabled_plugins = {
        "gzip",
        -- "matchit",
        -- "matchparen",
        -- "netrwPlugin",
        "tarPlugin",
        "tohtml",
        "tutor",
        "zipPlugin",
      },
    },
  },
})
```

其中前 8 行是用于安装 `lazy.nvim` 插件本身的，而第 9 行 `require("lazy").setup({spec = {{import = "plugins"},},})`（省略掉了其他语句），会安装并运行 `lua/plugins/` 目录下的所有插件。

如果不安装 LazyVim，只是安装 `lazy.nvim` 插件，那么 `nvim/lua/config/lazy.lua` 的内容应该如下：

```lua
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", -- latest stable release
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

require("lazy").setup("plugins")
```

最后一行 `require("lazy").setup("plugins")` 的功能也是安装并加载 `lua/plugins/` 目录下的所有插件。

要安装 `Comment.nvim` 插件，只需要在 `nvim/lua/plugins/` 目录下新建 `comment.lua` 插件，内容修改为：

```lua
return {
  {
    "numToStr/Comment.nvim",
    lazy = false,
  },
}
```

即可。


## ssh 下剪贴板同步

即要求 `ssh` 连接到远程服务器，在远程服务器打开 `nvim` 时，`nvim` 复制的内容会自动同步到本地电脑的剪贴板。

从 neovim 0.10.0 开始，neovim 自带了 osc52 插件，因此不再需要手动安装插件，只需要在 `~/.config/nvim/lua/config/options.lua` 中追加以下内容即可：

```lua
function my_paste(reg)
  return function(lines)
    --[ 返回 “” 寄存器的内容，用来作为 p 操作符的粘贴物 ]
    local content = vim.fn.getreg('"')
    return vim.split(content, "\n")
  end
end

if vim.env.SSH_TTY then
  --[ 当前环境为本地环境，也包括 wsl ]
  opt.clipboard:append("unnamedplus")
  vim.g.clipboard = {
    name = "OSC 52",
    copy = {
      ["+"] = require("vim.ui.clipboard.osc52").copy("+"),
      ["*"] = require("vim.ui.clipboard.osc52").copy("*"),
    },
    paste = {
      --[ 小括号里面的内容可能是毫无意义的，但是保持原样可能看起来更好一点 ]
      ["+"] = my_paste("+"),
      ["*"] = my_paste("*"),
    },
  }
end
```

> 本小节之后的内容已经不再需要。

在配置文件 `nvim/lua/config/` 中创建文件：`osc52.lua`，追加内容为：

```lua
local function copy(lines, _)
  require('osc52').copy(table.concat(lines, '\n'))
end

local function paste()
  return {vim.fn.split(vim.fn.getreg(''), '\n'), vim.fn.getregtype('')}
end

vim.g.clipboard = {
  name = 'osc52',
  copy = {['+'] = copy, ['*'] = copy},
  paste = {['+'] = paste, ['*'] = paste},
}

-- Now the '+' register will copy to system clipboard using OSC52
vim.keymap.set('n', '<leader>c', '"+y')
vim.keymap.set('n', '<leader>cc', '"+yy')
```

使用 `lazy.nvim` 插件安装 [nvim-osc52](https://github.com/ojroques/nvim-osc52) 插件。

最后，在 `init.lua` 中，添加内容：`require("vscode.config.lazy")`，故 `init.lua` 内容如下：

```lua
if vim.g.vscode then
  -- if true then
  -- Vscoe extension
  require("vscode.config.options")
  require("vscode.config.keymaps")
  -- require("vscode.init")
  require("vscode.config.lazy")
  -- require("vscode.plugins.disabled")
else
  -- bootstrap lazy.nvim, LazyVim and your plugins
  require("config.lazy")
  require("config.osc52")
end
```

这里的 `if, else` 是为了兼容 `scode-neovim` 插件。

> Lazyvim 插件会自动加载 `nvim/lua/config/` 目录下的 `autocmds.lua`、`keymaps.lua`、`lazy.lua`、`options.lua` 这几个文件，目录下的其他配置文件的内容需要我们手动在 `init.lua` 中选择加载。

## ssh + tmux 的剪贴板同步

如果远程服务器打开了 tmux，并在 tmux 中打开 neovim，那么还需要做一点额外配置，使得 neovim yank 的内容能同步到本地机器的剪贴板。

在 `~/.tmux.conf` 中，添加 `set -s set-clipboard on` 即可。

> 本节后续内容已不再需要。

首先，编辑 `~/.tmux.conf`，追加内容 `set -g allow-passthrough on`，在安装 `osc52` 插件时，配置需要设置为 `tmux_passthrough = true`，对应“利用 lazy.nvim 安装并管理插件”部分的 `opts = {tmux_passthrough = true},`。


## 设置 tab-size 为 4
修改`~/.config/nvim/lua/config/optionas.lua`，追加内容
```lua
opt.tabstop = 4
opt.shiftwidth = 4
```

## LSP 设置

进入 neovim 之后输入 `:Mason`，即可像使用 LazyVim 一样安装 LSP 所需的插件，我安装了以下这几个插件：

![ZRzDt7gAhmpJuNH](https://pic-upyun.zwyyy456.tech/smms/2023-12-26-065724.png)

## Neovim 插件介绍

### mini.ai

没搞懂干嘛的。

### mini.surround

没搞懂，待续。

### flash.nvim

功能有点像浏览器 vimium 插件按下 f 的时候，例如我按下 `/`，再按下 `tr`，表示我要搜索 `tr`，这时屏幕上会显示一系列的 label（字母），表示 `tr` 所处的不同位置，按下对应的字母，即可跳转到对应的位置。

### mini.pairs

括号，引号等的自动补全，即输入 `(` 会显示 `()`。

## Neovim 快捷键

`gi`，跳转到上次 `insert` 的地方并进入 `insert` 模式。

## vim.repeat




## 杂项
`<S-Tab>`已经被 lazyvim 设置于为`Go to left windows`

见 github 讨论 [切换 tab](https://github.com/LazyVim/LazyVim/discussions/332)



## 备份
文件存档于[github-nvim-config](https://github.com/zwyyy456/nvim_config)，理论上安装`lazyvim`之后`pull`即可。

# 参考

[NeoVintageous-clipboard](https://github.com/NeoVintageous/NeoVintageous/issues/829)

[NeoVintageous-jk](https://github.com/NeoVintageous/NeoVintageous/issues/815)

[Vscode.neovim](https://github.com/vscode-neovim/vscode-neovim)

[Vscode.neovim-gc-comment](https://github.com/vscode-neovim/vscode-neovim/issues/199)


