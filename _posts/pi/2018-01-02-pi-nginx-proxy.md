---
layout: default
title: 利用nginx实现反向代理
tags: pi
---

## 这是啥？

在使用树莓派充当个人服务器的过程中，由于网络运营商的问题，无法给我的树莓派分配公网ip，所以需要在第三方服务提供商购买内网穿透服务。本人选用的是 [natapp](https://natapp.cn/) 。最便宜的低至5元/月。但是在使用穿透服务的时候，有一个问题，那就是每一个隧道只能对应一个端口，那么也就是说，我在树莓派上部署多个服务的时候，就要购买多个natapp的隧道。这个就很不愉快了。所以，我决定使用nginx来进行反向代理。

### 整体的结构图应该是这样(没仔细画):

![截图]({{ site.url }}/source/images/pi-nginx-proxy/nginx01.png)

- 第一步：购买natapp隧道部分我就不细表了，官网都有文档教程。
- 第二步：在树莓派上安装nginx，并配置反向代理。
    - 安装nginx我也不细说了。
    - 主要写一下怎么配置反向代理。

### nginx反向代理配置（多用于一个外网端口，多个内网服务端口）
首先我在本地用koa开启了两个http服务。
端口分别是3000和3001
返回的是当前访问的url和服务的端口号

![截图]({{site.url}}/source/images/pi-nginx-proxy/nginx02.png)
![截图]({{site.url}}/source/images/pi-nginx-proxy/nginx03.png)

### 然后nginx的配置文件
```nginx
server {
  listen 80;
  location /service1/ {
    proxy_pass http://127.0.0.1:3000/;
  }

  location /service2/ {
    proxy_pass http://127.0.0.1:3001/;
  }
}
```
### 最后重启nginx

![截图]({{ site.url }}/source/images/pi-nginx-proxy/nginx05.png)
![截图]({{ site.url }}/source/images/pi-nginx-proxy/nginx06.png)

### nginx配置（一个外网端口，多个静态网站）

- 为了方便测试，我在本地创建了两个文件夹，作为两个网站的根目录。

![截图]({{ site.url }}/source/images/pi-nginx-proxy/nginx04.png)

- 开始配置nginx：

```nginx
server {
  listen 80;
  location ^~ /service1/ {
    alias /Users/qiusz/qiushangzhe/mycode/testcode/web01/;
    index index.html;
  }

  location ^~ /service2/ {
    alias /Users/qiusz/qiushangzhe/mycode/testcode/web02/;
    index index.html;
  }
}

```


## 额外的

这里着重说一下location中的alias，当这个地方写root 的时候，当你访问`127.0.0.1/service1/index.html`的时候，nginx访问你本地的路径是`/Users/qiusz/qiushangzhe/mycode/testcode/web01/service1/index.html` 而使用alias的时候，同样前提下，nginx访问你本地的文件路径则是`/Users/qiusz/qiushangzhe/mycode/testcode/web01/index.html` 注意这个区别。具体需求用不同的方案配置。

- 关于root和alias的区别，参考 http://blog.csdn.net/u011510825/article/details/50531864
