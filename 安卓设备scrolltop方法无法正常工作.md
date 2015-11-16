title: 安卓设备scrolltop方法无法正常执行
date: 2015-10-09 14:37:26
categories: 兼容性
tags: [兼容性,移动端,Android]
---
[github地址](https://github.com/yangzj1992/articles/blob/master/安卓设备scrolltop方法无法正常工作.md)

###问题情况
今天遇到一个问题，在Android设备下scrollTop()方法无法正常执行。
###解决办法
相关参考了一些解决办法之后，了解问题如下:
解决办法大致如下：

- CSS+JS:
原理在于这个问题发生在overflow属性为scroll时

``` css
.androidFix {
    overflow:hidden !important;
    overflow-y:hidden !important;
    overflow-x:hidden !important;
}

```

``` javascript
$(yourSelector).addClass("androidFix").scrollTop(0).removeClass("androidFix");
```
<br>
###相关参考
一篇不错的各浏览器内核使用scrollTop方法介绍：[如何正确的获取scrollTop/scrollLeft的值](http://bbs.csdn.net/topics/340198399)


