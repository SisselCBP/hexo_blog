---
title: 搭blog服务器
date: 2019-03-17 06:46:28
tags:
- 服务器
- nginx
---

"我搭了一个服务器，吼吼"

在vultr租了3.5刀一个月的服务器，然后要把他折腾成 个人网站/博客+酸酸两用的服务器嗝，记录一下都做了哪些配置。

<!-- more -->

> 这写文章的风格咋和一个大一小朋友一样哈哈
 
## nginx 

首先是购买域名，买了[www.aisissel.cn](www.aississl.cn) ，和自己公众号[AiSissel]一样耶，稍微规划一下二级域名

- blog.aisissel.cn 用来做博客
- [*.\]*aisissel.cn 其他用来做个人主页

然后酸酸开在高端口上，还要去签一个证书，[腾讯云-申请证书](https://console.cloud.tencent.com/ssl) ，启动~

![](https://md.byr.moe/uploads/upload_287a5e563f04f1883ca78fcd99c07eb7.png)

然后看看配一下nginx~

![](https://md.byr.moe/uploads/upload_0e947d7f43b85fab7c0b8d6ba4b7ff5c.png)

写一下配置哈哈

```nginx
server {
    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;

    server_name blog.aisissel.cn;

    ssl on;
    ssl_certificate /xxxxx/blog.aisissel.cn.pem;
    ssl_certificate_key /xxxxx/blog.aisissel.cn.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers AESGCM:ALL:!DH:!EXPORT:!RC4:+HIGH:!MEDIUM:!LOW:!aNULL:!eNULL;
    ssl_prefer_server_ciphers on;

        location / {
                proxy_pass http://127.0.0.1:4000;
        }
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name www.aisissel.cn aisissel.cn;
    ssl on;
    # 这里这里，为什么我两个域名用了同一个证书呀？
    ssl_certificate /xxxxx/aisissel.cn.pem;
    ssl_certificate_key /xxxxx/aisissel.cn.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers AESGCM:ALL:!DH:!EXPORT:!RC4:+HIGH:!MEDIUM:!LOW:!aNULL:!eNULL;
    ssl_prefer_server_ciphers on;
   
    root /var/www/html;
    
    index index.html index.htm index.php;

    location / {
    }
}


server {
    listen       80;
    server_name  aisissel.cn www.aisissel.cn blog.aisissel.cn;
    rewrite ^(.*) https://$host$1 permanent;
}
```

有一个小问题，为什么我两个域名，用了同一个证书，还没问题呢？

## 再配hexo

hexo是一个基于nodejs的，支持md，非常好用的博客框架，总之就是非常厉害吼吼。

https://hexo.io/zh-cn/

安装非常简单，配一些基本的内容，装了些主题和插件就ok了。

主题：Archer


