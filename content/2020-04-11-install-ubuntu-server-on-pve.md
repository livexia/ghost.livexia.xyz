+++
title = "PVE后续使用记录Part1-UbuntuServer"
slug = "install-ubuntu-server-on-pve"
date = 2020-04-11T09:08:28.000Z
excerpt = "在pve上安装ubuntu server"

[taxonomies]
tags = [ "Tech" ]
+++

## 安装Ubuntu Server 18.04

需要使用Linux的环境来部署一些工具，由于这些工具对Ubuntu的支持都较为良好，所以选用Ubuntu Server.

### 下载

可以在ubnutu的官网进行下载，虽然官网会根据地区自动优化下载镜像源，不过在这里还是推荐直接在国内的镜像源上下载镜像，保证高速下载，在此给出：[清华大学镜像源的下载地址](https://mirror.tuna.tsinghua.edu.cn/ubuntu-releases/bionic/ubuntu-18.04.4-live-server-amd64.iso)

**注意下载完的镜像需要上传到pve的存储中，或者在pve上利用wget进行下载**

### 在PVE上创建虚拟机

#### Part1：PVE创建虚拟机

**Step1：点击创建虚拟机**

![image](https://user-images.githubusercontent.com/15051530/79039176-0c8c5380-7c12-11ea-8e26-eb64636f071c.png)

**Step2：选择创建虚拟机在哪一个节点、VM的ID和名称，点击下一步**

![image](https://user-images.githubusercontent.com/15051530/79039189-275ec800-7c12-11ea-8086-864a54d4758f.png)

**Step3：选择前面上传的系统安装镜像，点击下一步**

![image](https://user-images.githubusercontent.com/15051530/79039197-30e83000-7c12-11ea-80e0-9c3685fd592c.png)

**Step4：选择图像选项和BISO类型，推荐图像选项选择VirtIO-GPU，BIOS类型可不选择UEFI，如果选择了UEFI，需要选择UEFI的存储位置，点击下一步**

![image](https://user-images.githubusercontent.com/15051530/79039202-42c9d300-7c12-11ea-854c-5fc1e9a54479.png)

**Step5：修改磁盘的默认大小，点击下一步**

![image](https://user-images.githubusercontent.com/15051530/79039206-4c533b00-7c12-11ea-964a-5d7776d82cf8.png)

**Step6：修改CPU核数和CPU启用的额外标志，此处默认，点击下一步**

![image](https://user-images.githubusercontent.com/15051530/79039210-55440c80-7c12-11ea-99c9-593fb0ab1e7f.png)

**Step7：修改内存大小，此处分配8G，点击下一步**

![image](https://user-images.githubusercontent.com/15051530/79039217-5d03b100-7c12-11ea-8770-45bef5a29fc7.png)

**Step8：修改网络设置即可，点击下一步**

![image](https://user-images.githubusercontent.com/15051530/79039225-642abf00-7c12-11ea-8161-5f1901a3f489.png)

**Step9：最后的整体设置预览，点击完成**

![image](https://user-images.githubusercontent.com/15051530/79039227-69880980-7c12-11ea-8800-4afc50732d83.png)

#### Part2：安装Ubuntu Server 18，04

**Step1：启动安装**，点击z右上角的启动，进入安装阶段

![image](https://user-images.githubusercontent.com/15051530/79039832-976f4d00-7c16-11ea-9dac-8011bcbbc531.png)

**Step2：点击控制台，输出vnc画面**

![image](https://user-images.githubusercontent.com/15051530/79039850-b241c180-7c16-11ea-929b-5232bf1a5222.png)

**Step3：选择语言**

![image](https://user-images.githubusercontent.com/15051530/79039881-f8972080-7c16-11ea-9ec2-0de5bc84b2c3.png)

**Step4：更新安装器**，推荐进行更新，网络环境较差也可不更新

![image](https://user-images.githubusercontent.com/15051530/79039527-6857dc00-7c14-11ea-8ef3-24d46f8f6512.png)

**Step5：配置语言和键盘**

![image](https://user-images.githubusercontent.com/15051530/79039537-74439e00-7c14-11ea-81ab-67875166db11.png)

**Step6：配置网络，默认即可**

![image](https://user-images.githubusercontent.com/15051530/79039540-79a0e880-7c14-11ea-809f-12be95bd2131.png)

**Step7：配置代理，按需配置**

![image](https://user-images.githubusercontent.com/15051530/79039551-7efe3300-7c14-11ea-81c8-bb10e9b4da81.png)

**Step8：配置镜像源，按需配置**

![image](https://user-images.githubusercontent.com/15051530/79039553-845b7d80-7c14-11ea-878e-87ea2409a7be.png)

**Step9：设置使用一整个磁盘，有需要要也可选择其他选项**

![image](https://user-images.githubusercontent.com/15051530/79039558-8f161280-7c14-11ea-9b6b-ec1752c43b77.png)

**Step10：选择安装的磁盘**

![image](https://user-images.githubusercontent.com/15051530/79039560-93dac680-7c14-11ea-9337-ddac162b12cd.png)

**Step11：确认自动分区情况**

![image](https://user-images.githubusercontent.com/15051530/79039565-99381100-7c14-11ea-969e-c13d1979a791.png)

**Step12：配置用户名、密码与host name**

![image](https://user-images.githubusercontent.com/15051530/79039569-a05f1f00-7c14-11ea-88d2-647293dcfd37.png)

**Step13：选择安装openssh-server**

![image](https://user-images.githubusercontent.com/15051530/79039578-a94ff080-7c14-11ea-8c3e-0a7215443ac1.png)

**Step14：推荐不在此页面选择任何额外的软件，此处的软件应该是通过ubnutu的snap包管理器进行安装的，如果不熟悉的情况下，可能选取之后找不到相应的安装结果，我就在前几次的安装中选择了docker，导致系统上有两个docker sever端，令人十分迷惑。**

![image](https://user-images.githubusercontent.com/15051530/79039582-b79e0c80-7c14-11ea-8633-ff004aaf9e37.png)

**Step15：开始安装，等待安装结束即可**

![image](https://user-images.githubusercontent.com/15051530/79039584-bf5db100-7c14-11ea-8622-9dfd90d3f1ec.png)
