---
layout: single
title: http security header
description: http security header
headline: http security header
categories: http,security
headline:
tags: [http,security]
comments: true
published: true
---

## google

| header                    | 值               | 描述                          |
| ------------------------- | ---------------- | ----------------------------- |
| content-security-policy   | *                | **很奇怪，没有找到csp的定义** |
| x-content-type-options    | *                |                               |
| x-xss-protection          | 1; mode=block    |                               |
| x-frame-options           | SAMEORIGIN       |                               |
| strict-transport-security | max-age=31536000 |                               |



## facebook

| header                    | 值                                                           | 描述                                  |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------- |
| content-security-policy   | default-src * data: blob:;script-src *.facebook.com *.fbcdn.net *.facebook.net *.google-analytics.com *.virtualearth.net *.google.com 127.0.0.1:* *.spotilocal.com:* 'unsafe-inline' 'unsafe-eval' *.atlassolutions.com blob: data: 'self';style-src data: blob: 'unsafe-inline' *;connect-src *.facebook.com facebook.com *.fbcdn.net *.facebook.net *.spotilocal.com:* wss://*.facebook.com:* https://fb.scanandcleanlocal.com:* *.atlassolutions.com attachment.fbsbx.com ws://localhost:* blob: *.cdninstagram.com 'self' chrome-extension://boadgeojelhgndaghljhdicfkmllpafd chrome-extension://dliochdbjfkdbacpmhlcpmleaejidimm; | 都包括了'unsafe-inline' 'unsafe-eval' |
| x-content-type-options    | nosniff                                                      |                                       |
| x-xss-protection          | 0                                                            |                                       |
| x-frame-options           | DENY                                                         |                                       |
| strict-transport-security | max-age=15552000; preload                                    |                                       |



## 币安

区分静态域名和动态域名，下面这些配置都加在动态域名下面

| header                    | 值                                                           | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| content-security-policy   | default-src 'self'; block-all-mixed-content; script-src 'self' 'sha256-/fCUycOSPg5W5rt7pgbdlufk2T9mZRRPEsV2mct1B/I=' 'sha256-5N4Pp5UCHKbIUxXXFe+KDYsfhzhQXoIzN80eQ+jF9P4=' 'unsafe-eval' 'nonce-786a9083fa0bcd2437cd51a2b9a864391f859eec' https://api.geetest.com https://ex.bnbstatic.com https://resource.binance.co.ug https://resource.binance.com https://resource.binance.sg https://static.geetest.com https://translate.google.com https://translate.googleapis.com https://www.binance.co https://www.google-analytics.com; style-src 'self' 'unsafe-inline' https://ex.bnbstatic.com https://resource.binance.co.ug https://resource.binance.com https://resource.binance.sg https://static.geetest.com https://translate.googleapis.com; font-src 'self' https://at.alicdn.com https://ex.bnbstatic.com https://fonts.gstatic.com https://resource.binance.co.ug https://resource.binance.com https://resource.binance.sg https://sensors.binance.cloud https://sensors.binance.com; connect-src 'self' https://ex.bnbstatic.com https://resource.binance.co.ug https://resource.binance.com https://resource.binance.sg https://sensors.binance.cloud https://sensors.binance.com https://sentry.io https://translate.googleapis.com wss://jpush.binance.im:5000 wss://stream.binance.cloud:9443 wss://stream.binance.com:9443 wss://stream2.binance.cloud:443 wss://stream2.binance.com:9443; img-src 'self' data: https://ex.bnbstatic.com https://resource.binance.co.ug https://resource.binance.com https://resource.binance.sg https://sensors.binance.cloud https://sensors.binance.com https://translate.google.com https://translate.googleapis.com https://www.binance.co https://www.google-analytics.com https://www.google.com https://www.gstatic.com; object-src 'none'; base-uri 'self' | **nonce的值是不变，这个有问题；**                            |
| x-content-type-options    | nosniff                                                      | 例如，我们即使给一个html文档指定Content-Type为"text/plain"，在IE8-中这个文档依然会被当做html来解析。利用浏览器的这个特性，攻击者甚至可以让原本应该解析为图片的请求被解析为JavaScript。通过下面这个响应头可以禁用浏览器的类型猜测行为： |
| x-frame-options           | SAMEORIGIN                                                   | SAMEORIGIN：不允许被本域以外的页面嵌入；                     |
| x-xss-protection          | 1; mode=block                                                | 启用XSS保护，并在检查到XSS攻击时，停止渲染页面（例如IE8中，检查到攻击时，整个页面会被一个#替换）； |
| strict-transport-security | max-age=31536000; includeSubdomains                          | 支持HSTS的浏览器遇到这个响应头，会把当前网站加入HSTS列表，然后在max-age指定的秒数内，当前网站所有请求都会被重定向为https。即使用户主动输入`http://`或者不输入协议部分，都将重定向到`https://`地址。 |
| x-dns-prefetch-control    | off                                                          | 避免浏览器自动的dns prefetch导致信息泄露                     |

## 测试方法

mkdir test将下面两个文件放进去

csp_test.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta http-equiv="Content-Security-Policy" content="script-src 'self' ">
    <title>CSP Test</title>
</head>
<body>
    <script src="https://unpkg.com/react@16/umd/react.development.js"></script>
    <script src="./test.js"></script>
    <script>
        window.onload=function(){
            alert('hi jack!')
        }
    </script>
</body>
</html>
```

test.js

```javascript
alert('hello from test.js')
```

在test目录下执行，在浏览器中localhost:8000就可以测试

```python
python -m SimpleHTTPServer 8000
```



# 参照

1. [使用DNS 预读取绕过Content Security Policy CSP](http://www.mottoin.com/article/web/91044.html)
2. [一些安全相关的HTTP响应头](https://imququ.com/post/web-security-and-response-header.html)
3. [mozilla关于http header的解释](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)
4. [Content Security Policy 入门教程](http://www.ruanyifeng.com/blog/2016/09/csp.html)
5. [Content Security Policy (CSP) 介绍](https://i.jakeyu.top/2018/08/26/Content-Security-Policy-CSP-%E4%BB%8B%E7%BB%8D/)
