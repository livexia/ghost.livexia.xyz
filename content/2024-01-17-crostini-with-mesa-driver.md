+++
title = "ChromeOS/Crostini 安装 mesa virtio-venus-experimental 驱动"
slug = "crostini-with-mesa-driver"
date = 2024-01-17

[taxonomies]
tags = [ "Linux" ]
+++
tl;dr ：在 ChromeOS/Crostini 容器中安装 mesa 驱动，实现 vulkan 在容器中的可用，手动编译安装成功，但 Steam/Proton 游戏帧率提升不明显，ChromeOS/Crostini 通过 Proton 运行游戏（The Conquest of Go）仍与 Linux 直接 Proton 运行有较大差距。

## 起因

最近沉迷围棋，Steam 上的游戏 The Conquest of Go 很棒，所以想在每台电脑上都能玩，这个游戏本不需要性能非常好的电脑，这个游戏原生只支持 Windows 和 MacOS ，在 Linux 上运行需要依赖 Steam 的 Proton 运行，我手上的系统为 Fedora 39 的 Surface Go2 运行没问题，可以跑满 60 帧，为了省电运行在 20 帧运行也没问题，毕竟是围棋游戏。根据这一点，我觉得在 ChromeOS 上运行一定没问题，我这台 PixelBook Go 性能应该优于 Surface Go2 ，很可惜并不是这样，在 Crostini 中安装 Steam ，同样利用 Proton 运行游戏，结果不到 10FPS 这就不可玩了，我猜测是驱动的问题，谷歌官方虽然有在对更好的驱动进行试验，可是我的这台并不在测试名单中，具体见：**[Play Steam for Chromebook（Beta 版）](https://support.google.com/chromebook/answer/14220699)**，最初我也就放弃了，可惜日渐沉迷，于是又在网上查找可能的解决方法，以下就是我的搜索结果，通过安装 mesa 的测试驱动，实现更好的 Vulkan 支持，使得 Proton 可以通过 DXVK 将 d3d 9/10/11 的 api 转为 Vulkan 。

## 原教程

1. [Debian Bullseye](https://gist.github.com/Usulyre/bb33f77b225b8d9336c1f9e744114fba)
2. [Arch](https://old.reddit.com/r/chromeos/comments/uq13t0/steam_gaming_by_vulkan_for_crostini_and_this/)

初步阅读教程，可以发现大部分的操作并不太可能损害环境，所以我决定直接在原有环境上进行编译，而不新建容器，构建 Arch 虚拟机可能可以让整个流程加快，但是我还是喜欢用统一的环境。

初始时运行 `vulkaninfo` 和 `vkcube` 输出情况，可见是 CPU

```
❯ vulkaninfo | grep driverName
	driverName      = llvmpipe
	driverName                                           = llvmpipe

❯ vkcube
Selected GPU 0: llvmpipe (LLVM 15.0.6, 256 bits), type: Cpu
```

## 前置条件，启用 crostini

在 chrome://flags 中：

1. 使用最新版本的 Debian 我使用的是最新的 Bookworm chrome://flags/#crostini-container-install
2. 启用 GPU 加速 chrome://flags/#crostini-gpu-support

## 具体操作

关闭现有的容器（虚拟机），在 `crosh` （ctrl + alt + t） 中增加启用 GPU 参数启动容器环境，参数为 `--enable-vulkan` 例如 `vmc start --enable-vulkan termina`

在 `/etc/apt/sources.list` 增加源，并更新容器

1. 增加源 `deb-src [arch=amd64,i386] http://deb.debian.org/debian bookworm main`
2. 注意不需要增加 sid 的源

### 安装编译驱动需要的依赖，拉取代码

```bash
sudo apt-get build-dep mesa
sudo apt-get install libunwind-dev
sudo apt-get install libudev-dev

git clone https://gitlab.freedesktop.org/mesa/mesa.git
```

进入目录 `cd mesa`

### 编译 64 位驱动

**配置编译**

```bash
meson setup build64 --libdir /usr/lib/x86_64-linux-gnu \
	-Ddri3=enabled                              \
  -Dprefix=/usr                               \
  -Dglx=dri                                   \
  -Degl=enabled                               \
  -Dgbm=enabled                               \
  -Dgallium-vdpau=disabled                    \
  -Dvalgrind=disabled                         \
  -Dgallium-drivers=virgl              \
  -Dvulkan-drivers=virtio              \
  -Dvulkan-layers=device-select
```

运行命令，未有明显错误输出。

**运行编译 `sudo ninja -C build64 install`** 。编译的时候，我这台无风扇的电脑 CPU 温度达到 75C 左右，还是比较吃性能和缓慢的，编译成功，在我的机器上运行了 15 分钟不到。

### 编译 32 位驱动

**配置 gcc** 

新建文件夹 mkdir -p ~/.local/share/meson/cross

新增文件 ~/.local/share/meson/cross/gcc-i686

内容为：

```
# gcc-i686
[binaries]
c = '/usr/bin/gcc'
cpp = '/usr/bin/g++'
ar = '/usr/bin/gcc-ar'
strip = '/usr/bin/strip'
pkgconfig = '/usr/bin/i686-linux-gnu-pkg-config'
llvm-config = '/usr/bin/llvm-config'

[built-in options]
c_args = ['-m32']
c_link_args = ['-m32']
cpp_args = ['-m32']
cpp_link_args = ['-m32']

[host_machine]
system = 'linux'
cpu_family = 'x86'
cpu = 'i686'
endian = 'little'
```

**安装编译依赖**，也许非必需，作者并不确定，因为是 bookworm 所以要修改原教程中部分的 sid 安装命令为正常的安装命令。尝试不安装依赖，直接编译测试，编译配置就失败了。

```bash
sudo apt-get install gcc-multilib
sudo apt-get install g++-multilib
sudo dpkg --add-architecture i386
sudo apt-get update
sudo apt-get install pkg-config:i386
sudo apt-get install libdrm-dev:i386
sudo apt-get install libwayland-dev:i386
sudo apt-get install libwayland-egl-backend-dev:i386
sudo apt-get install libxext-dev:i386
sudo apt-get install libxfixes-dev:i386
sudo apt-get install x11proto-dev:i386
sudo apt-get install libxcb-glx0-dev:i386
sudo apt-get install libxcb-shm0-dev:i386
sudo apt-get install libx11-xcb-dev:i386
sudo apt-get install libxcb-dri2-0-dev:i386
sudo apt-get install libxcb-dri3-dev:i386
sudo apt-get install libxcb-present-dev:i386
sudo apt-get install libxshmfence-dev:i386
sudo apt-get install libxxf86vm-dev:i386
sudo apt-get install libxrandr-dev:i386
sudo apt-get install libunwind-dev:i386
sudo apt-get install libudev-dev:i386
sudo apt-get install libelf-dev:i386
sudo apt-get install libzstd-dev:i386
```

**配置 32 位编译**

```bash
meson setup build32 --cross-file gcc-i686 --libdir /usr/lib/i386-linux-gnu                                                        -Ddri3=enabled                              \
  -Dprefix=/usr                               \
  -Dglx=dri                                   \
  -Degl=enabled                               \
  -Dgbm=enabled                               \
  -Dgallium-vdpau=disabled                    \
  -Dvalgrind=disabled                         \
  -Dgallium-drivers=virgl              \
  -Dvulkan-drivers=virtio              \
  -Dvulkan-layers=device-select
```

提示 gcc 的一些参数被弃用，但貌似并不影响最后的编译配置，修改 `~/.local/share/meson/cross/gcc-i686` 中 `[properties]` 为 `[built-in options]`

**运行编译 `sudo ninja -C build32 install` 编译错误，提示**：

`/usr/bin/ld: cannot find -lLLVM-14: No such file or directory` 貌似编译过程中配置了错误的 llvm 版本参数

虽然 32 位驱动没能编译成功，但是运行 `vulkaninfo` 和 `vkcube` 都能取得预期结果。

```bash
❯ vulkaninfo | grep driverName
MESA: error: Use of VkSurfacePresentModeCompatibilityEXT without a VkSurfacePresentModeEXT set. This is an application bug.
ERROR while creating surface for extension VK_KHR_wayland_surface : ./vulkaninfo/vulkaninfo.h:237:vkGetPhysicalDeviceSurfacePresentModesKHR failed with ERROR_SURFACE_LOST_KHR
	driverName      = venus
	driverName                                           = venus
	driverName      = llvmpipe
	driverName                                           = llvmpipe

❯ vkcube
Selected GPU 0: Virtio-GPU Venus (Intel(R) UHD Graphics 615 (AML-KBL)), type: IntegratedGpu
```

先尝试运行游戏：Steam 客户端一样的卡，但是运行游戏时弹出了配置 Vulkan 着色器的窗口，也许是好迹象，大概 15FPS ，依旧是不可玩的状态，看来是需要 32 位的驱动。因为在 Fedora 上，使用 Proton 7.0 有更好的兼容性，所以更换 Proton 版本再测试一下。测试 ProtonDB 上提到的启动参数 `PROTON_USE_WINED3D11=1 %command%` 情况依旧。

**安装额外的依赖后编译成功：**

```bash
sudo apt install llvm:i386 llvm-dev:i386
```

## 检查安装情况

安装 vulkan 测试工具 `sudo apt-get install vulkan-tools`

查看安装的驱动版本 `ls /usr/share/vulkan/icd.d/`

预期：`virtio_icd.x86_64.json` 和 `virtio_icd.i686.json`

简单测试：`VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/virtio_icd.i686.json:/usr/share/vulkan/icd.d/virtio_icd.x86_64.json vkcube`

修改环境变量，令驱动持续生效

编辑 `/etc/environment` 文件增加环境变量`VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/virtio_icd.i686.json:/usr/share/vulkan/icd.d/virtio_icd.x86_64.json`

运行 `vulkaninfo` 查看 vulkan 驱动信息

运行 `vkcube` 输出对应 GPU 信息

```bash
❯ vulkaninfo | grep driverName
WARNING: [Loader Message] Code 0 : env var 'VK_INSTANCE_LAYERS' defined and adding layers "VK_LAYER_MESA_overlay"
WARNING: [Loader Message] Code 0 : env var 'VK_INSTANCE_LAYERS' defined and adding layers "VK_LAYER_MESA_overlay"
MESA: error: Use of VkSurfacePresentModeCompatibilityEXT without a VkSurfacePresentModeEXT set. This is an application bug.
ERROR while creating surface for extension VK_KHR_wayland_surface : ./vulkaninfo/vulkaninfo.h:237:vkGetPhysicalDeviceSurfacePresentModesKHR failed with ERROR_SURFACE_LOST_KHR
	driverName      = venus
	driverName                                           = venus

~
❯ vkcube
Selected GPU 0: Virtio-GPU Venus (Intel(R) UHD Graphics 615 (AML-KBL)), type: IntegratedGpu
```

## 后续

1. 在 chrome://flags 中有用的选项 chrome://flags/#exo-ordinal-motion
2. 如果驱动存在问题，可以尝试拉取最新代码，重新编译安装，再进行测试
3. 增加 vulkan 程序 FPS 计数器，在 /etc/environment 文件中添加环境变量  `VK_INSTANCE_LAYERS=VK_LAYER_MESA_overlay`

## 前后游戏帧数对比

**The Conquest of Go**

默认：不到 10 FPS 不可玩

Vulkan：在 Steam 中增加游戏运行参数 `DXVK_HUD=devinfo,fps %command%` 可见游戏已经运行在正确的 Vulkan 驱动下，但是游戏帧数在 18FPS 左右，稍微好点，但是游玩依旧难受，至少要 20 FPS 才行。
