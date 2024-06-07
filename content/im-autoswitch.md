---
title: 实现 Neovim 输入法自动切换
slug: im-autoswitch
description: 使用 rlue/vim-barbaric 实现 Neovim 自动切换输入法。 
author: livexia
taxonomies:
  tags: 
    - neovim
date: 2024-06-07T11:20:38Z
updated: 2024-06-07T12:36:27Z
version: 1.1.1
---


如果是只使用 Neovim 写代码，那么 Neovim 的体验很好，因为大部分的输入场景都可以使用英文输入，但是如果要使用 Neovim 来进行记录，也就是在频繁需要混用输入法的场景下，那么 Neovim 的体验就稍差。在 insert 模式下使用中文输入法输入，然后进入 normal 或 visual 模式后由于输入法仍是中文，导致按键无法被 Neovim 获取，这时就需要手动切换输入法到英文，而这并不是很好的体验。


## 确定自动切换的方法

在搜索解决方案的时候，发现一篇很不错的文章，介绍了各式各样的解决方案，具体可见 链接 1 。在阅读过文章，以及进入各个方案的具体页面之后，我决定使用 [vim-barbaric](https://github.com/rlue/vim-barbaric) ，主要是因为这个插件需要的配置相对少，同时在 Linux 上不需要使用第三方软件（这一点其实是大部分解决方案的共同优点），虽然相较于其他的解决方案，这个方案的适配范围稍小，因为我的主要环境是 macOS 加上少部分的 Linux ，所以这个方法对于我的使用场景是比较充分的。


## 安装 xkbswitch-macosx

### intel Mac

参考 [myshov/xkbswitch-macosx](https://github.com/myshov/xkbswitch-macosx) ，总结其实就是复制仓库，然后将 `bin/xkbswitch` 复制到 `$PATH` 路径即可，也可以不复制仓库，直接运行 `curl -o /usr/local/bin/xkbswitch https://raw.githubusercontent.com/myshov/xkbswitch-macosx/master/bin/xkbswitch` 。

### Apple Silicon

需要将仓库 [xiehuc/xkbswitch-macosx](https://github.com/xiehuc/xkbswitch-macosx) 复制下来，然后运行 `make` ，接着将对应的克制文件同样的复制到 `$PATH` 即可，这个仓库是 [myshov/xkbswitch-macosx] 的 Fork 专门为了兼容 Apple Silicon 。
具体步骤：
```bash
git clone https://github.com/xiehuc/xkbswitch-macosx.git
cd xkbswitch-macosx
make
cp xkbswitch /usr/local/bin/xkbswitch
```

只执行 `make` 会生成包含 x86 和 arm 的通用可执行文件，可以执行 `make xkbswitch-x86` 或执行 `xkbswitch-arm` 单独生成对应架构的可执行文件。


## 安装 rlue/vim-barbaric 插件

我是使用 Lazy.nvim 安装的，具体如下：
```lua
{
  "rlue/vim-barbaric",
},
```

对于我的需求，简单这样安装之后即可，安装之后完成了这篇记录内容，使用体验良好，唯一需要注意的是在切换模式后不能非常快的输入按键，因为插件切换输入法到输入法完成切换是存在一点延时的，不过这个问题很轻微，我仅仅遇到一次，即使是遇到的一次也对使用几乎没有任何影响。
在插件的介绍页面，对于配置插件使用的是 Vim Scipt 的语法，如果需要进一步配置，那么可以参考如下的方法对应设置：
`let g:barbaric_ime = 'fcitx'` -> `vim.g.barbaric_ime = 'fcitx'`


## 链接

1. [如何让 Neovim 中文输入时自动切换输入法](https://jdhao.github.io/2021/02/25/nvim_ime_mode_auto_switch)
