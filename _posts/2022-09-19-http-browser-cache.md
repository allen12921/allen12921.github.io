---
title: "Http cache 之 Broswer cache"
categories:
  - Blog
tags:
  - http
  - cache
  - http headers
---
![http-cache](/assets/images/http-cache.png "ch")
# 缘起
最近在部署关于react的js项目，项目文件中涉及到多个固定名称以及大量一次性命名的文件，由于项目迭代频繁，因而在部署的过程中就难免需要考虑客户端（这里特指浏览器）缓存的问题。

对于这两种类型的文件我们需要实施不同的缓存策略:
- 强制缓存
- 协商缓存


> 参考
> > [https://www.jianshu.com/p/1a1536ab01f1](https://www.jianshu.com/p/1a1536ab01f1)
<script src="{{ "/assets/js/mermaid.min.js" | relative_url }}"></script>
