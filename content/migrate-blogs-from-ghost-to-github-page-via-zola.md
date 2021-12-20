+++
title = "利用Zola将博客从Ghost迁移到Github Pages"
slug = "migrate-blogs-from-ghost-to-github-page-with-zola"
date = 2021-12-20
excerpt = "目前采用的Ghost博客框架，是搭建在DigitalOcean上的，每个月需要花费我5刀。而我只把这个服务器用作博客，我也不需要服务器用做其他的事情，而且架设在服务器上需要更多的精力进行维护，那么不如就把博客迁移到Github Pages好了，成本低，而且也解决了博客内容备份的问题。"

[taxonomies]
tags = [ "Tech", "Coding" ]
categories = [ "Unfinished" ]
+++
目前采用的Ghost博客框架，是搭建在DigitalOcean上的，每个月需要花费我5刀。而我只把这个服务器用作博客，我也不需要服务器用做其他的事情，而且架设在服务器上需要更多的精力进行维护，那么不如就把博客迁移到Github Pages好了，成本低，而且也解决了博客内容备份的问题。

在写下这篇文章的时候，我习惯于把所有的记录都写在Notion中，不论是私人的还是公开的，后续大概也是如此。这一篇文章也是先在Notion上完成，然后黏贴到md文件中，进一步再上传至Github Pages。目前这篇文章的效果我觉得还是不错的，这也加强了后续我更新博客动力。

> 静态网站生成器：Zola [https://www.getzola.org/documentation/getting-started/overview/](https://www.getzola.org/documentation/getting-started/overview/)
> 

## Zola环境搭建

以下操作均在macOS上完成

Setp1: 安装Zola 

`brew install zola`

Setp2: 初始化Zola仓库 

`zola init ghost.livexia.xyz`

Setp3: 一些简单的zola命令

1. 构建网站  `zola build`
2. 本地预览 `zola serve`
3. 编译检查 `zola check`

## 安装主题

参考的博客的主题应该是手写的？不确定我能在短期内实现，先选择一个主题，后续再看看是否有自己定制化的需求。

简单使用主题 even

`cd themes`

`git clone https://github.com/getzola/even`

在 `config.toml` 中启用:

`theme = "even"`

根据even文档，进一步修改 `config.toml`

并不是特别喜欢这个主题的样式，可能会找时间将我参考的博客的主题进行复制修改

## **Ghost内容迁移**

在Ghost管理页面上导出数据，导出后应该是一个json文件。

利用工具`ghost-to-md` 批量转换：

Step1：安装工具 `npm install -g ghost-to-md`

Step2: 在Ghost管理面板中导出数据，执行命令批量转换。

`ghost-to-md cheng-xu-yuan-de-sheng-huo.ghost.2021-12-20-00-54-57.json`

虽然利用工具能转换大部分的内容到 markdown，但是仍有部分错误需要手动修复。工具转出的markdown 头部的信息是yaml格式，为了和Rust的语言生态保持一致，修改yaml格式的markdown为toml。

进一步梳理博客中的内容，将部分博客文章删除，并分类了未完成的文章。

## **Github仓库架设**

**详细说明参见：**[https://www.getzola.org/documentation/deployment/github-pages/](https://www.getzola.org/documentation/deployment/github-pages/)

Step1：执行git submodule 来包含主题仓库。

`git submodule add https://github.com/getzola/even.git themes/even`

Step2：生成github personal token

Step4：在仓库的 Secert 里添加token，注意要取和下一步Action 脚本中的 Token 一样的名字。

Step5：添加Github Actions

## **域名确定和解析调整**

原有域名为 [https://ghost.livexia.xyz/](https://ghost.livexia.xyz/) 需要调整ghost为其他的什么吗？暂时保持不动吧。

将ghost.livexia.xyz的域名解析进行修改，原有dns解析是在 [https://dns.he.net/](https://dns.he.net/) （注意不是[https://he.net/](https://he.net/)）

1. 删除原有的A类域名解析
2. 添加CNAME域名解析，到github.io 页面
3. 在仓库的Pages页面中增加域名即可，参见："[Managing a custom domain for your GitHub Pages site](https://docs.github.com/en/articles/managing-a-custom-domain-for-your-github-pages-site#configuring-a-subdomain)."

## **安装评论插件**

参考：[https://utteranc.es/](https://utteranc.es/)

在安装教程里，我看见需要修改博客的模版文件，我不想去动even的模版文件，大概率我自己会写一个和参考博客的类似的模版，所以这个部分暂时挂起。

## **DigitalOcean退费**

收到第二封回复，告知我能够将我账户上的未使用的余额进行退款，而我需要在账单到期后进行余额补足。

账户中的所有余额都已经完成退费，而这个月的账单是3.43美元，如果我用PayPal进行支付，我需要最少支付5美元。

参考：[https://fasterthanli.me/articles/a-new-website-for-2020](https://fasterthanli.me/articles/a-new-website-for-2020)