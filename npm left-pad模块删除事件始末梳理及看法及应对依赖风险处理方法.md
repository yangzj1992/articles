title: npm left-pad模块删除事件始末梳理及看法及应对依赖风险处理方法
date: 2016-03-24 11:37:26
categories: npm
tags: [npm,模块化]
---
## 事件起因
事件原因是在23日凌晨，一个开发者因为对 NPM 不满，unpublish 了自己的所有模块。其中包括被广泛使用的`left-pad`，导致`Babel`、`ReactNative`、`Ember`等大量工具构建失败。

[left-pad](https://www.npmjs.com/package/left-pad) 是一个依赖度非常高的仓库，根据 NPM 的统计显示，`left-pad`在23日统计显示昨日的下载量是 10 万，上周的下载量为 57 万，上个月下载量达到了 255 万。

之后`left-pad`作者 Azer 在 Medium 发布了一篇文章来阐述他删除自己模块的原因
[I’ve Just Liberated My Modules](https://medium.com/@azerbike/i-ve-just-liberated-my-modules-9045c06be67c#.7unu9xvjk)

这里参考 justjavac 提供的[翻译文章](http://mp.weixin.qq.com/s?__biz=MzI0NTAyNjE0NQ==&mid=427175657&idx=1&sn=57c0cda917797198df18552f7d8f4eca&scene=0#wechat_redirect)如下：
> 作者在《I've Just Liberated My Modules》文章中写道：

>几个星期前有位专利律师给我发了一封电子邮件，要求我取消发布 NPM 上的 “KIK” 模块。我的回答是“不”，于是他回复我说：“I don’t wanna be dick about it（这句就不翻译了，你只需要知道 dick 是什么意思就够了），但 “KIK” 是我们的注册品牌，并且我们的律师遍布世界各地。”

>当我开始编写 kik 时，并不知道有同名的公司。而我也不希望因为这个公司而被迫改变项目的名字。在遭到了我的拒绝后，他们联系了 NPM 的技术支持，为了强调他们的律师权力，每一个电子邮件都抄送给了我。在未经我允许的情况下，@izs 更改了此模块的所有权。

>鉴于此我才意识到，NPM 是某个人的私有地盘，他比其他人有更多的控制权，但是我是做开源的，因为权力属于人民。（Power To The People 是约翰·列侬的同名歌曲）

>概述一下就是; NPM 不再是我分享开源工作的地方，所以，我取消了曾经发布的所有模块。（一共取消了 273 个）

>这不是一个下意识的行为。我喜欢开源，相信开源社区将最终创造一个真正自由的 NPM。

>如果你的项目因此而构建失败，我向你道歉。你可以在仓库（azer/dependency）指出你的依赖，或者如果你自愿参加我的 Github 上的任何模块，我会高兴地转移所有权。

>干杯，再见。

## npm 模块依赖的生态环境
之后有人对[left-pad源码](https://github.com/azer/left-pad)进行查看，惊讶的发现其源码仅有11行：
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

之后 Haney 的一篇博文对类似这样的模块进行了更多的搜索，发现了相当多类似 `left-pad` 这样的模块。提出了疑问：我们都忘记了怎样去写程序了吗？原文在此 [NPM & left-pad: Have We Forgotten How To Program?](http://www.haneycodes.net/npm-left-pad-have-we-forgotten-how-to-program/) 
这里简单翻译一下他列举的事实给大家：
>好的开发者们，接下来是一个比较严肃的讨论。正如你们可能已经知道的那样，这一周里`babel`,`react`以及类似一大堆高依赖的模块在 npm 无法被成功构建。他们宕机的原因是相当惊人的。

>因为他们共同依赖了一个简单的 NPM 包 `left-pad`，它的目的是实现左填充一个字符串，他的源码也总共只有 11 行。
令人担心的是这里有这么多包选择去依赖一个这样简单的函数，而不是采取几分钟去写一个这样的基本功能。

>由于这次灾难，我开始调查整个 NPM 的生态系统。这里果然有一些类似的事情：

>- 有一个叫[IsArray](https://www.npmjs.com/package/isarray)的包，每天达到 88 万的下载量，而它的源代码仅有一行：
>```
return toString.call(arr) == '[object Array]';
>```
> - 有一个判断是否为正数的函数[is-positive-integer ](https://www.npmjs.com/package/is-positive-integer)，源代码只有 4 行，截至昨日仍然有 3 个项目依赖使用。后来作者重构了它依赖才变为 0
> - 一个全新安装的babel包，包含了 41000 个文件
> - 一个空白的jspm/npm-based 应用模板起始就包含了 28000 个文件
>**这些都让我感到相当惊讶。难道我们都忘记了怎样编程了吗？**

## 看法
这次的危机算是一个潜在问题的大爆发,的确现在模块依赖是大势所趋，如果大量的基础函数可以通过外部依赖解决，那么自己的程序的代码行数就可以减少很多，程序员便可以专注于写一些新的不同的东西。这种思想肯定是没有问题的。
而类似 Haney 所列举的这些简单依赖包能够产生的原因正是因为标准函数库存在不足。让任何人都必须在最开始得亲手写一个左填充字符串函数也是十分荒谬的。
另外，其实 npm 对于解决这种包名纠纷也是有很明确的规则 [npm-disputes](https://docs.npmjs.com/misc/disputes)。我觉得其实 NPM 官方的处理也不能说有太大的问题。

类似这样的事件我认为最需要解决的两件事是:
- 语言应该能够提供一个足够健壮并且充分的标准库函数，这是解决这样的问题的一个重要关键。
- 出现这种问题的原因也一部分在于NPM目前没有应对这种问题的机制，NPM 应该尽快补充类似这种危机的处理手段，如尝试修改 unpublish 包的规则、依赖项目如果缺失尝试寻找更多的替代品，防范其他人伪造同名库等。

## 后续

npm 今日在[官方宣布](http://blog.npmjs.org/post/141577284765/kik-left-pad-and-npm)，由于昨天的事件，今后将严格限制 unpublish 命令。并且，不再允许自由认领已经废弃的模块，必须向 npm 官方提出申请。


## 经验教训([参考](https://segmentfault.com/a/1190000004700432))

### 不要太依赖他人
如果我们想构建稳定的应用，非常重要的一点就是“不要将自己的全部身家都拴在别人的裤腰带”上，你永远不知道那个别人什么时候会“扑街”。其实在代码实现中，我们一直都被教导要小心强依赖，依赖过强会导致我们不够灵动。“牵一发而动全身”是很可怕的一件事。

具体到 Node 模块的依赖这件事上，太依赖他人会有些什么“可怕”的事情？

- 模块全部要从 NPM 的 registry 拉取然后安装，每天的持续集成越来越慢、越来越慢。
- 像 left-pad 这个模块一样，你依赖的模块被作者怒删了，应用挂掉。
- 你在 package.json 里面指定依赖时使用了 ~a.b.c 这种表示法（意思是小版本我都要），这表示每次 npm install 时其实获取到的模块依赖很可能是和你测试后发布的版本不一致的（模块发布了新的小版本），心里慌慌的。
- 你依赖模块的作者是个傻逼，不小心将不兼容的改动当作小版本发布了一个新版。npm install 或者 npm update 以后你依赖了这个新版，应用挂掉了。
*注：对 3、4 有疑问的可以查看：[深入 Node 模块的安装和发布](https://segmentfault.com/a/1190000004221514)*

### 破解强依赖

先来列出我们需要些什么：

- 在发布前“冻结”依赖模块的版本号。这让我们对安装的依赖有信心，依赖模块的版本都是我们验证、测试过的。
- 在发布前“打包”依赖模块到自己项目。这让我们可以坦然面对我们依赖的某个模块“没有了”这样的囧境。

### 冻结依赖模块
冻结依赖模块的版本号最简单的办法就是直接在`package.json`里面写死版本号，但是这解决不了深度依赖的问题。我们来看个例子。

假设有下面这样的依赖：
```javascript
A@0.1.0
|—B@0.0.1
   |—C@0.0.1
```
A 模块依赖了 B 模块，B 模块又依赖了 C 模块。我们可以将 B 模块的依赖写死成 `0.0.1` 版本，但是如果 B 模块对 C 模块的依赖写的是 `C: ~0.0.1`，会怎样？

这时候 C 模块更新到了 0.0.2 版本，虽然我们安装的 B 模块是 `B@0.0.1`，但是安装的 C 模块却是 `C@0.0.2`。如果不巧这个`C@0.0.2` 刚好有 bug，那我们的模块很有可能就不能正常工作了。

实际上，NPM 提供了一个叫做 `npm shrinkwrap` 的命令来我们解决这个问题：
```
NAME
  npm-shrinkwrap -- Lock down dependency versions

SYNOPSIS
  npm shrinkwrap

DESCRIPTION
  This  command  locks down the versions of a package's dependencies so that you can control exactly which versions of each  dependency  will be used when your package is installed.
```
这条命令会根据目前我们 `node_modules` 目录下的模块来生成一份“冻结”住的模块依赖（npm-shrinkwrap.json）。

还是上面的例子，我们在模块 A 的根目录执行 `npm shrinkwrap` 后，生成的 `npm-shrinkwrap.json` 文件内容大概是下面这样：
```
{
    "name": "A",
    "dependencies": {
        "B": {
            "version": "0.0.1",
            "resolved": "http://registry.npmjs.com/B-0.0.1.tgz",
            "dependencies": {
                "C": {  
                    "version": "0.0.1",
                    "resolved": "http://registry.npmjs.com/C-0.0.1.tgz"
                }
            }
        }
    }
}
```
然后，当我们执行 `npm install` 时，依赖查找的“来源”不再是 `package.json`，而是我们生成的 `npm-shrinkwrap.json`，再也不会突然装上什么 `C@0.0.2` 了，依赖里面的模块版本都是我们验证、测试后的版本，让人安心。

注：`npm shrinkwrap` 默认只会生成 `dependencies` 的依赖，不会生成 `devDependencies` 的依赖，如果你真的需要，可以加 `--dev` 参数。
### 打包依赖模块
我们解决了依赖模块版本号的问题，但是每次安装时其实还是会去 NPM 的 registry 获取模块的 tgz 包然后进行安装。我们需要将这些依赖都打包进我们的项目。这可能会带来一些问题（比如：项目体积的增大），但是好处也是显而易见的。

上面生成的 `npm-shrinkwrap.json` 里面有个 `resolved` 字段，表示模块所在的位置，实际上这个字段完全可以写一个文件路径。所以，我们可以递归的遍历 `npm-shrinkwrap.json` 文件，将所有的 tgz 包先下载到我们项目的某个目录，然后改写 `resolved` 字段为对应的文件路径。这样的功能有开发者已经实现了，我们可以直接[享用](https://github.com/JamieMason/shrinkpack)

还是上面的例子：
```javascript
A@0.1.0
|—B@0.0.1
   |—C@0.0.1
```
执行 `shrinkpack` 后，会生成下面的打包目录：
```
node_shrinkpack
 - B-0.0.1.tgz
 - C-0.0.1.tgz
 ```
和 node-shrinkwrap.json 文件：
```
{
    "name": "A",
    "dependencies": {
        "B": {
            "version": "0.0.1",
            "resolved": "./node_shrinkpack/B-0.0.1.tgz",
            "dependencies": {
                "C": {  
                    "version": "0.0.1",
                    "resolved": "./node_shrinkpack/C-0.0.1.tgz"
                }
            }
        }
    }
}
```
于是，我们以后再进行 `npm install --loglevel=http` 时会发现依赖模块的获取根本没有网络请求了（因为依赖都在我们自己的仓库里了嘛）。

可能有人会说，为啥不直接把 node_modules 目录提交进仓库算了？原因主要是这样：

有些模块需要编译，编译是和环境有关的，你当前的环境编译可用，其他环境直接使用该模块不一定能用。
`node_modules` 目录里面啥东西都有，太凌乱，很容易把提交给搅乱。diff 时突然 diff 出 `node_modules` 下的源代码、README，你应该不想这样吧？
只存储模块的 tgz 包，安装编译的过程交给 NPM 命令更明智。

### 新方式
于是，现在我们使用 NPM 模块的正确姿势应该是这样了：

本地安装、更新需要的模块，测试、验证
执行 `npm shrinkwrap` 将依赖模块的版本冻结
执行 `shrinkpack` . 将依赖模块打包进仓库
提交代码（注意要将 `npm-shrinkwrap.json` 和 `node_shrinkpack` 一起提交哦）
发布模块或者部署应用
如果你觉得这样很繁琐，可以定义一个 NPM 命令：
```
"scripts": {
  "pack": "npm shrinkwrap & shrinkpack ."
}
```
### 总结
拆分模块是必要的，我们应该坚持模块“小而美”
不要太依赖他人，一定要有依赖方挂掉的应急方案
推荐使用 `npm shrinkwrap`（冻结依赖模块的版本） 加 `shrinkpack`（打包依赖模块到自己项目） 来解决依赖模块的不确定性
