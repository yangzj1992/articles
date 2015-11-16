title: 关于chrome报错net::ERR_INCOMPLETE_CHUNKED_ENCODING的解决方案
date: 2015-09-11 22:55:11
categories: bug
tags: [Chrome,JSON,Debug]
---
[github地址](https://github.com/yangzj1992/articles/blob/master/关于chrome报错netERR_INCOMPLETE_CHUNKED_ENCODING的解决方案.md)

##背景
今天遇到一个bug问题，这个bug问题很神奇，在chrome下刷新项目页时会报错`net::ERR_INCOMPLETE_CHUNKED_ENCODING`而显示不出网页，在Safari和Firefox下正常。查了一下这个错误的网上的解决方案。众说纷纭比较多，由于这个BUG着实查着改了很久..这里进行一个总结，来帮助可能被坑的后来人。
##BUG描述及解决办法
在某项目页中，后台传给我了一串比较长的json数据，我在处理这些数据，刷新页面时chrome意外报错net::ERR_INCOMPLETE_CHUNKED_ENCODING。页面加载不了dom结构。在firefox及Safari上加载正常。
在调试后发现，报错原因在于json数据在chrome接收的过程中会被随机截断，不能加载完全的json数据。导致json结构报错。
![报错提示](http://7bv937.com1.z0.glb.clouddn.com/qcyoung/关于chrome报错netERR_INCOMPLETE_CHUNKED_ENCODING的解决方案/chromeERR_INCOMPLETE_CHUNKED_ENCODING.png)

这里参考我的尝试步骤以及网上的部分可行的办法来推荐尝试：

1.关闭杀毒软件。
2.关闭chrome 高级设置里的 “预提取资源，以便更快速的加载网页”
3.关闭chrome扩展程序或停用代理类扩展
4.设置header报表头content-length

	<?php
	header('Content-length: ' . strlen($output));
	?>
5.nginx fastcgi buffer的设置.
6.nginx 打开gzip

