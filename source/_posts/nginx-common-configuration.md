---
title: Nginx 常见配置
date: 2020-07-19 18:22:26
tags: ["Nginx"]
categories: ["Nginx"]
---

最近接触Nginx 配置比较多，所以整理一下，方便后面回顾。

<!-- more -->

## 多站点配置
如果一台服务器，需要配置多套站点，推荐使用 `IP + 端口`配置站点，然后使用反向代理指向端口。

站点配置
```
server {
    listen       40001;
    
    location ~ \.php {
        ...
    }
    location / {
        ...
	  }
}
```

多站点配置
```
// 站点1 
server {
 	 	server_name  yoursite.com;
		listen 80;

 	 	location / {
		    proxy_pass http://127.0.0.1:40001;
			  index  index.html index.htm index.jsp index.js;
		}
}

// 站点2
server {
 	 	server_name  yoursite2.com;
		listen 80;

 	 	location / {
		    proxy_pass http://127.0.0.1:40001;
			  index  index.html index.htm index.jsp index.js;
		}
}
```

## 反向代理
反向代理其实已经在上面的配置中出现过了，多站点配置的原理就是利用反向代理。

```
server {
 	 	server_name  yoursite2.com;
		listen 80;

 	 	location / {
		    proxy_pass http://127.0.0.1:40001;
			  index  index.html index.htm index.jsp index.js;
		}
}
```

## SSL 配置
申请好证书之后，将其放在服务器上，然后编辑Nginx 配置：

```
server {
    server_name  yoursite.com;
  
    listen 443 ssl;
    ssl on;
    ssl_certificate ssl_0123cp_net/full_chain.pem;  // 证书所在路径
    ssl_certificate_key ssl_0123cp_net/private.key;  // 证书对应的私钥所在路径
  
    location / {
        proxy_pass http://127.0.0.1:40001;
        index  index.html index.htm index.jsp index.js;
    }
}
```

## http重定向
配置好 `https`之后，还需要做一件事，才能保证 `https`能够正常访问。

因为访问任何一个网站时，默认使用的是`http`协议，所以需要在`Web Server`中配置`http` 自动跳转 `https`。
```
server {
    server_name yoursite.com;
  
    listen 80;
    rewrite ^(.*) https://$server_name$1 permanent;
}
```
