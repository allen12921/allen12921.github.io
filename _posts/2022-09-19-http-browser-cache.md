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
- 强缓存
- 协商缓存

# 强缓存:
所谓强缓存，即是浏览器在本地存在未过期的缓存数据的情况下，不向服务器发送请求，直接使用本地数据。
涉及的Http Headers及其对应的值:
- Expires: 是http1.0规范中的header，其值是一个GMT格式的时间字符串，代表资源过期时间，例如: Mar, 06 Sep 2022 11:47:02 GMT
- Cache-Control: 是http1.1规范中的header,它可能包含多个值，其中max-age=number 来表示资源相对有效期，例如: max-age=600 表示自下载之时起，该资源的有效期是600秒



# 协商缓存
所谓协商缓存，即是浏览器在本地存储中拥有目标资源的情况下，依然需要向服务端发起协商请求，以判断本地缓存是否可继续使用。
涉及的Http Headers及其对应的值:
- Etag和If-None-Match
  - Etag: 其值是服务端根据文件内容生成的文件唯一标识符,包含在请求响应的header里。
  - If-None-Match: 当浏览器中的强缓存过期或者无强缓存相关的hdaers时,会向服务端发起一个协商请求,它包含了If-None-Match这个header，其值为本地缓存中Etag的值，如果服务端发现未改变则返回304，告诉浏览器此任可继续使用本地缓存内容。
- Last-Modified和If-Modified-Since

> 参考
> > [https://www.jianshu.com/p/1a1536ab01f1](https://www.jianshu.com/p/1a1536ab01f1)
<script src="{{ "/assets/js/mermaid.min.js" | relative_url }}"></script>
