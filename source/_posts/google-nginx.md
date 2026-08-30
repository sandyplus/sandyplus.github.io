---
title: "如何用搬瓦工反向代理谷歌"
date: 2017-03-13 00:50:59
updated: 2017-03-13 00:50:59
---

## 最终成果

[谷歌](http://google.sandyplus.com)

## 前提

先将搬瓦工安装debian 8.0 x64系统（安装shadowsocks的时候推荐用[go版一键安装脚本](https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-go-debian)）
注意，在修改系统设置之前，最好先把系统备份一下：点击左侧的snapshot（可以免费备份五个，保存一个月）

## 安装nginx（参考[教程](https://zhgcao.github.io/2016/06/09/nginx-reverse-proxy-google/))

安装流程基本参考教程，主要有几点需要注意：
先要设置一下：

```text
wget http://nginx.org/keys/nginx_signing.key
apt-key add nginx_signing.key
rm nginx_signing.key
```

- 修改nginx的源：

  ```text
  vi /etc/apt/sources.list.d/nginx.list
  ```
  - 如果是debian 8.x （代号jessie，其他版本的代号见[网站](https://www.debian.org/releases/))，则添加以下内容：

    ```text
    deb http://nginx.org/packages/debian/ jessie nginx
    deb-src http://nginx.org/packages/debian/ jessie nginx
    ```
- 安装一些必备的包：

  ```text
  apt-get install libpcre3 libpcre3-dev
  apt-get install zlib1g zlib1g-dev openssl libssl-dev
  apt-get install libxml2 libxml2-dev libxslt1-dev
  apt-get install libgd-dev libgeoip-dev
  apt-get install -y gcc g++ make automake
  apt-get install git
  apt-get install dnsutils
  ```
- 用源码安装时的命令如下(注意：nginx 1.9.5 已经没有了 –with-http_spdy_module ，取代的是 –with-http_v2_module)：

  ```text
  cd nginx-1.10.3
  ./configure \
  --with-cc-opt='-g -O2 -fPIE -fstack-protector-strong -Wformat -Werror=format-security -D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-Bsymbolic-functions -fPIE -pie -Wl,-z,relro -Wl,-z,now' --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-debug --with-pcre-jit --with-ipv6 --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_addition_module --with-http_dav_module --with-http_geoip_module --with-http_gzip_static_module --with-http_image_filter_module --with-http_v2_module --with-http_sub_module --with-http_xslt_module --with-mail --with-mail_ssl_module \
  --add-module=../ngx_http_substitutions_filter_module \
  --add-module=../ngx_http_google_filter_module

  make
  make install
  ```
- 复制到/usr/sbin目录下：

  ```text
  cp -rf objs/nginx /usr/sbin/nginx
  ```
- 新建文件夹，以免报错

  ```text
  mdkir -p /var/lib/nginx
  mdkir -p /var/lib/nginx/body
  mdkir -p /var/lib/nginx/fastcgi
  ```
- 测试是否成功：

  ```text
  nginx -t
  ```
  - 在chrome里面输入vps的IP，查看是否出现nginx的安装成功的提示
  - 如果修改配置后，只需要reload就可以了

    ```text
    nginx -s reload
    ```

## 配置nginx,反代谷歌

```text
vi /etc/nginx/nginx.conf
```

#### 如果没有域名(最好申请一个域名，没有域名没有办法申请SSL证书）：

- 找到nginx.conf的第一个 server 增加两行配置，如下：
  - 增加两行

    ```text
    server {
            listen       80;
            server_name  localhost;
            resolver 8.8.8.8;    //加这句
            location / {
                root   html;
                index  index.html index.htm;
                google on;        //加这句
            }
            .......
    }
    ```
  - 然后重启nginx：

    ```text
    nginx -s reload
    nginx -t
    ```

#### 如果有域名：

- 先将域名指向vps的IP地址（在dnspod里面添加A记录，可能需要10min-24小时才生效）
- 再将vps里面nginx配置相应位置设置成域名（参考我的[配置](https://github.com/sandyplus/nginx_config/blob/master/nginx.conf)）
- 注意，启用SSL需要在nginx.conf相应位置添加证书
  - 可以申请免费SSL证书，如STARTSSL（不推荐，操作太麻烦了），let’s encrypt（推荐）
  - 如果不想麻烦，直接在linux生成个自签名证书，然后在nginx.conf修改相应参数（不推荐，访问时会提供有风险）

    ```text
    openssl genrsa -out sandyplus.pem 2048
    openssl req -new -key sandyplus.pem -out sandyplus.csr
    ```

## 如何在let’s Encrypt中申请证书

- 参考申请let’s encrypt的[教程](https://hitian.info/notes/2017/02/16/acme-sh-create-letsencrypt-certificates-with-dns-api/).
- 先在DNSPOD中生成API ID和API key：在安全设置-API token中生成（注意截图）
- 然后，把这两行手动添加到/root/.acme.sh/account.conf

```text
DP_Id="your_id"
DP_Key="Your_token"
```

- 参考教材做后续内容，会出现以下提示

  ```text
  -----END CERTIFICATE-----
  [Sat Mar 11 03:51:45 EST 2017] Your cert is in  /root/.acme.sh/sandyplus.com/sandyplus.com.cer
  [Sat Mar 11 03:51:45 EST 2017] Your cert key is in  /root/.acme.sh/sandyplus.com/sandyplus.com.key
  [Sat Mar 11 03:51:45 EST 2017] The intermediate CA cert is in  /root/.acme.sh/sandyplus.com/ca.cer
  [Sat Mar 11 03:51:45 EST 2017] And the full chain certs is there:  /root/.acme.sh/sandyplus.com/fullchain.cer
  ```

## 设置nginx，修改nginx.conf中的证书位置：

```text
ssl_certificate /root/.acme.sh/sandyplus.com/fullchain.cer;
ssl_certificate_key /root/.acme.sh/sandyplus.com/sandyplus.com.key;
```

- 以下可选，让网站评级从B级上升为A级，参考[网站](https://www.textarea.com/zhicheng/fenxiang-yige-https-a-di-nginx-peizhi-320/) 和[论坛](https://ruby-china.org/topics/31983) ：
  去这个[网站](https://www.ssllabs.com/ssltest/analyze.html)评级

## 如何替换google页面的“简单中文”（参考[博客](https://blog.nyan.im/posts/877.html))

- 确保安装了模块ngx_http_substitutions_filter_module

  ```text
  nginx -V
  ```

或者：

```text
2>&1 nginx -V | tr ' '  '\n'|grep sub
```

- 在location里面加一句：

  ```text
  subs_filter logo-subtext">简体中文 'logo-subtext">博客:sandyplus.com' ;
  ```
- 注意，如是vim打开中文是乱码的话，需要修改vimrc的配置
  - 先找到$HOME/.vimrc，找不到就从/etc/vim/vimrc复制到home目录下的.vimcr
  - 在vimrc里面添加设置

    ```text
    set fileencodings=utf-8,gb2312,gb18030,gbk,ucs-bom,cp936,latin1
    set enc=utf8
    set fencs=utf8,gbk,gb2312,gb18030
    ```
