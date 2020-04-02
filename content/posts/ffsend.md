---
title: "Ffsend"
date: 2020-04-02T14:56:02+08:00
showDate: true
draft: false
tags: ["blog","story"]
---

# “阅后即焚” 服务

Firefox 为用户提供了一项服务叫做 `send`，可以配置下载次数、过期时间等，实现“阅后即焚”的效果。这个服务也是开源的，我在自己的站点上也部署了一下，这篇文章记录部署过程，以及遇到的一些问题。


## guide

`send` 这个项目有自己的部署文档，见参考链接，与文档不同的是，我这里没有用 Apache 做服务器，我的机器已经使用了 nginx，那么其实直接使用 nginx 做反向代理即可。


### 流程

1. 在 DNS 解析上那里配置一条 A 记录 `*` 指向到服务器 IP。之后便可以支持二级域名了。
2. nginx 做配置，在 `/etc/nginx/nginx.conf` 里面可以看到，`http` 块下面引入了 `sites-enabled`
3. 在 `sites-enabled` 下面添加反向代理的配置

创建文件 `sites-enabled/ffsend` 内容如下

```
server {
        listen 80;
        server_name send.hexyoungs.club;

        location / {
                proxy_pass http://127.0.0.1:1443;
                proxy_redirect off;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "Upgrade";
        }
}
```

下面几行的 `Upgrade` 的 header 是为了支持 websocket，之前没有添加时，websocker 链接是无法建立的，因此 send 服务是不会上传文件的。


完成上述步骤后，检查一下 nginx 的配置是不是有效

>sudo nginx -t -c /etc/nginx/nginx.conf

在这里的时候，由于我的域名太长了，这里爆了个错，需要添加一行

> server_names_hash_bucket_size  64;

在默认配置的 http 模块下面。

最后，利用 certbot 添加 https 支持就好，certbot 会检测并修改上面的配置。

> sudo certbot --nginx

## 尝试

访问 https://send.hexyoungs.club/ 试试吧


## 参考链接

- [send](https://github.com/mozilla/send)
- [send 部署文档](https://github.com/mozilla/send/blob/master/docs/deployment.md)
- ["could not build the server_names_hash, you should increase server_names_hash_bucket_size"](https://stackoverflow.com/questions/13895933/nginx-emerg-could-not-build-the-server-names-hash-you-should-increase-server)
- [certbot](https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx)
