---
title: chrome-version-nginx-ciphers
date: 2019-02-01 23:37:00
tags:
- nginx
- Chrome
---

﻿

最近遇到开发的某域名升级为https后手机qq浏览器、魅族自带浏览器等低版本chrome内核浏览器访问不了在SSL诊断工具：https://www.ssllabs.com/ssltest/analyze.html    中诊断网站发现对比了一下该网站和另一网站的ssllabs的报告，区别地发现在Handshake Simulation中有标红的Chrome 49 / XP SP3 Server negotiated HTTP/2 with blacklisted suite。查找资料后定位到服务器nginx配置中的ssl_ciphers可能会导致部分浏览器访问不了，认为服务器网址使用了不安全的加密选项，于是开始尝试调整加密选项。

<!--more-->

​	参考[HTTP/2升级小记](http://starlight36.com/post/nginx-support-http2)和[分享一个 HTTPS A+ 的 nginx 配置](https://www.textarea.com/zhicheng/fenxiang-yige-https-a-di-nginx-peizhi-320/)以及其他文章。配置了

	>ssl_ciphers "ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA";

然后重启`nginx -s reload`重启nginx后可以访问了。 