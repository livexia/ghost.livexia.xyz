+++
title = "修复 zsh 启动缓慢"
slug = "fix-zsh-start-slow"
date = 2024-01-23

[taxonomies]
tags = [ "Linux" ]
+++
tl;dr: 终端启动缓慢，实际上是 `zsh` 启动缓慢，通过使用 `zprof` 确定问题来自 `nvm` 的初始化，利用 oh-my-zsh 插件实现 `nvm` 只在使用时进行初始化。

## 起因

主要使用的环境有 3 个，两个 Linux 一个 MacOS，自己从大学开始就在用 oh-my-zsh ，长久以来都是如此。不知道从何时开始启动终端就特别的缓慢，特别是在 Linux 的环境上，因为 Linux 环境上我用的是 Alacritty 而 MacOS 上用的则是 iterm2 ，所以最初我以为是终端的问题，因为问题并不是很大，特别是涉及到的两个环境的机器并不高，所以我并没有想要查清问题。昨天晚上太冷了，在冰冷的笔记本上等待终端启动让我觉得很有必要解决这个问题。

## 确定问题

终端启动缓慢有两种可能：一是终端模拟器启动缓慢，二是 `zsh` 启动缓慢。当前点击 Alacritty 时，终端窗口出现并不慢，而 shell 环境仍未出现，所以更可能是 `zsh` 的问题。通过启动系统自带的终端模拟器，发现启动依旧缓慢，所以并不是终端模拟器的问题。

### zsh 的启动时间

使用 `time` 命令测试 `zsh` 启动时间 `time zsh -i -c exit` 命令会在启动 `zsh` 后立即退出，所以可以估算 `zsh` 的启动时间，运行结果为 `zsh -i -c exit  0.95s user 1.26s system 106% cpu 2.075 total` 可见启动时间在一秒左右，这是很明显感到启动缓慢的原因。

### 启用 zsh profiling

