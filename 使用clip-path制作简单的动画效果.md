title: 使用clip-path制作简单的动画效果
date: 2015-11-18 16:48:18
categories: web动画
tags: [CSS,web动画]
---

##介绍
今年4月左右有一个比较火的网站species-in-pieces在前端圈里比较出名，[点击这里查看](http://species-in-pieces.com/#)
![species-in-pieces效果](http://qcyoung.qiniudn.com/qcyoung/使用clip-path制作简单的动画效果/poster-detail-2.png)
它的原理就是运用了clip-path来进行实现。
在《species in pieces》中每个动物的组成节点如下所示

```
<div class="wrap left-to-right">
    <div class="shard-wrap">
        <div class="shard"></div>
    </div>
    <div class="shard-wrap">
        <div class="shard"></div>
    </div>
    .
    .
    省略若干30个
    .
    .
    <div class="shard-wrap">
        <div class="shard"></div>
    </div>
    <div class="shard-wrap">
        <div class="shard"></div>
    </div>
</div>
```

对应的css文件如下：

```
.shard-wrap { width: 100%; height: 100%; position: absolute; transition: .5s; z-index: 2; }
/* crow 乌鸦的图形描述 */
.crow .shard-wrap:nth-child(1) .shard {
  -webkit-clip-path: polygon(20% 50%,25% 52.4%,11.5% 54.5%);
  background-color: #2C323D
}
.crow .shard-wrap:nth-child(2) .shard {
  -webkit-clip-path: polygon(14.7% 47.5%,35.2% 50.2%,25% 52.5%);
  background-color: #63676F
}
.crow .shard-wrap:nth-child(3) .shard {
  -webkit-clip-path: polygon(22.9% 44.5%,35.2% 50.2%,25% 48.9%);
  background-color: #0F1622
}
/*
.
.
省略
.
.
*/
.crow .shard-wrap:nth-child(29) .shard {
  -webkit-clip-path: polygon(61.7% 44.7%,64.4% 44%,65.1% 36.2%);
  background-color: #0f1622
}
.crow .shard-wrap:nth-child(30) .shard {
  -webkit-clip-path: polygon(78.5% 21.5%,76.3% 23.7%,74.6% 22.5%);
  background-color: #0f1622
}
```

这样利用30个三角形，每个三角形代表一对.shard-wrap>.shard节点, 来进行拼接展现，所有的图形描述由:nth-child伪类选择器来控制形状。

##举个栗子
这里做了一个简单的demo,鼠标滑到界面上github小猫就会发生变化（仅支持webkit内核浏览器）

<iframe src="/project/clip-path.html" width="100%" height="550px" id="framedemo" frameborder="0" scrolling="no"></iframe>

##拓展

此外clip-path属性也可以实现3D模型渲染的效果，如下图所示：

![3D动画效果1](http://qcyoung.qiniudn.com/qcyoung/使用clip-path制作简单的动画效果/snapshot.gif)

![3D动画效果2](http://qcyoung.qiniudn.com/qcyoung/使用clip-path制作简单的动画效果/6252205cgw1eqmyg50fsjg208e0a44mf.gif)

关于更详细的描述，这里有一篇很详细的博文可以参考[网易萝卜的博文](http://leeluolee.github.io/2015/04/01/render-3d-use-clip-path/)

<script>
  var width = $("#framedemo").width();
  $("#framedemo").height(width*0.77)
</script>