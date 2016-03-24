title: npm left-pad模块删除事件始末梳理及看法
date: 2016-03-24 11:37:26
categories: npm
tags: [npm,模块化]
---
## 事件起因
事件原因是在23日凌晨，一个开发者因为对NPM不满，unpublish了自己的所有模块。其中包括被广泛使用的`left-pad`，导致`Babel`、`ReactNative`、`Ember`等大量工具构建失败。

[left-pad](https://www.npmjs.com/package/left-pad) 是一个依赖度非常高的仓库，根据 NPM 的统计显示，left-pad 在23日时统计显示昨日的下载量是 10 万，上周的下载量为 57 万，上个月下载量达到了 255 万。

之后`left-pad`作者Azer在medium发布了一篇博文来阐述他删除自己模块的原因
[I’ve Just Liberated My Modules](https://medium.com/@azerbike/i-ve-just-liberated-my-modules-9045c06be67c#.7unu9xvjk)

这里参考justjavac提供的[翻译文章](http://mp.weixin.qq.com/s?__biz=MzI0NTAyNjE0NQ==&mid=427175657&idx=1&sn=57c0cda917797198df18552f7d8f4eca&scene=0#wechat_redirect)如下：
> 作者在《I've Just Liberated My Modules》文章中写道：

>几个星期前有位专利律师给我发了一封电子邮件，要求我取消发布 NPM 上的 “KIK” 模块。我的回答是“不”，于是他回复我说：“I don’t wanna be dick about it（这句就不翻译了，你只需要知道 dick 是什么意思就够了），但 “KIK” 是我们的注册品牌，并且我们的律师遍布世界各地。”

>当我开始编写 kik 时，并不知道有同名的公司。而我也不希望因为这个公司而被迫改变项目的名字。在遭到了我的拒绝后，他们联系了 NPM 的技术支持，为了强调他们的律师权力，每一个电子邮件都抄送给了我。在未经我允许的情况下，@izs 更改了此模块的所有权。

>鉴于此我才意识到，NPM 是某个人的私有地盘，他比其他人有更多的控制权，但是我是做开源的，因为权力属于人民。（Power To The People 是约翰·列侬的同名歌曲）

>概述一下就是; NPM 不再是我分享开源工作的地方，所以，我取消了曾经发布的所有模块)。（一共取消了 273 个）

>这不是一个下意识的行为。我喜欢开源，相信开源社区将最终创造一个真正自由的 NPM。

>如果你的项目因此而构建失败，我向你道歉。你可以在仓库（azer/dependency）指出你的依赖，或者如果你自愿参加我的 Github 上的任何模块，我会高兴地转移所有权。

>干杯，再见。

## npm模块依赖的生态环境
之后有人对[left-pad源码](https://github.com/azer/left-pad)进行查看惊讶的发现其源码仅有11行：
```
module.exports = leftpad;
function leftpad (str, len, ch) {
  str = String(str);
  var i = -1;
  if (!ch && ch !== 0) ch = ' ';
  len = len - str.length;
  while (++i < len) {
    str = ch + str;
  }
  return str;
}
```
于是一部分的人产生了疑问：**为什么这么短的代码，也能成为众多库比如 RN 的依赖？**

之后Haney的一篇博文对类似这样的模块进行了更多的搜索，发现了相当多类似`left-pad`这样的模块。提出了疑问：我们都忘记了怎样去写程序了吗？原文在此[NPM & left-pad: Have We Forgotten How To Program?](http://www.haneycodes.net/npm-left-pad-have-we-forgotten-how-to-program/)
这里简单翻译一下他列举的事实给大家：
>好的开发者们，接下来是一个比较严肃的讨论。正如你们可能已经知道的那样，这一周里`babel`,`react`以及类似一大堆高依赖的模块在npm无法被成功构建。他们宕机的原因是相当惊人的。

>因为他们共同依赖了一个简单的NPM包`left-pad`，它的目的是实现左填充一个字符串，他的源码也总共只有11行。
令人担心的是这里有这么多包选择去依赖一个这样简单的函数，而不是采取几分钟去写一个这样的基本功能。

>由于这次灾难，我开始调查整个NPM的生态系统。这里果然有一些类似的事情：

>- 有一个叫[IsArray](https://www.npmjs.com/package/isarray)的包，每天达到88万的下载量，而它的源代码仅有一行：
>```
return toString.call(arr) == '[object Array]';
>```
> - 有一个判断是否为正数的函数[is-positive-integer ](https://www.npmjs.com/package/is-positive-integer)，源代码只有4行，截至昨日仍然有3个项目依赖使用。后来作者重构了它依赖才变为0
> - 一个全新安装的babel包，包含了41000个文件
> - 一个空白的jspm/npm-based 应用模板起始就包含了28000个文件
>**这些都让我感到相当惊讶。难道我们都忘记了怎样编程了吗？**

## 看法
这次的危机算是一个潜在问题的大爆发,的确现在模块依赖是大势所趋，如果大量的基础函数可以通过外部依赖解决，那么自己的程序的代码行数就可以减少很多，程序员便可以专注于写一些新的不同的东西。这种思想肯定是没有问题的。
而类似Haney所列举的这些简单依赖包能够产生的原因正是因为标准函数库存在不足。让任何人都必须写一个左填充的字符串函数是十分荒谬的。
另外其实npm对于解决这种包名纠纷也是有很明确的规则[npm-disputes](https://docs.npmjs.com/misc/disputes)。我觉得其实NPM官方的处理也不能说有太大的问题。

类似这样的事件我认为最需要解决的两件事是
- 语言能够提供一个足够健壮和充分的标准库函数，这是解决这样的问题的一个重要关键。
- NPM应该尽快解决类似这种危机的处理，出现这种问题的原因也一部分在于NPM目前没有应对这种问题的机制，尝试修改unpublish包的规则、依赖项目如果缺失尝试寻找更多的替代品，防范其他人伪造同名库等.

## 后续

npm今日在[官方宣布](http://blog.npmjs.org/post/141577284765/kik-left-pad-and-npm)，由于昨天的事件，今后将严格限制 unpublish 命令。并且，不再允许自由认领已经废弃的模块，必须向 npm 官方提出申请。





