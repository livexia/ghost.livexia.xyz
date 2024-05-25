---
title: 初试 Neorg
description: 初次尝试 Neorg ，如果顺利预计替换 Notion 
author: livexia
taxonomies:
  tags: 
    - note-taking
    - neovim
    - neorg
date: 2024-05-25T18:22:20Z
updated: 2024-05-25T23:10:39Z
version: 1.1.1
---

初次尝试 Neorg ，如果顺利预计替换 Notion 


## 安装 Neorg

根据官方文档，在macOS 下通过 lazy.vim [安装 Neorg] ，初次启动 nvim 后发现 nvim-treesitter 会自动安装 norg 的 parser ，安装过程中会报编译错误，大概错误是说使用的 cc 编译器不支持 C++11 ，目前 Neorg 已经解决了这个问题，具体见 [fix: TSInstall issues on macOS, hopefully once and for good] ，大致思路就是忽略 TS 的报错，进入 nvim 后手动运行 `:Neorg sync-parsers` 让 Neorg 来安装 parser 即可。
~~设置 treesitter 不自动安装 norg parser ，在 lunarvim 的 config.lua 文件中增加 `lvim.builtin.treesitter.ignore_install = { "norg" \}`~~ 不知道是我设置的问题，总之这个方法并不成功，依旧会尝试安装 norg parser ，最后还是通过官方的方法解决的。


## 配置 Neorg/启用 Concealer

Neorg 的配置是递归的，在 Lazy 的安装配置中修改 config 的值为函数，然后在 load 中增加需要加载的模块，比如要使用 cincealer 那就要 load 的值就应该为：
```lua
load = {
    ["core.defaults"] = {},
    ["core.concealer"] = {},
}
```
然后如果要对 concealer 模块进行进一步的设置就要在对于模块中增加 config table ，具体需要配置的选项可以在 Neorg 的 wiki 中的具体模块确认，具体可参考 [Setup Guide]。

## 自动生成元数据/summary

