+++
title = "ROCm安装记录"
slug = "install-rocm"
date = 2020-04-06T14:21:08.000Z
excerpt = "ROCm安装记录"

[taxonomies]
tags = [ "Coding" ]
+++

## 安装环境

- PVE虚拟机：Ubuntu 18.04.4
- 显卡：直通RX580

## 安装教程

> [https://rocm-documentation.readthedocs.io/en/latest/Installation_Guide/Installation-Guide.html](https://rocm-documentation.readthedocs.io/en/latest/Installation_Guide/Installation-Guide.html)

**简单将Ubuntu的部分简单翻译**

#### 从debian仓库安装ROCm

**Step1：执行下列命令，确保系统的组件为最新:**

> 建议替换为 [清华源](https://mirror.tuna.tsinghua.edu.cn/help/ubuntu/)

    sudo apt update
    
    sudo apt dist-upgrade
    
    sudo apt install libnuma-dev
    
    sudo reboot
    

**Step2：增加ROCm apt 仓库地址**

对于类似Ubuntu的基于Debian的系统执行下列命令：

    wget -q -O - http://repo.radeon.com/rocm/apt/debian/rocm.gpg.key | sudo apt-key add -
    
    echo 'deb [arch=amd64] http://repo.radeon.com/rocm/apt/debian/ xenial main' | sudo tee /etc/apt/sources.list.d/rocm.list
    

**Step3：更新仓库列表然后安装 rocm-dkms meta-package:**

    sudo apt update
    
    sudo apt install rocm-dkms
    

**Step4：设置权限。**

**必须成为video用户组的一员才能访问GPU。 利用以下命令确定当前用户是否是video用户组的一员**

    groups
    

**执行下列语句添加当前用户到video组，需要当前用户是sudoer**

    sudo usermod -a -G video $LOGNAME
    

**执行下列语句，会在下次新建用户时，默认将用户加到video组:**

    echo 'ADD_EXTRA_GROUPS=1' | sudo tee -a /etc/adduser.conf
    
    echo 'EXTRA_GROUPS=video' S| sudo tee -a /etc/adduser.conf
    

**Step5：重启**

**Step6：简单测试安装**

重启之后运行以下命令，确认安装是否成功，如果在命令输出结果中都能看见你的GPU，那么安装成功了

    /opt/rocm/bin/rocminfo
    /opt/rocm/opencl/bin/x86_64/clinfo
    

Note:为了运行ROCm程序更加高效，可执行以下命令，增加ROCm二进制路径到PATH中.

    echo 'export PATH=$PATH:/opt/rocm/bin:/opt/rocm/profiler/bin:/opt/rocm/opencl/bin/x86_64' |
    sudo tee -a /etc/profile.d/rocm.sh
    

#### 从Ubuntu中删除ROCm相关软件包

从Ubuntu 16.04.6 或者 Ubuntu 18.04.4中卸载ROCm相关软件包，执行以下命令:

    sudo apt autoremove rocm-opencl rocm-dkms rocm-dev rocm-utils
    

#### 安装相关开发软件包以支持跨平台编译

推荐在不同的平台上开发和测试软件包。例如有的开发或者构建环境上并没有安装AMD的GPU，在这样的情况下应该避免安装ROCk 内核驱动在开发的机器中。

应该使用下列命令安装开发软件包：

    sudo apt update
    sudo apt install rocm-dev
    

Note: 如果需要执行ROCm使能的应用软件，那么你必须在环境中安装完整的ROCm驱动栈。

#### 配合上有内核使用基于Debian的ROCm

你可以不安装AMD的定制化ROCk内核驱动，直接安装ROCm作为用户层软件。

如果想要使用上游内核，使用以下命令安装ROCm而不是直接安装rocm-dkms：

    sudo apt update
    sudo apt install rocm-dev
    echo 'SUBSYSTEM=="kfd", KERNEL=="kfd", TAG+="uaccess", GROUP="video"'
    sudo tee /etc/udev/rules.d/70-kfd.rules
    

## 利用TensorFLow benchmarks确认安装结果

**下载代码**

    git clone https://github.com/tensorflow/benchmarks.git
    

**安装依赖**

    sudo apt install rocm-libs python3-pip 
    

如果运行报错：

`librccl.so: cannot open shared object file: No such file or directory`

说明缺少依赖 rccl

    sudo apt install rccl
    

**安装tensorflow-rocm**

> 参照 pypi[镜像使用帮助](https://mirror.tuna.tsinghua.edu.cn/help/pypi/) 利用清华源镜像下载tensorflow-rocm

    pip install -i https://pypi.tuna.tsinghua.edu.cn/simple tensorflow-rocm
    

**运行脚本**

    cd benchmarks/scripts/tf_cnn_benchmarks
    python3 tf_cnn_benchmarks.py --num_gpus=1 --batch_size=64 --model=resnet50
    

**运行结果**

由于是在不完全显卡直通的pve虚拟机上运行的，基本性能只有网上测评的一般，上述命令测试结果是：88 images/sec

**后续测试**

由于tensorflow benchmarks中的script不在推荐，下次会再次使用其中提供的perfzero进行测试。
