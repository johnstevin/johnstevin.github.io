---
layout: post
title: js跨域请求
categories: js跨域请求
description: js-ajax跨域请求
keywords: js,ajax,跨域请求
---

## js跨域请求

前端对Cross-Origin Resource Sharing 问题（CORS，中文又称'跨域'）应该很熟悉了。

众所周知出于安全的考虑，浏览器有个同源策略，对于不同源的站点之间的相互请求会做限制（跨域限制是浏览器行为，不是服务器行为。）。不过下午想到了一个略无趣的问题：浏览器和服务器到底是如何判定有没有跨域呢？本文主要分两个部分，一是对这个问题的总结，二是nginx下如何配置服务器允许跨域。

#### 同源策略

> 同源指的是域名（或IP），协议，端口都相同，不同源的客户端脚本(javascript、ActionScript)在没明确授权的情况下，不能读写对方的资源。

同源的判定

以 http://www.example.com/dir/page.html 为例，以下表格指出了不同形式的链接是否与其同源：（原因里未申明不同的属性即说明其与例子里的原链接对应的属性相同）

|链接|结果|原因|
|-----|-----|-----|
|`http:// www.example.com /dir/page2.html`|	是|	同协议同域名同端口|
|`http:// www.example.com /dir2/other.html`|	是|	同协议同域名同端口|
|`http://user:pwd@ www.example.com/dir2/other.html`|	是|	同协议同域名同端口|
|`http://www.example.com: 81/dir/other.html`|	否|	端口不同|
|`https://www.example.com/dir/other.html`|	否|	协议不同端口不同|
|`http:// en.example.com/dir/other.html`|	否|	域名不同|
|`http:// example.com/dir/other.html`	|否|	域名不同（要求精确匹配）|
|`http:// v2.www.example.com/dir/other.html`|	否|	域名不同（要求精确匹配）|
|`http://www.example.com:80/dir/other.html`|	不确定	取决于浏览器的实现方式|

#### 是否允许跨域的判定

前文提到了同源策略的判定，然而同源策略在加强了安全的同时，对开发却是极大的不便利。因此开发者们又发明了很多办法来允许数据的跨域传输（常见的办法有JSONP、CORS)。当域名不同源的时候，由于跨域实现的存在，浏览器不能直接根据域名来判定跨域限制。那么浏览器具体又是如何实现判定的呢？看下面的例子。

待完成。。。

[原文地址](https://segmentfault.com/a/1190000003710973)
