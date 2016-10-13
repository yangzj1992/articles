---
title: iOS不兼容position:fixed属性
date: 2015-10-17 17:33:32
categories: 兼容性
tags: [兼容性,移动端,iOS]
---

### 问题情况
在移动端开发过程中如果在头部或底部设置 position:fixed 的元素，可能会在 ios8 以下的系统中出现以下问题，当用户进行输入时系统键盘激活，此类 fixed 的元素会出现位置浮动问题。类似如下图所示：
![position:fixed](http://qcyoung.qiniudn.com/qcyoung/iOS不兼容position-fixed属性/ios_position_fixed.jpeg)
### 解决办法
解决办法大致如下：
确保自己的页面已引入浏览器适应性 meta

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=0">
```

- CSS + JS:
原理在于这个问题发生在 overflow 属性为 scroll 时

``` css
.fixfixed{
   position: absolute;
   ...
}
```

``` javascript
var u = navigator.userAgent, app = navigator.appVersion;
var isIOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/); //ios终端

if (isIOS) {
    $(document).on("focus", "input", function () {
        $(yourselector).addClass("fixfixed");
    }).on("focusout", "input", function () {
        $(yourselector).removeClass("fixfixed");
    });
}
```

或使用:iscroll.js 框架

### 额外参考
另外 iOS 系统中目前针对 fixed 的元素还会出现闪烁的现象，具体表现为呼出系统键盘，元素在一定时间后才回归原位。

具体的移动端 fixed 兼容性情况原因在阿里的支付宝开发经验中有详细讲过[无线Web开发经验谈](http://am-team.github.io/amg/dev-exp-doc.html)

- ios 系统: 在 ios5 之后，iOS 才正式开始支持 fixed 的布局，在 ios5 之前，苹果处于性能上的考虑，并没有实现，因此在使用 fixed 的时候，需要注意你所做的项目对 ios 的版本最低支持的版本，不过即使 ios5 之后，开始支持 fixed 属性，在实际使用中，还是有很多小坑在，国外专门有个网页再说 ios 的 fixed 的问题。提供以下地址，可供参考：[iOS-fixed issues](http://remysharp.com/2012/05/24/issues-with-position-fixed-scrolling-on-ios/)。比较安全的做法是，在固定的布局里面，尽可能保持里面的结构简单，不要出现过于复杂的布局，一般 app 的头部和尾部可以使用 fixed 属性。
- Android 系统 Android 系统在 2.1 之后，就已经开始支持 fixed，不过由于各个厂商对于 fixed 的实现不同，2.1 和2.2 对于 fixed 的支持不是很好，在滚动的时候会出现闪动，消失、位移等各种渲染问题。2.3 之后的版本，fixed 的问题相对少一些，不过在个别厂商的手机上也会出现各种渲染问题。从 4.x 开始，fixed 的表现比较好。因此如果在 Android 上需要 fixed 的效果，需要综合评判其效果。