+++
title = "Ghost博客升级"
slug = "ghostv4-update"
date = 2021-12-03T13:03:29.000Z
excerpt = "很长时间都没有看过我的博客了，今天偶然用ipad连上了服务器，就想着升级一下吧，由于涉及到很多曲折的历程，于是特此做个简单记录。"

[taxonomies]
tags = [ "Tech" ]
+++

**背景**：很长时间都没有看过我的博客了，今天偶然用ipad连上了服务器，就想着升级一下吧，由于涉及到很多曲折的历程，于是特此做个简单记录。

服务器是搭建在DigitalOcean的vps上的，远程连接之后需要切换用户，并进入/var/www/ghost目录进行操作升级。

**Step1**：目录下执行 `ghost update`，提示node.js版本不够，于是需要先升级nodejs。

**Step2**：升级NodeJs

我不太熟悉NodeJs，于是找了一圈没找到很好的方法，结果我发现服务器是Ubuntu的18.04版本，决定先吧服务器升级到20.04。

**Step3**：升级服务器到20.04

参考：[https://www.digitalocean.com/community/tutorials/how-to-upgrade-to-ubuntu-20-04-focal-fossa](https://www.digitalocean.com/community/tutorials/how-to-upgrade-to-ubuntu-20-04-focal-fossa)

执行：`sudo do-release-upgrade`

执行时程序会创建一个screen session保证不会因为ssh断开而中断升级，期间的确发生了ssh断链的情况，安装中会提示升级各种服务，包括mysql，这是一个坑，我都选择了跟随发行版。

**Step4**：在20.04下继续升级，nodejs并没有自动升级，所以ghost继续报错，然后我仔细阅读了如何升级nodejs，方法是`sudo n 12.22.1`。我根据提示升级了多个版本的nodejs，但是最后选择了这个版本。

因为较长时间没有升级，我是从v3升级到v4，版本跨度较大，默认自动升级到v4，当我安装了nodejs 17.x 版本的时候，提示我需要先升级到最新版本的v3。使用命令`ghost update v3`，提示我nodejs版本不兼容，于是参照提示又安装了12.x的版本。最后成功升级v3，然后执行`ghost update` 升级v4，升级没有其他的错误，但是运行发生了严重的错误。

**Step5**：运行`ghost start`时，提示数据库的一个错误，根据关键词我找到了一篇解决的社区讨论，参见：[https://forum.ghost.org/t/ghost-4-1-0-errored-during-boot/21006/5](https://forum.ghost.org/t/ghost-4-1-0-errored-during-boot/21006/5) 。利用提及的方法，我修改了数据库的默认collation，参见如下sql代码。

    SELECT SCHEMA_NAME,
    DEFAULT_CHARACTER_SET_NAME,
    DEFAULT_COLLATION_NAME
    FROM INFORMATION_SCHEMA.SCHEMATA
    WHERE SCHEMA_NAME='ghost_production';
    
    ALTER DATABASE `ghost_production`
    DEFAULT CHARACTER SET utf8mb4
    DEFAULT COLLATE utf8mb4_general_ci;
    

执行之后，错误信息发生了变化。

**Step6**：继续解决新出现的错误。

根据错误信息我找到了另外一篇的社区讨论，参见：[https://forum.ghost.org/t/unable-to-upgrade-ghost-from-v4-2-0-to-v4-3-0-cascade-unknown-code-please-report/22086/56](https://forum.ghost.org/t/unable-to-upgrade-ghost-from-v4-2-0-to-v4-3-0-cascade-unknown-code-please-report/22086/56) 。根据这篇讨论和上一篇讨论，我大致明白发生了什么，简单说下。在我升级服务器的时候，MySQL也升级了，而在8.0中数据库的默认collation是utf8mb4_0900_ai_ci，而遗留的数据表则是utf8mb4_general_ci。所以我需要修改mysql的配置，如讨论中所说，我也需要修改已有数据库的默认collation，但是在我改完也添加完配置文件后，错误依旧。

**Step7**：因为MySQL较长时间没有用了，很多语句都忘记了使用方式，但是经过两篇文章我大概明白了为什么，即使我运行`ghost update --force`强制重新升级也无法解决错误，因为在修改完配置和数据库设置后，就算我重新强制升级，也只是在创建新表的时候，采用了新的collation，而那些老的表，则无法自动修改，于是我需要一个一个表进行修改。

**Step8**：修改数据表的collation。

参考：[https://betterprogramming.pub/how-to-check-and-change-the-collation-of-mysql-tables-6095fada0ebd](https://betterprogramming.pub/how-to-check-and-change-the-collation-of-mysql-tables-6095fada0ebd)

mysql中执行命令 `SELECT TABLE_CATALOG, TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME, COLLATION_NAME FROM INFORMATION_SCHEMA.COLUMNS where COLLATION_NAME like 'utf8mb4_0900_ai_ci';` 找到仍旧是采用utf8mb4_0900_ai_ci的所有与ghost相关的数据表。

执行命令 `ALTER TABLE <table_name> CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;` 把所有与ghost相关的采用utf8mb4_0900_ai_ci的数据表，进行修改。修改后 ghost 启动正常。

**参考链接：**

1. [https://forum.ghost.org/t/ghost-4-1-0-errored-during-boot/21006/5](https://forum.ghost.org/t/ghost-4-1-0-errored-during-boot/21006/5)
2. [https://forum.ghost.org/t/unable-to-upgrade-ghost-from-v4-2-0-to-v4-3-0-cascade-unknown-code-please-report/22086/56](https://forum.ghost.org/t/unable-to-upgrade-ghost-from-v4-2-0-to-v4-3-0-cascade-unknown-code-please-report/22086/56)
3. [https://github.com/TryGhost/Ghost-CLI/issues/1359](https://github.com/TryGhost/Ghost-CLI/issues/1359)
4. [https://betterprogramming.pub/how-to-check-and-change-the-collation-of-mysql-tables-6095fada0ebd](https://betterprogramming.pub/how-to-check-and-change-the-collation-of-mysql-tables-6095fada0ebd)
