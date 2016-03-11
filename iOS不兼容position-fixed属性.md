title: iOS不兼容position:fixed属性
date: 2015-10-17 17:33:32
categories: 兼容性
tags: [兼容性,移动端,iOS]
---

### 问题情况
在移动端开发过程中如果在头部或底部设置position:fixed的元素，可能会在ios8以下的系统中出现以下问题，当用户进行输入时系统键盘激活，此类fixed的元素会出现位置浮动问题。类似如下图所示：
![position:fixed](http://qcyoung.qiniudn.com/qcyoung/iOS不兼容position-fixed属性/ios_position_fixed.jpeg)
### 解决办法
解决办法大致如下：
确保自己的页面已引入浏览器适应性meta

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=0">
```

- CSS+JS:
原理在于这个问题发生在overflow属性为scroll时

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

或使用:iscroll.js框架

### 额外参考
另外iOS系统中目前针对fixed的元素还会出现闪烁的现象，具体表现为呼出系统键盘，元素在一定时间后才回归原位。

具体的移动端fixed兼容性情况原因在阿里的支付宝开发经验中有详细讲过[无线Web开发经验谈](http://am-team.github.io/amg/dev-exp-doc.html)

- ios系统: 在ios5之后，iOS才正式开始支持fixed的布局，在ios5之前，苹果处于性能上的考虑，并没有实现，因此在使用fixed的时候，需要注意你所做的项目对ios的版本最低支持的版本，不过即使ios5之后，开始支持fixed属性，在实际使用中，还是有很多小坑在，国外专门有个网页再说ios的fixed的问题。提供以下地址，可供参考：[iOS-fixed issues](http://remysharp.com/2012/05/24/issues-with-position-fixed-scrolling-on-ios/)。比较安全的做法是，在固定的布局里面，尽可能保持里面的结构简单，不要出现过于复杂的布局，一般app的头部和尾部可以使用fixed属性。
- android系统 android系统在2.1之后，就已经开始支持fixed，不过由于各个厂商对于fixed的实现不同，2.1和2.2对于fixed的支持不是很好，在滚动的时候会出现闪动，消失、位移等各种渲染问题。2.3之后的版本，fixed的问题相对少一些，不过在个别厂商的手机上也会出现各种渲染问题。从4.x开始，fixed的表现比较好。因此如果在android上需要fixed的效果，需要综合评判其效果。