参考 [why does zsh start so slowly?](https://pickard.cc/posts/why-does-zsh-start-slowly/) 和 [Profiling zsh startup time](https://stevenvanbael.com/profiling-zsh-startup) 对 `zsh` 的启动进行采样，具体操作是在 `~/. zshrc` 的顶部增加 `zmodload zsh/zprof` 和在最底部增加 `zprof` ，然后重新启动 `zsh` 即可取得 profiling 的结果。

重启 `zsh` 后取得 profiling 结果如下：

```bash
num  calls                time                       self            name
-----------------------------------------------------------------------------------
 1)    1         804.38   804.38   89.73%    436.31   436.31   48.67%  nvm_auto
 2)    2         368.07   184.03   41.06%    206.64   103.32   23.05%  nvm
 3)    1         146.69   146.69   16.36%    122.63   122.63   13.68%  nvm_ensure_version_installed
 4)   21          43.31     2.06    4.83%     34.24     1.63    3.82%  _omz_source
 5)    1          24.06    24.06    2.68%     24.06    24.06    2.68%  nvm_is_version_installed
 6)    2          21.35    10.68    2.38%     21.35    10.68    2.38%  compaudit
 7)    1          14.63    14.63    1.63%     14.47    14.47    1.61%  nvm_die_on_prefix
 8)    1          11.41    11.41    1.27%     11.41    11.41    1.27%  (anon) [/home/livexia/.oh-my-zsh/tools/check_for_upgrade.sh:155]
 9)    1           8.27     8.27    0.92%      8.27     8.27    0.92%  zrecompile
10)    1          25.99    25.99    2.90%      4.64     4.64    0.52%  compinit
11)    1          13.74    13.74    1.53%      2.33     2.33    0.26%  handle_update
12)    1           1.91     1.91    0.21%      1.91     1.91    0.21%  test-ls-args
13)    5           1.88     0.38    0.21%      1.88     0.38    0.21%  add-zsh-hook
14)    1           1.86     1.86    0.21%      1.86     1.86    0.21%  colors
15)   13           1.54     0.12    0.17%      1.54     0.12    0.17%  compdef
16)    5           1.15     0.23    0.13%      1.15     0.23    0.13%  is-at-least
17)    1           1.05     1.05    0.12%      1.05     1.05    0.12%  regexp-replace
18)    4           0.16     0.04    0.02%      0.16     0.04    0.02%  nvm_npmrc_bad_news_bears
19)    1           0.28     0.28    0.03%      0.13     0.13    0.01%  complete
20)    1           0.11     0.11    0.01%      0.11     0.11    0.01%  nvm_has
21)    2           0.08     0.04    0.01%      0.08     0.04    0.01%  is_plugin
22)    3           0.08     0.03    0.01%      0.08     0.03    0.01%  is_theme
23)    2           0.06     0.03    0.01%      0.06     0.03    0.01%  bashcompinit
24)    1         804.43   804.43   89.73%      0.05     0.05    0.01%  nvm_process_parameters
25)    2           0.05     0.02    0.01%      0.05     0.02    0.01%  env_default
26)    1           0.01     0.01    0.00%      0.01     0.01    0.00%  nvm_is_zsh
```

根据结果可见**启动过程中因为 `nvm` 而花费了大量的时间**，并不是很多人遇见的因为 `compinit` 而导致的启动缓慢，例如 [zsh starts incredibly slowly](https://superuser.com/questions/236953/zsh-starts-incredibly-slowly) 中的题主。

## 解决因为 `nvm` 导致的启动缓慢

`~/.zshrc` 文件中涉及 `nvm` 的部分如下，主要是设定 `nvm` 主目录，然后加载 `nvm` ，最后加载 `nvm` 的 `shell` 补全，这样三个部分就导致了 `zsh` 的启动缓慢

```bash
export NVM_DIR="$HOME/.config/nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

`nvm` 是 Nodejs 的版本管理器，我实际上并不直接使用 Nodejs ，但是在 LunarVim 中安装插件需要，所以直接删除 `~/.zshrc` 中涉及 nvm 的部分可能会导致问题。

在搜索过程中发现通过跳过初始化 `zsh` 时跳过加载 `nvm` 解决的[方法](https://superuser.com/a/1611283)，这个方法通过设置别名只在运行 `nvm` 时进行加载，这个方法无法实现在使用 `node` 或 `npm` 以及一系列其他程序时自动加载 `nvm`。但是后续发现了更好的解决方法，那就是通过使用 oh-my-zsh 的 `nvm` 插件实现类似的方法，只在需要时初始化 `nvm` 。

根据 [nvm plugin](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/nvm) 在 `~/.zshrc` 中启用 `nvm` 插件，然后设置 `nvm` 启动状态为 `lazy` 。

1. 启用 `nvm` 插件 `plugins=(... nvm)` ，在 `~/.zshrc` 文件中搜索 `plugins` 可以发现默认就启用了 `git` 插件，所以直接在原有的基础上增加 `nvm` 即可，直接加上文档中的 `plugins=(... nvm)` 这一句会覆盖 `git` 插件。例如 `plugins=(git nvm)`****
2. 在 `~/.zshrc` 中增加 `zstyle ':omz:plugins:nvm' lazy yes` 实现只有在需要时才加载 `nvm`
3. 删除 `~/.zshrc` 中原有的两行加载 nvm 和 nvm shell 补全的部分，要保留设定 nvm 目录的部分，因为 `nvm` 插件默认 `~/.nvm` 为主目录，具体可见 [nvm/settings](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/nvm#settings)

最初这么操作之后，发现启动中依旧加载了 `nvm` ，的确 `nvm` 插件替换了 `nvm` 的两条初始化语句，但是在 profiling 中依旧加载了 `nvm` ，也就是说 `nvm` 依旧在 `zsh` 初始化时启动了。

测试后发现 `plugins=(git nvm)` 和 `zstyle ':omz:plugins:nvm' lazy yes` 的语句顺序导致了这样的问题，`zstyle ':omz:plugins:nvm' lazy yes` 要在 `plugins=(git nvm)` 之前，否则 `zsh` 依旧会自动初始化 `nvm` 。正确的是在 `~/.zshrc` 最初位置就设定 `nvm` 主目录和增加 `zstyle ':omz:plugins:nvm' lazy yes` ，`~/.zshrc` 头部示例如下：

```bash
# profiling zsh
zmodload zsh/zprof

# nvm lazy load
export NVM_DIR="$HOME/.config/nvm"
zstyle ':omz:plugins:nvm' lazy yes
```

### 不成功的尝试

上面的方法可以实现预期，但是分离了配置，一般来说我喜欢将新增/修改的配置放在配置文件的末尾，这个方法对配置进行了分裂。`plugins` 是 `bash` 的 `array` 数据类型，应该可以通过附加操作增加元素，因为 `zstyle` 设置 `nvm` 插件语句只需要在插件启用之前之前即可。所以可以将所有 `nvm` 相关的语句集合在一起，同时可以在配置文件的最后实现 nvm 的按需加载。具体如下：

```bash
# nvm plugin lazy load
export NVM_DIR="$HOME/.config/nvm"
zstyle ':omz:plugins:nvm' lazy yes
plugins+=(nvm)
```

但是这个方法依旧存在问题，`plugins` 需要在 `source $ZSH/oh-my-zsh.sh` 之前进行设置，如果放在配置文件末尾，将导致 `nvm` 不可用。所以我希望将所有新增配置都放在配置文件末尾的希望落空了，最后仍旧是按照最初的方式，将设置 `nvm` 主目录和 `zstyle` 设置 `nvm` 插插件的语句放在配置文件的头部，同时新增 `plugins` 变量为 `plugins=(git nvm)` 。

### 确定 `nvm` 可用

使用 `nvm` 插进后运行 `nvm` `node` 或者 `npm` 都可以成功，同时启动终端模拟器和 zsh 几乎是瞬时的，毫无停滞的感觉。

## 再次测试

再次 profiling 结果如下：

```bash
❯ zsh
num  calls                time                       self            name
-----------------------------------------------------------------------------------
 1)   21          31.85     1.52   41.86%     24.68     1.18   32.44%  _omz_source
 2)    2          24.05    12.02   31.60%     24.05    12.02   31.60%  compaudit
 3)    1           8.25     8.25   10.85%      8.25     8.25   10.85%  (anon) [/home/livexia/.oh-my-zsh/tools/check_for_upgrade.sh:155]
 4)    1          29.06    29.06   38.20%      5.02     5.02    6.59%  compinit
 5)    1           3.57     3.57    4.69%      3.57     3.57    4.69%  zrecompile
 6)    1          11.15    11.15   14.66%      2.90     2.90    3.81%  handle_update
 7)    1           1.97     1.97    2.59%      1.97     1.97    2.59%  test-ls-args
 8)    5           1.25     0.25    1.64%      1.25     0.25    1.64%  add-zsh-hook
 9)    1           1.22     1.22    1.61%      1.22     1.22    1.61%  colors
