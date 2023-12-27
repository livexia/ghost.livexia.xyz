+++
title = "扩展 LUKS/Btrfs 主分区"
slug = "expand-luks-btrfs"
date = 2023-12-27

[taxonomies]
tags = [ "Linux" ]
+++

tl;dr ：这么复杂的操作是没法长话短说，长话段说容易进不了系统。

设备环境：

- 硬盘总大小 128G
- 原始分配情况：Windows 占用 80G ，Fedora 占用 38G
- 目标分配情况：Windows 占用 55G ，Fedora 占用 63G
- 实际分配情况：Windows 占用 64G ，Fedora 占用 54G
- 初始时硬盘分区大概的情况：从左到右
    - FAT 分区，200MB ，Fedora 下挂载为 /boot/efi 实际上就是 EFI 分区
    - Windows 主分区
    - Ext4 分区，Fedora 的 boot 分区
    - LUKS + Btrfs 分区，Fedora 主分区
    - 在这些分区间散落着其他各种Windows需要的分区（恢复分区）

### 清理 Windows 11 预留空间

利用 [WinDirStat](https://windirstat.net/) 扫描硬盘，删除不必要的文件，最初我能将 Windows 的主分区减少到 55G，可是为了确保 Windows 的系统可用，我又更新了系统，最初在设置的更新中更新了多次，都提示空间不足，所以我努力的清理空间，几乎将主分区占用下降到 40G ，剩余就有 15G 可惜安装依旧一直失败，即时在过程中根据提示插入外置硬盘用作存放临时文件，依旧是不断的失败。经过简单的搜索，才发现因为是大版本更新，所以更新需要满足一定的系统要求，这个要求就是硬盘空间至少 64G，所以并不是因为剩余空间不够才失败的，而是分配的空间太小了。最后重新将硬盘空间调整到 64G，同时用 Windows 11 升级助手直接升级到最新的大版本，最后终于成功。

潜在的问题，更新完之后主分区的空间从 64G 变成了 63G ，因为有 1G 被划分作为新的恢复分区了，这也许会导致下一次的大版本更新失败。

### 调整 Fedora 分区大小

因为 Windows 更新完有了新的恢复分区，所以我删除在分区表末尾的原有恢复分区，这部分的为分配空间也可以并入 Fedora 的分区。缩小了 Windows 主分区后，有大概 16G 的为分配空间，这部分的空间是在 Ext4 的 boot 分区前的，我利用 Aomei 的硬盘分区工具，将 Ext4 分区移动到未分配空间的最左侧，紧接着 Windows 的主分区和恢复分区。这时未分配空间依旧位于 Fedora 的左侧，Fedora 的主分区是利用 LUKS 加密的分区，同时文件系统是 Btrfs，Aomei 硬盘工具虽然显示这有一个分区，但是却没能识别出具体是什么文件系统，同时也显示空间全部可用，所以不敢直接在 Windows 下移动，实际上这个凑巧的谨慎而是很有必要的，后续尝试中我也在 USB live 的环境中使用 GParted ，同样的也是不能直接移动分区，再扩容。

到现在为止，扩张 Fedora 的分区，需要合并 Fedora 主分区前后的未分配空间，合并主分区右侧的未分配空间较为容易，而合并左侧的空间则更加麻烦和困难，也正是这个原因我才决定写这样一篇东西，先从简单的合并开始。

### 合并右侧的未分配空间

需要解锁 LUKS 分区，然后利用 Parted 或 GParted 扩展物理分区，最后再扩展 Btrfs 文件系统大小，这些操作最好在另外一个 Linux 环境或者 USB live 的 Linux 环境中操作，因为涉及到调整 root 分区的大小，所以本系统里应该是无法操作的。

- 在其他 Linux 系统环境中需要解锁 LUKS 分区 `sudo cryptsetup luksOpen /dev/nvme0n1p6 crypt-volume`
- 如果是在另外的环境中，那就要挂载 LUKS 分区 Mapper，以挂载点 ~/test 为例 `sudo mount /dev/mapper/crypt-volumt ~/test`
- 调整 Btrfs 文件系统大小 `sudo btrfs filesystem resize max /`
    - 如果是在另外的环境中执行 `sudo btrfs filesystem resize max ~/test` 进行调整
- 这样两步就完成了 Fedora 主分区右侧的磁盘扩容

### 合并左侧的未分配空间

因为未分配空间在左侧，实际上也就没有办法很容易的直接合并，这一点也是在折腾过程中我才意识到的，好像的确是这样，以前如果要合并左侧空间，那就会花很多时间移动数据，我不确定为什么在这个配置下这样做不行，也许是因为 LUKS ，也许是因为 Btrfs ，我不能确定。总之合并左侧的未分配空间并不容易，上面的两步完全没用。在搜索解决方法的过程中，我寻找到了一个可行的方法，大致的步骤如下。同样的这些步骤也可以在另外的 Linux 环境中实现，我推荐在做任何的操作前先准备一个环境，我手头刚好有之前的 Fedora 36 Live USB 的环境，有这个环境可以确保当操作错误时安心的回退操作。

1. 在未分配空间上新建一个 Btrfs 分区，可以用 Parted 或 GParted
2. 新建一个 LUKS 分区，设置加密密钥，设置 tpm2 自动解锁
    - 应该可以跳过，如果不需要加密？但是我也不能确定，因为合并的对象是加密的分区，这会有什么影响我并不确定。
    
    ```bash
    # 新建并格式化 LUKS 分区
    sudo cryptsetup luksFormat /dev/nvme0n1p7
    # 检查分区是否已经 LUKS 加密
    sudo cryptsetup isLuks /dev/nvme0n1p7 && echo Sucess
    # 输出分区 LUKS 信息
    sudo cryptsetup luksDump /dev/nvme0n1p7
    ```
    
3. 设置分区名，我这里设置为 luks-UUID 实现唯一的名字 
    
    ```bash
    # 取得 LUKS UUID
    sudo cryptsetup luksUUID /dev/nvme0n1p7
    # 设置 LUKS 分区名
    sudo cryptsetup luksOpen /dev/nvme0n1p7 luks-84f5172e-c2d6-4651-812f-0d6271be004a
    # 检查 LUKS 分区情况
    sudo dmsetup info luks-84f5172e-c2d6-4651-812f-0d6271be004a
    ```
    
4. 修改 `/etc/crypttab` 确保分区在启动时会被解锁
    1. 利用编辑器打开 crypttab 新增 LUKS 分区信息 `sudo nvim /etc/crypttab`
    2. 我直接参考原有 LUKS 分区进行新增（黏贴复制，修改名字和 UUID）
    3. `none` 表示不使用密钥文件， `tpm2-device=auto` 用 tpm 设备解密，`discard` 用于 SDD 设备
    
    ```bash
    luks-3bf2404b-95b5-4c38-b51f-7e8d4611d173 UUID=3bf2404b-95b5-4c38-b51f-7e8d4611d173 none tpm2-device=auto,discard
    luks-84f5172e-c2d6-4651-812f-0d6271be004a UUID=84f5172e-c2d6-4651-812f-0d6271be004a none tpm2-device=auto,discard
    ```
    
5. 运行 `sudo dracut -f` 重新构造 `intiramfs`
6. 修改 `/etc/default/grub` 增加新的 LUKS 分区到内核启动参数，确保在 Grub 阶段能够解锁分区，我以为是运行 `dracut` 自动会新增的，可惜并不是，遗漏这一步导致我一直无法成功
    1. 在 `GRUB_CMDLINE_LINUX` 字段新增参数 `rd.luks.uuid`，原有应该就有一个同样的参数，值为原有的 LUKS 分区名，直接将新的 LUKS 分区以同样的形式增加即可
    2. 根据参考，多个 `rd.luks.uuid` 和 `/etc/crypttab` 将会在系统启动阶段激活其中的设备，如果不修改，那么将无法在启动阶段使用新的分区，自然也就无法启动成功
    3. 参考：https://www.freedesktop.org/software/systemd/man/latest/systemd-cryptsetup-generator.html
        
        ```bash
        GRUB_CMDLINE_LINUX="rd.luks.uuid=luks-3bf2404b-95b5-4c38-b51f-7e8d4611d173 rd.luks.uuid=luks-84f5172e-c2d6-4651-812f-0d6271be004a rhgb quiet"
        ```
        
7. 运行 `sudo grub2-mkconfig -o /etc/grub2.cfg` 更新 `grub2` 配置文件
    1. 我记得以前是要运行 `sudo grub2-mkconfig -o /boot/efi/EFI/fedora/grub.cfg` 的，但是在 Fedora 文档中关于更新 Grub 配置的部分提及，在新版本的 Fedora 中，`/boot/efi/EFI/fedora/grub.cfg` 只是一个小文件，这个文件简单的指向 `/boot/grub2/grub.cfg`，同时 `/etc/grub2.cfg` 也是这个文件的一个链接
8. 将新建的 Btrfs 分区添加到原有的 Btrfs 文件系统中 `sudo btrfs device add /dev/mapper/luks-84f5172e-c2d6-4651-812f-0d6271be004a /`
9. 重启验证
10. 如果系统无法启动，说明以上步骤中存在错误或者遗漏，进入准备好的其他 Linux 环境撤销操作，在尝试过程中我一直没有将新的 LUKS 分区增加到内核启动参数中，系统无法启动，准备好的 Linux 环境就救命了。在这里也记录一下我的撤销步骤。
    1. 进入环境
    2. 解锁两个 LUKS 分区
        
        ```bash
        sudo cryptsetup luksOpen /dev/nvme0n1p6 origin
        sudo cryptsetup luksOpen /dev/nvme0n1p7 new
        ```
        
    3. 挂载其中一个 Btrfs 分区到文件系统（任意一个都没事，因为之前操作中合并了）
        
        ```bash
        mkdir ~/test
        sudo mount /dev/mapper/origin ~/test
        ```
        
    4. 从 Btrfs 文件系统中删除新增的 Luks/Btrfs 分区
        
        ```bash
        sudo btrfs device remove /dev/mapper/new ~/test
        ```
        
    5. 重启 
11. 备份文件增加 LUKS 可用性
    1. 备份 `/etc/fstab`
    2. 备份 `/ect/crypttab`
    3. 备份 `/ect/default/grub`
    4. 备份 LUKS header
