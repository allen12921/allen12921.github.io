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

# Caddy如何实现自动https?
Caddy会在后台运行证书管理器程序，它会通过ACME协议从兼容此协议的CA处申请证书（默认会从Let's Encrypt和ZeroSSL申请）


> ACME: Automatic Certificate Management Environment
