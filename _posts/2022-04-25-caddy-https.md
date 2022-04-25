---
title: "Automatic https your site with Caddy"
categories:
  - Blog
tags:
  - web
  - caddy
  - https
---
# Caddy是什么？
Caddy是一款用Go编写的开源web server，虽然它又造了一个轮子，且未拥有其它web server的全部功能，但其依然能坐拥海量粉丝，究其缘由，非其自动申请并维护ssl证书功能莫属。

## Caddy如何实现自动https?
- Caddy会通过在后台运行证书管理程序(不会block其他请求处理)
- 证书管理进程会通过ACME协议从兼容此协议的CA处申请证书（默认会从Let's Encrypt和ZeroSSL申请）
  - 支持HTTP challenge
  - 支持TLS-ALPN challenge
  - 支持DNS challenge

## Caddy如何处理失败的证书申请请求？
- Caddy retries once after a brief pause just in case it was a fluke
- Caddy pauses briefly, then switches to the next enabled challenge type
- After all enabled challenge types have been tried, it tries the next configured issuer
  - Let's Encrypt
  - ZeroSSL
- After all issuers have been tried, it backs off exponentially
  - Maximum of 1 day between attempts
  - For up to 30 days

## Caddy何时会申请证书?
- 默认会在启动后自动申请配置中所含域名的证书
```
example.com

root * /var/www/wordpress
php_fastcgi unix//run/php/php-version-fpm.sock
file_server
```
- 当a TLS handshake is received for a server name (SNI)时按需申请
```
{
        on_demand_tls {
                ask "http://domain.authorization.server/"
                interval 10s
                burst 100
        }
 
}
```

> ACME: Automatic Certificate Management Environment

