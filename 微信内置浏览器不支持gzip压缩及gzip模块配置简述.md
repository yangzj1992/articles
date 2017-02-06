---
title: 微信内置浏览器不支持 gzip 压缩方式及 gzip 模块配置简述
date: 2015-11-11 17:50:02
categories: 微信X5浏览器
tags: [微信浏览器,AJAX,gzip,性能优化,HTTP]
---

### 情况
今天在做微信项目用 ajax 传值时发现了一些异常的现象。使用微信版本：6.3.7

![异常 ajax 列表](http://qcyoung.qiniudn.com/qcyoung/微信内置浏览器不支持gzip压缩方式及gzip模块配置简述/yichang1.png)

1 个 ajax 请求请求了 120kb?!打开此请求详情看了一下发现了一个关键点。

![微信请求头](http://qcyoung.qiniudn.com/qcyoung/微信内置浏览器不支持gzip压缩方式及gzip模块配置简述/weixin_requestheader.png)

这个微信请求，在请求头中没有接受编码头(`Accept-Encoding`),而在正常的电脑 Chrome 中 `Accept-Encoding` 是正常的。

![Chrome 请求头](http://qcyoung.qiniudn.com/qcyoung/微信内置浏览器不支持gzip压缩方式及gzip模块配置简述/chrome_requestheader.png)

难道是微信浏览器不支持 gzip 的压缩方式？这里我抓包做了实验，首先我在电脑端 Chrome 下把 `Accept-Encoding` 中的 gzip 值干掉进行请求。果然，请求的大小一下子飙升到微信上请求的相应大小

![请求列表](http://qcyoung.qiniudn.com/qcyoung/微信内置浏览器不支持gzip压缩方式及gzip模块配置简述/chrome_kill_gzip_list.png)

此请求报表头如下：

![清理头](http://qcyoung.qiniudn.com/qcyoung/微信内置浏览器不支持gzip压缩方式及gzip模块配置简述/chrome_kill_gzip_header.png)

### w3标准
由于在 w3.org 中关于 [XMLHttpRequest](http://www.w3.org/TR/XMLHttpRequest/) 的描述中指出

> 4.6.2 The setRequestHeader() method
> 
> Terminate these steps if header is a case-insensitive match for one of the following headers:
> - Accept-Charset
> - Accept-Encoding
> - Access-Control-Request-Headers
> - Access-Control-Request-Method
> - Connection
> - Content-Length
> - Cookie
> - Cookie2
> - Date
> - DNT
> - Expect
> - Host
> - Keep-Alive
> - Origin
> - Referer
> - TE
> - Trailer
> - Transfer-Encoding
> - Upgrade
> - User-Agent
> - Via

> The above headers are controlled by the user agent to let it control those aspects of transport. This guarantees data integrity to some extent. Header names starting with Sec- are not allowed to be set to allow new headers to be minted that are guaranteed not to come from XMLHttpRequest.

上面的消息头被认为是应当由浏览器控制，而不能用 XMLHttpRequest 对象来修改，即不能通过 JavaScript 修改。但是这也只是 W3 建议的标准而已，至于浏览器遵不遵循标准，那就得看开发人员了。

这里为了测试，可以试着写一个 PHP,回显 `User-Agent`：

``` php
<?php
echo $_SERVER['HTTP_USER_AGENT'];
?>
```

然后再次发送 ajax 请求，并在发送之前用 XMLHttpRequest 对象的 `setRequestHeader` 方法修改 `User-Agent`：

``` js
$.ajax({
    type: "GET",
    url: "ua.php",
    success: function(data) {
        alert(data);
    },
    beforeSend: function(xhr) {
        xhr.setRequestHeader("User-Agent", "uTorrent");
    }
});
```

通过测试最后得到结论：IE 还是跟往常一样无视标准的存在，可以用 JavaScript 在 ajax 请求中设置 `User-Agent`，而 FireFox 和 Chrome 都无法修改 `User-Agent`。

### 其余网站验证
之后我测试了一米鲜和京东的相关页面，也发现了类似的情况。这里拿 JD 举例。

在微信中未压缩请求，大小 17.53kb。
![jd](http://qcyoung.qiniudn.com/qcyoung/微信内置浏览器不支持gzip压缩方式及gzip模块配置简述/weixinjd.png)

请求头如下

![jd](http://qcyoung.qiniudn.com/qcyoung/微信内置浏览器不支持gzip压缩方式及gzip模块配置简述/chrome_jd_header副本.png)

在手机 Chrome 下启用 gzip 大小为 5.2kb:

![jd](http://qcyoung.qiniudn.com/qcyoung/微信内置浏览器不支持gzip压缩方式及gzip模块配置简述/mobile_chrome_jd_list.png)

![jd](http://qcyoung.qiniudn.com/qcyoung/微信内置浏览器不支持gzip压缩方式及gzip模块配置简述/mobile_chrome_jd_header.png)


### gzip 配置说明
另外在这个过程中还核对了 nginx 的 gzip 模块配置，这里顺便一路贴出来吧：

``` bash
> Gzip 模块的各项配置

> gzip on|off;
> # 默认值: gzip off 
> # 开启或者关闭gzip模块
>  
> gzip_static on|off;
> # nginx对于静态文件的处理模块
> # 该模块可以读取预先压缩的gz文件，这样可以减少每次请求进行gzip压缩的CPU资源消耗。该模块启用后，
nginx首先检查是否存在请求静态文件的gz结尾的文件，如果有则直接返回该gz文件内容。为了要兼容不支持
gzip的浏览器，启> 用gzip_static模块就必须同时保留原始静态文件和gz文件。这样的话，在有大量静态
文件的情况下，将会大大增加磁盘空间。我们可以利用nginx的反向代理功能实现只保留gz文件。
> # 可以google"nginx gzip_static"了解更多
>  
> gzip_comp_level 4;
> # 默认值：1（建议选择为4）
> # gzip压缩比/压缩级别，压缩级别 1-9，级别越高压缩率越大，当然压缩时间也就越长（传输快但比较消耗cpu）。
>  
> gzip_buffers 4 16k;
> # 默认值: gzip_buffers 4 4k/8k 
> # 设置系统获取几个单位的缓存用于存储gzip的压缩结果数据流。 例如 4 4k 代表以4k为单位，按照原
始数据大小以4k为单位的4倍申请内存。 4 8k 代表以8k为单位，按照原始数据大小以8k为单位的4倍申请内存。
> # 如果没有设置，默认值是申请跟原始数据相同大小的内存空间去存储gzip压缩结果。
>  
> gzip_types mime-type [mime-type ...];
> # 默认值: gzip_types text/html （默认不对js/css文件进行压缩）
> # 压缩类型，匹配MIME类型进行压缩
> # 不能用通配符 text/*
> # （无论是否指定）text/html默认已经压缩 
> # 设置哪压缩种文本文件可参考 conf/mime.types
>  
> gzip_min_length  1k;
> # 默认值: 0 ，不管页面多大都压缩
> # 设置允许压缩的页面最小字节数，页面字节数从header头中的Content-Length中进行获取。
> # 建议设置成大于1k的字节数，小于1k可能会越压越大。 即: gzip_min_length 1024
>  
> gzip_http_version 1.0|1.1;
> # 默认值: gzip_http_version 1.1（就是说对HTTP/1.1协议的请求才会进行gzip压缩）
> # 识别http的协议版本。由于早期的一些浏览器或者http客户端，可能不支持gzip自解压，用户就会看到乱码，
所以做一些判断还是有必要的。 
> # 注：99.99%的浏览器基本上都支持gzip解压了，所以可以不用设这个值,保持系统默认即可。
> # 假设我们使用的是默认值1.1，如果我们使用了proxy_pass进行反向代理，那么nginx和后端的upstream server
之间是用HTTP/1.0协议通信的，如果我们使用nginx通过反向代理做Cache > Server，而且前端的nginx没有开启gzip，
同时，我们后端的nginx上没有设置gzip_http_version为1.0，那么Cache的url将不会进行gzip压缩
>  
> gzip_proxied [off|expired|no-cache|no-store|private|no_last_modified|no_etag|auth|any] ...;
> # 默认值：off
> # Nginx作为反向代理的时候启用，开启或者关闭后端服务器返回的结果，匹配的前提是后端服务器必须要返回
包含"Via"的 header头。
> # off - 关闭所有的代理结果数据的压缩
> # expired - 启用压缩，如果header头中包含 "Expires" 头信息
> # no-cache - 启用压缩，如果header头中包含 "Cache-Control:no-cache" 头信息
> # no-store - 启用压缩，如果header头中包含 "Cache-Control:no-store" 头信息
> # private - 启用压缩，如果header头中包含 "Cache-Control:private" 头信息
> # no_last_modified - 启用压缩,如果header头中不包含 "Last-Modified" 头信息
> # no_etag - 启用压缩 ,如果header头中不包含 "ETag" 头信息
> # auth - 启用压缩 , 如果header头中包含 "Authorization" 头信息
> # any - 无条件启用压缩
>  
> gzip_vary on;
> # 和http头有关系，加个vary头，给代理服务器用的，有的浏览器支持压缩，有的不支持，所以避免浪费不支持
的也压缩，
所以根据客户端的HTTP头来判断，是否需要压缩
>  
> gzip_disable "MSIE [1-6].";
> # 禁用IE6的gzip压缩，又是因为杯具的IE6。当然，IE6目前依然广泛的存在，所以这里你也可以设置为“MSIE [1-5].”
> # IE6的某些版本对gzip的压缩支持很不好，会造成页面的假死，为了确保其它的IE6版本不出问题，所以建议加上
gzip_disable的设置
```