运行 `:Neorg inject-metadata` 即可在 norg 文件的顶部生成元数据，具体可见 [metagen](https://github.com/nvim-neorg/neorg/wiki/Metagen) 。
在配置文件中加载 `core.summary` 模块，还可以根据 norg 文件生成工作区间的所有链接，可以有效的减少手动在 index.norg 文件中增加一条一条的链接，加载后运行命令 `:Neorg generate-workspace-summary` 即可在 index 的 heading 下生成，目前我还没找到特别有效的可以自动更新生成结果的方法，具体可见 [summary](https://github.com/nvim-neorg/neorg/wiki/Summary)。


## 导出为 markdown

平时在 Notion 上记录之后，如果觉得需要放到个人博客上，往往我需要将内容复制导出为 markdown 文件，然后手动增加元数据，如果要迁移到 Neorg 那么导出为 markdown 格式也是很重要的功能。
直接使用 `core.export.markdown` 导出即可，在配置中增加 `extensions = "all"` ，就可以实现包括 Neorg 生成的元数据导出，具体在导出时需要 nvim 中运行 `:Neorg export to-file`，具体可见 [Exporting Files](https://github.com/nvim-neorg/neorg/wiki/Exporting-Files) 。
同时我个人使用的博客框架是 zola ，而 zola 需要的 markdown 文件内要包含 [Front matter](https://www.getzola.org/documentation/content/page/#front-matter) ，虽然 zola 推荐这个数据采用 TOML 的格式，但是也支持 YAML 格式，而 Neorg 导出后的元数据正是 YAML 格式，所以只需要很小的改动即可快速的将内容修改至兼容 zola 的格式，其中不同的数据内容理论上可以自动化实现。
虽然相比 Notion 好像是更加方便了，但是因为使用了类似 Anchor 的功能，在导出 markdown 和 zola 构建的过程中，依旧会出现不兼容的情况，markdown 的内部锚点链接不支持中文。而对于 Neorg 简化的锚点链接（只需要在 \[\] 中输入完整的链接名称即可，会调转到那个完整的链接位置），markdown 好像并不支持，目前这两个问题只能暂时规避。

## 一些问题

### strikethrough 渲染不成功

macOS 上我使用的是 iTerm2 在模拟终端中是可以渲染删除线的，但是在 neovim 和 lunarvim 中删除线均不能正常显示，通过修改 term.info 即可实现正常渲染删除线，具体可见[删除线渲染](#shan-chu-xian-xuan-ran)。
终端中运行 `infocmp $TERM > myterm.info` 生成当前终端的信息，然后在 myterm.info 文件末尾加上 `smxx=E[9m, rmxx=E[29m,` ，最后运行 `tic -x myterm.info` 将修改后的终端信息写入数据库，具体可见 [Add strikethrough support in terminal] 。
虽然解决了不渲染的问题，但是底层的根本原因我并未弄清楚，留待以后学习吧。


### 链接 Concealer 渲染

默认 `conceallevel` 为 0 ，设置 conceallevel 为 2 即可实现链接渲染，通过新增文件 `~/.config/lvim/after/ftplugin/norg.lua`，并在其中添加 `vim.opt_local.conceallevel = 2` 实现只针对 norg 文件设置 conceallevel ，具体可见 `:h conceallevel` 。
链接渲染成功，但是在开启自动换行（line wrap）后出现了大段的空白，这是一个 vim/neovim 的遗留问题，目前并没有任何可靠的修复，Neorg 官方给出了一些解决方案但都不完美，具体见 [Ugly line wraps when concealing text]。

### 奇怪的折叠表现

当我第一次使用 nvim 打开文本的时候，折叠并不会发生，但是如果此时再打开一个新的文件，那么折叠就发生了，而且不限制文件类型，例如我第一次打开一个 norg 文件，然后再窗口内打开 lvim 的配置文件 config.lua ，此时这个配置文件就发生了折叠。在 Neorg 的 core.concealer 的配置中设置 folds 为 false 貌似解决了这个问题，因为我并不怎么需要使用折叠，所以目前这个解决方案还行。

### ~~中文输入法自动切换~~

作为文字记录的工具，对于本地输入法的支持就显得尤为重要，这个部分的记录就单独的放在另外的记录中。

## 结论

目前我的 Neorg 体验并不完美，官方的文档显得有些杂乱，仓库内容也很分散，导致在查找的时候要多方搜索，配置 Neorg 最好的地方是在 Neorg 仓库的 [wiki](https://github.com/nvim-neorg/neorg/wiki) ，需确认 Neorg 语法最好的是直接在 nvim 中运行 `:h neorg` 其中也有指向 norg spec 的仓库，然后还有一个非官方的 [Tutorial](https://github.com/pysan3/Norg-Tutorial) 仓库，最后就是作者的 [YouTube 视频系列](https://www.youtube.com/watch?v=NnmRVY22Lq8&list=PLx2ksyallYzVI8CN1JMXhEf62j2AijeDa&index=1) 。即使是不完美的 Neorg 也让我想要将所有 Notion 中的内容进行迁移，Notion 引入了大量的 AI 功能，然后在网速不佳的情况下使用体验实在是太差了，更别提 Notion 的性能了，在我看来 Notion 的缺点远远 Neorg 的缺点，接下来就是慢慢的将 Notion 中的内容逐步进行迁移，并逐渐的扩展 Neorg 和 Neovim 的知识体系。



## 配置（lazy.vim）

```lua
{
    "nvim-neorg/neorg",
    dependencies = { "luarocks.nvim" },
    lazy = false, -- Disable lazy loading as some `lazy.nvim` distributions set `lazy = true` by default
    version = "*", -- Pin Neorg to the latest stable release
    config = function()
        -- setting up Neorg
        require("neorg").setup {
            load = {
                ["core.defaults"] = {},
                ["core.concealer"] = {
                    config = {
                        folds = false,
                        icon_preset = "diamond",
                    },
                },
                ["core.export"] = {},

                ["core.completion"] = {
                    config = {
                        engine = "nvim-cmp",
                    },
                },
                ["core.summary"] = {},
                ["core.esupports.indent"] = {
                    config = {
                        format_on_enter = true,
                        format_on_escape = true,
                    },
                },
                ["core.export.markdown"] = {
                    config = {
                        extensions = "all",
                    },
                },
            },
        }
    end,
    -- fix: TS error see: https://github.com/nvim-neorg/neorg/pull/891
    -- ignore the TS error and run `:Neorg sync-parsers` inside nvim
},
```



## Links

### 安装

- [Installation](https://github.com/nvim-neorg/neorg?tab=readme-ov-file#lazynvim)
- [fix: TSInstall issues on macOS, hopefully once and for good](https://github.com/nvim-neorg/neorg/pull/891)

### 配置

- [Setup Guide](https://github.com/nvim-neorg/neorg/wiki/Setup-Guide)

### line wrap with concealer

- [Ugly line wraps when concealing text](https://github.com/nvim-neorg/neorg/wiki/Dependencies#ugly-line-wraps-when-concealing-text)

### 删除线渲染

- [Why the strikethrough mark can't be rendered?](https://github.com/neovim/neovim/discussions/24346)
- [Add strikethrough support in terminal](https://github.com/neovim/neovim/issues/3436)
- [strikethrough does't work](https://github.com/nvim-neorg/neorg/discussions/755#discussioncomment-5052354)