10)   12           1.22     0.10    1.60%      1.22     0.10    1.60%  compdef
11)    5           1.13     0.23    1.48%      1.13     0.23    1.48%  is-at-least
12)    1           0.59     0.59    0.77%      0.59     0.59    0.77%  regexp-replace
13)    2           0.13     0.06    0.17%      0.13     0.06    0.17%  is_plugin
14)    3           0.08     0.03    0.10%      0.08     0.03    0.10%  is_theme
15)    2           0.03     0.01    0.04%      0.03     0.01    0.04%  env_default
16)    1           0.02     0.02    0.03%      0.02     0.02    0.03%  bashcompinit
```

禁用 profiling 测试启动时间：

```bash
❯ time zsh -i -c exit
zsh -i -c exit  0.19s user 0.22s system 117% cpu 0.348 total
```

可以发现启动时间从最初的 `2.075` 减少到 `0.348` ，实际的使用体验也反映了这一提升，终端启动几乎是即时没有延迟感。在我的 `~/.zshrc` 文件中，除了初始化 `nvm` 还初始化了 `zoxide` `starship` 和 `pyenv` ，虽然在测试中并未发现这些初始化占用了大量的启动时间，但并不代表未来不会有这样的问题，后续再遇到也许也可以通过这个方法解决，使用插件。虽然在这些插件的页面并不涉及可能的加速方法，因此在此并不将原有初始化方法修改为使用配置。

## 参考

1. [Fix slow ZSH startup due to NVM](https://dev.to/thraizz/fix-slow-zsh-startup-due-to-nvm-408k)
2. [why does zsh start so slowly?](https://pickard.cc/posts/why-does-zsh-start-slowly/)
3. [Profiling zsh startup time](https://stevenvanbael.com/profiling-zsh-startup)
4. [zsh starts incredibly slowly](https://superuser.com/questions/236953/zsh-starts-incredibly-slowly)
5. [nvm plugin](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/nvm)
6. [oh-my-zsh/Plugins](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins)
