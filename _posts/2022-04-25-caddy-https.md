---
title: "Automatic https your site with Caddy"
categories:
  - Blog
tags:
  - web
  - caddy
  - https
---
![Caddy2](/assets/images/caddy.png "caddy")
# Caddy是什么？
Caddy是一款用Go编写的开源web server，虽然它又造了一个轮子，且未拥有其它web server的全部功能，但其依然能在短时间内坐拥海量粉丝，究其缘由，非其自动申请并维护ssl证书功能莫属（单文件安安装,动态reload配置,模块化可以插拔架构等特性也功不可没）,因此本文主要基于此功能进行介绍。

## Caddy如何实现自动https?
- Caddy会通过在后台运行证书管理程序(不会block其他请求处理)进行证书申请
- 证书管理进程会通过ACME协议从兼容此协议的CA处申请证书（默认会从Let's Encrypt和ZeroSSL申请）
  - 支持HTTP challenge
  - 支持TLS-ALPN challenge
  - 支持DNS challenge
- 支持在多个CA间failover

## Caddy如何处理失败的证书申请请求？
- 首先会在短暂的停顿后进行retry
- 然后在短暂的停顿后切换为下一种可用的challenge type
- 在所有可用的challenge type都试过，依然未能成功，会继续在下一个配置的issuer处申请
  - Let's Encrypt
  - ZeroSSL
  - Other issuers from configuration
- 在试过了所有的issuers之后,请求的间隔会进行指数级增长。
  - Maximum of 1 day between attempts
  - For up to 30 days

## Caddy何时会申请证书?
- 默认会在启动后自动申请配置中所含域名的证书（适用于事先已经知道所有需要申请证书的域名信息，且都已经将dns解析指向caddy server）

```
{
example.com

root * /var/www/wordpress
php_fastcgi unix//run/php/php-version-fpm.sock
file_server
}
```

- 当接收到TLS handshake时，根据的SNI中的server name按需申请(适用于事先不知道所有域名信息，域名不在自己的控制范围内或者当前郁闷解析并未指向caddy server),此种模式下，我们可以通过增加ask配置来判断域名的合法行，以防止恶意的访问触发大量的证书申请请求，导致正常的申请请求受限。

```
{
        on_demand_tls {
                ask "http://domain.authorization.server/"
                interval 10s
                burst 100
        }
 
}
```

## Caddy将证书保存在哪里？
Caddy天生设计为集群模式，可无限水平扩展，其运行模式根据caddy.storage配置决定，默认使用file_system，以单机模式工作,当多台caddy server配置使用相同的storage时，它们便会共享资源，且coordinate certificate management as a cluster，不同模式下的storage module:
- 单机模式
  - file_system
- 集群模式
  - s3
  - redis
  - consul
  - dynamodb

> ACME: [Automatic Certificate Management Environment](https://en.wikipedia.org/wiki/Automatic_Certificate_Management_Environment)

> SNI: [SNI](https://en.wikipedia.org/wiki/Server_Name_Indication)

