+++
title = "申请Let's Encrypt SSL证书"
slug = "install-lets-encrypt-ssl-certificate"
date = 2018-09-16T14:11:04.000Z
excerpt = "申请Let's Encrypt SSL证书，自己的大概一个流程，有很多也是尚未完全理解的"

[taxonomies]
tags = ["Tech", "Coding"]
+++

需要知识基础：Http、Https、SSL、域名解析、手动设置域名解析服务器、Nginx配置编写

- 安装Nginx
- 域名解析：Dnspod加速解析
- 安装Certbot客户端

申请证书的三种方式

dns-01：给域名添加一个 DNS TXT 记录。

http-01：在域名对应的 Web 服务器下放置一个 HTTP well-known URL 资源文件。

tls-sni-01：在域名对应的 Web 服务器下放置一个 HTTPS well-known URL 资源文件。

针对单一域名，需要指定网站的根目录：

    $ sudo certbot --authenticator webroot --installer nginx
    

这样会自动在在域名对应的 Web 服务器下放置一个 well-known URL 资源文件。

通配符证书只能使用dns-01

通配符证书：

    sudo certbot certonly  -d "*.livexia.xyz" --manual --preferred-challenges dns-01  --server https://acme-v02.api.letsencrypt.org/directory
    

需要给域名增加一个txt，dns解析，具体输入命令后会有细节

利用dig查询解析是否成功，解析成功在进行下一步操作

    $ dig  -t txt _acme-challenge.xxx.com @8.8.8.8 
    

90天会到期，需要进行续期。

自动renewal：

    $ sudo certbot renew --dry-run
    

Nginx配置文件的编写：

    server {
        server_name xxx.com;
        listen 443 http2 ssl;
        ssl on;
        ssl_certificate /etc/cert/xxx.com/fullchain.pem;
        ssl_certificate_key /etc/cert/xxx.com/privkey.pem;
        ssl_trusted_certificate  /etc/cert/xxx.com/chain.pem;
    
        location / {
          proxy_pass http://127.0.0.1:6666;
        }
    }
    

> [https://letsencrypt.org/getting-started/](https://letsencrypt.org/getting-started/)
> [https://certbot.eff.org/](https://certbot.eff.org/)
> [https://www.dnspod.cn/console/dns/livexia.xyz](https://www.dnspod.cn/console/dns/livexia.xyz)


**Update:** 2021-12-20，准备移除Ghost，所以记录之前对证书自动刷新的命令，留有记录。

执行 crontab -e 增加如下内容，保证每周日0点对证书进行刷新

    0 0 * * 0 sudo /home/ghost-mgr/.acme.sh/acme.sh --force --renew --home /etc/letsencrypt --domain [domain] --webroot /var/www/ghost/system/nginx-root --reloadcmd "nginx -s reload" --accountemail [email] --debug &> /home/ghost-mgr/renew.log


> crontab参考网址：https://crontab.guru/#0_0_*_*_0s