title: 关于chrome报错net::ERR_INCOMPLETE_CHUNKED_ENCODING的解决方案
date: 2015-09-11 22:55:11
categories: bug
tags: [Chrome,JSON,Debug]
---

## 背景
今天遇到一个 bug ，这个 bug 很神奇，在 Chrome 下刷新项目页时会报错 `net::ERR_INCOMPLETE_CHUNKED_ENCODING` 而显示不出网页，在 Safari 和 Firefox 下正常。查了一下这个错误的网上的解决方案。众说纷纭比较多，由于这个 bug 着实查着改了很久..这里进行一个总结，来帮助可能被坑的后来人。
## bug 描述及解决办法
在某项目页中，后台传给我了一串比较长的 json 数据，我在处理这些数据，刷新页面时 Chrome意外报错 net::ERR_INCOMPLETE_CHUNKED_ENCODING。页面加载不了 dom 结构。在 Firefox 及 Safari 上加载正常。
在调试后发现，报错原因在于 json 数据在 Chrome 接收的过程中会被随机截断，不能加载完全的 json 数据。导致 json 结构报错。
![报错提示](http://qcyoung.qiniudn.com/qcyoung/关于chrome报错netERR_INCOMPLETE_CHUNKED_ENCODING的解决方案/chromeERR_INCOMPLETE_CHUNKED_ENCODING.png)

这里参考我的尝试步骤以及网上的部分可行的办法来推荐尝试：

1.关闭杀毒软件
2.关闭 Chrome 高级设置里的 “预提取资源，以便更快速的加载网页”
3.关闭 Chrome 扩展程序或停用代理类扩展
4.设置 header 报表头 content-length

	<?php
	header('Content-length: ' . strlen($output));
	?>
5.nginx fastcgi buffer 的设置
6.nginx 打开 gzip

