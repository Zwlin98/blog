---
title: Certbot 证书有关设置
date: 2020-01-31 10:34:19
---
更新证书的步骤比较麻烦，特此记录，因为我用的是 `certbot` 提供的泛域名的证书功能，我只会手动申请。而且貌似 dnspod 并不支持 certbot 提供的插件，故而只能手动更新。

<!--more-->
1. 申请泛域名证书
```bash
certbot certonly --preferred-challenges dns --manual  \
-d "example.com" \
--server https://acme-v02.api.letsencrypt.org/directory
```
2. 根据提示添加 DNS 的 TXT 记录
3. 生成证书
4. 更新 nginx 的证书设置，在最顶层统一设置证书，修改
```nginx
# /etc/nginx/nginx.conf 
http {
     include       /etc/nginx/mime.types;
     default_type  application/octet-stream;
 
     ssl_certificate       /etc/ssl/certs/fullchain.pem;
     ssl_certificate_key   /etc/ssl/certs/privkey.pem;
 
     ssl_session_timeout 5m;
     ssl_prefer_server_ciphers on;
     ssl_ciphers EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256::!MD5;
     ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
 
     log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                       '$status $body_bytes_sent "$http_referer" '
                       '"$http_user_agent" "$http_x_forwarded_for"';
 
     access_log  /var/log/nginx/access.log  main;
 
     sendfile        on;
     #tcp_nopush     on;
 
     keepalive_timeout  65;
 
     #gzip  on;
 
     include /etc/nginx/conf.d/*.conf;
 }
```
5. 设置或更新软链接
```bash
ln -snf /etc/letsencrypt/live/zwlinc.com-0001/fullchain.pem /etc/ssl/certs/fullchain.pem 
ln -snf /etc/letsencrypt/live/zwlinc.com-0001/privkey.pem /etc/ssl/certs/privkey.pem 
```
