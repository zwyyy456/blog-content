---
title: "安装 Debian bookworm 桌面环境"
date: 2022-11-12T09:58:14+08:00
lastmod: 2022-11-12T09:58:14+08:00 #更新时间
authors: ["zwyyy456"] #作者
categories: ["setup"]
tags: ["tips", "geek", "debian"]
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
## 配置语言环境
我在安装debian的时候，有个地方选择了`HK`，因此使用`locale`命令查看当前的区域相关设置时，显示为: ![R46CPdOY8refpnG](https://pic-upyun.zwyyy456.tech/smms/2023-12-26-065709.jpg)

其中:`LANG`为默认的区域设置，该变量的值会覆盖所有未设置的`LC_*`变量的值;

要修改为`en_US`，首先执行`sudo dpkg-reconfigure locales`，然后选中`en_US.UTF-8`，取消`en_HK`那个，取消`inherit`的那个，还可以选中`zh_CN.UTF-8`，这样就启用了`en_US.UTF-8`和`zh_CN.UTF-8`;

由于我的桌面环境为`KDE PLasma`，其语言设置会覆盖`locale.conf`的设置，执行`vim ~/.config/plasma-localerc`，修改该文件为:
```toml
[Formats]
LANG=en_US.UTF-8
LC_ADDRESS=en_US.UTF-8
LC_MEASUREMENT=en_US.UTF-8
LC_MONETARY=en_US.UTF-8
LC_NAME=en_US.UTF-8
LC_NUMERIC=en_US.UTF-8
LC_TELEPHONE=en_US.UTF-8
LC_TIME=en_US.UTF-8

[Translations]
LANGUAGE=en_US:C:zh_CN
```
其中`LANGUAGE`为备用语言，很多软件并未将其英文`locale`设置为`en`或`en_US`，而是使用默认`locale C`。如果在`LANGUAGE`中将非英文`locale`设置到`English`之后，例如`LANGUAGE=en_US:en:es_ES`，那么即使英语字符存在，应用程序可能会选择使用后备`locale`，解决方法是强制在英语`locale`后面设置`C`，例如 `LANGUAGE=en_US:en:C:es_ES`。



## fcitx-rime
安装fcitx-rime
`sudo apt install fcitx-rime`

安装小鹤双拼
`sudo apt-get install librime-data-double-pinyin`

将[小鹤双拼官方网盘](http://flypy.ysepan.com/)，3.1-挂接--音形码，小鹤音形鼠须管for macos里的`default.custom.yaml`复制到`~/.config/fcitx/rime`，并修改为:
```yaml
patch:
  menu:
    page_size: 8
  schema_list:
    - schema: flypy # 添加小鹤音形
    - schema: double_pinyin_flypy
    - schema: luna_pinyin_simp
  key_binder/bindings:
    - when: paging
      accept: bracketleft
      send: Page_Up
    - when: has_menu
      accept: bracketright
      send: Page_Down
    - when: has_menu
      accept: minus
      send: Page_Up
    - when: has_menu
      accept: equal
      send: Page_Down
```
启动`fcitx`，按`Ctrl+Space`切换为`fcitx`，Ctrl+`切换选择输入方案。

## 设置双击打开文件
`Dolphin`默认单击打开文件或者文件夹，点击`System setting->Workspace behavior->General behavior->clicking files or folders`修改为`(select them)`

## Grub
### Grub找不到Windows启动项
这是因为`OS_PROBER`没有默认被启用，要启用`OS_PROBEER`，执行`sudo vim /etc/default/grub`，插入一行`GRUB_DISABLE_OS_PROBER="false"`，然后执行`sudo grub-mkconfig -o /boot/grub/grub.cfg`

### Grub修改默认启动项为windows
执行`sudo vim /etc/default/grub`，将`GRUB_DEFAULT=0`修改为`GRUB_DEFAULT=2`，然后执行`sudo update-grub`

## Wayland下分数缩放模糊的问题
`System settings->Display and monitor`->`Display configuration`->`legacy application(x11)`设置为`Apply scaling themselves`

## 为自己下载的app创建快捷方式
### Jetbrains系
以Idea为例，打开Idea，`settings->create desktop entry`

### 需要自行创建的
以logseq为例，`touch Logseq.desktop`，`sudo vim Logseq.desktop`，编辑其中内容为
```yaml
#!/usr/bin/env xdg-open
[Desktop Entry]
Name=Logseq
Comment=Logseq
Exec=~/Desktop/Program/Logseq/Logseq.AppImage # AppImage所在目录
Icon=~/Desktop/Program/Logseq/logseq.png # 图片所在目录
Terminal=false
Type=Application
Categories=Math; # 对应Categories为Science&Math
```
