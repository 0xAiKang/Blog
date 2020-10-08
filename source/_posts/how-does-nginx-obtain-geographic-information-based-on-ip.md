---
title: Nginx 如何根据 IP 获取地域信息
date: 2020-10-08 17:29:34
tags: ["Nginx"]
categories: ["Nginx"]
---

最近有一个需求：需要根据用户的IP 获取其国家，然后根据不同国家进行代理转发。

想要完成这个需求，首先第一个解决的问题就是获取IP 地址所对应的地理位置：
1. 这个需求通常是由 GeoIP 这个模块来完成的，Nginx 默认没有开启该模块。
2. GeoIP 是基于 maxmind 提供的数据文件进行分析的，所以还需要下载 maxmind 的数据源文件。

### 安装GeoIP 模块
前面也提到了MaxMind GeoLite Legacy数据库目前已停产，应改用MaxMind GeoIP2或Geolite2数据库和NGINX Plus GeoIP2模块。

Centos：
```
yum install nginx-plus-module-geoip2
```

Ubuntu：
```
sudo apt-get install nginx-plus-module-geoip2
```

然后将 load_module 指令都放在nginx.conf 的配置文件的顶部：
```
load_module modules/ngx_http_geoip2_module.so;
load_module modules/ngx_stream_geoip2_module.so;

http {
    ...
}
```

### 安装 GeoIP 数据源
自从 2019年12月30日开始，就不能直接从MaxMind 上下载了，需要先注册一个账号，获取 license key，然后wget 时带上 key。具体可以查阅[这篇文章](https://blog.maxmind.com/2019/12/18/significant-changes-to-accessing-and-using-geolite2-databases/)。

这是一种安装方式，如果觉得麻烦，可以尝试下面这种方式。

1. 安装依赖：
```
sudo add-apt-repository ppa:maxmind/ppa
sudo apt update
sudo apt install libgeoip1 libgeoip-dev geoip-bin
```

2. 下载源码包，安装应用：
```
sudo wget https://github.com/maxmind/geoip-api-c/releases/download/v1.6.12/GeoIP-1.6.12.tar.gz

sudo tar -zxvf GeoIP-1.6.12.tar.gz
cd GeoIP-1.6.12 && \
./configure && \
make && sudo make install
```

3. 查找`GeoIP.dat`所在位置：
```
sudo find / -name GeoIP.dat
/usr/share/GeoIP/GeoIP.dat
```

4. 在配置文件中使用：
```
geoip_country /etc/nginx/geoip/GeoIP-1.6.12/data/GeoIP.dat;

server {
  ...
  
  location /myip {
        default_type text/plain;
        return 200 "$remote_addr $geoip_country_name $geoip_country_code $geoip_city";
    }
}
```

通过以下变量综合获取地域信息：
* `$remote_addr`：IP地址
* `$geoip_country_name`：国家
* `$geoip_country_code`：对应编码
* `$geoip_city`：城市名称

### 参考链接
* [nginx: [emerg] unknown directive "geoip_country" in /etc/nginx/nginx.conf:23](https://www.geek-share.com/detail/2733570382.html)
* [install GeoIP](https://github.com/maxmind/geoip-api-c)
* [install GeoIP module](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/geoip2/)