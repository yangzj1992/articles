---
title: 使用 npm scripts 构建项目
date: 2016-02-28 11:49:21
categories: 构建工具
tags: [npm,构建工具,Gulp,Grunt]
---
## 背景

最近看到了几篇文章，讲述了运用 npm 来代替 Gulp,Grunt 进行构建的工作，文中一些说法和场景让我感同身受、深表赞同，故在此整理一下相关内容与方法。
原帖地址：
[我为何放弃Gulp与Grunt，转投npm scripts](http://www.infoq.com/cn/news/2016/02/gulp-grunt-npm-scripts-part1)
[Why npm Scripts?](https://css-tricks.com/why-npm-scripts/)
[Why npm Scripts?【译】](http://www.cnblogs.com/zldream1106/p/why-npm-scripts.html)

众所周知，Gulp 与 Grunt 是很多项目所使用的构建工具，他们拥有非常丰富的插件。不过，我却认为 Gulp 与 Grunt 是完全不必要的抽象，npm scripts 更加强大，并且更易于使用。

我本人是 Gulp 的粉丝。不过在上一个项目中，gulpfile 竟然有 100 多行，而且还使用了不少 Gulp 插件。我尝试通过 Gulp 集成 Webpack、Browsersync、热加载、Mocha 等工具，为什么要这么做呢？这是因为有些插件的文档实在是太不充分了；还有些插件只公开了我所需的部分 API。其中有个插件存在一个奇怪的 Bug，它只能看到文件的部分内容。另一个插件则在输出到命令行时丢失了颜色。

当然了，这些问题都是可以解决的；不过，这势必会消耗我们相当的时间成本，最近，我注意到有很多开源项目只是使用了 npm scripts。因此，我决定重新审视一下自己的做法。我真的需要 Gulp 么？答案是并不需要。现在，相对于 Gulp 来说，我更倾向于使用 npm scripts，下面就来谈谈原因。

## Gulp 与 Grunt 怎么了？

随着时间的流逝，我们会逐渐发现诸如 Gulp 与 Grunt 等任务运行器都存在以下 3 个核心问题：

- 对插件作者的依赖
- 令人沮丧的调试
- 脱节的文档
下面就来详细分析上述 3 个问题。

### 问题 1：对插件作者的依赖

在使用比较新或是不那么流行的技术时，可能根本就没有插件。当有插件可用时，插件可能已经过时了。比如说，Babel 6 前一阵发布了。其 API 变化非常大，这样很多 Gulp 插件都无法兼容于最新的版本。在使用 Gulp 时，我就感到深深的受伤，因为我所需要的 Gulp 插件还没有更新。在使用 Gulp 或是 Grunt 时，你不得不等待插件维护者提供更新，或是自己修复。这会阻碍你使用最新版现代化工具的机会。与之相反，在使用 npm scripts 时，我会直接使用工具，不必再添加一个额外的抽象层。这意味着当新版本的 Mocha、Istanbul、Babel、Webpack、Browserify 等发布时，我可以立刻就使用上新的版本。对于选择来说，没有什么能够打败 npm：

![npm 插件对比](http://cdn2.infoqstatic.com/statics_s2_20160217-0123u3/resource/news/2016/02/gulp-grunt-npm-scripts-part1/zh/resources/31.png)

从上图可以看到，Gulp 有将近 2,100 个插件；Grunt 有将近 5,400 个；而 npm 则提供了 227,000 多个包，同时还以每天 400 多个的速度在持续增加。

在使用npm scripts时，你无需再搜索Grunt或是Gulp插件；只需从227,000多个npm包中选择就行了。公平地说，如果所需要的 Grunt 或是 Gulp 插件不存在，你当然可以直接使用 npm packages。不过，这样就无法再针对这个特定的任务使用 Gulp 或是 Grunt 了。

### 问题 2：令人沮丧的调试

如果集成失败了，那么在 Grunt 和 Gulp 中调试是一件令人沮丧的事情。因为你面对的是一个额外的抽象层，对于任何 Bug 来说都有可能存在更多潜在的原因：

- 基础工具出问题了么？
- Grunt/Gulp 插件出问题了么？
- 配置出问题了么？
- 使用的版本是不是不兼容？
使用 npm scripts 可以消除上面的第 2 点，我发现第 3 点也很少会出现，因为我通常都是直接调用工具的命令行接口。最后，第 4 点也很少出现，因为我通过直接使用 npm 而不是任务运行器的抽象减少了项目中包的数量。

### 问题 3：脱节的文档

一般来说，我所需要的核心工具的文档质量总是要比与之相关的 Grunt 和 Gulp 插件的好。比如说，如果使用了 gulp-eslint ，那么我就要在 [gulp-eslint](https://github.com/adametry/gulp-eslint) 文档与 ESLint 网站之间来回切换；不得不在插件与插件所抽象的工具之间来回切换上下文。Gulp 与 Grunt 的问题在于：光理解所用的工具是远远不够的。Gulp 与 Grunt 要求你还得理解插件的抽象。

大多数构建相关的工具都提供了清晰、强大，且具有高质量文档的命令行接口。[ESLint 的 CLI 文档](http://eslint.org/docs/user-guide/command-line-interface)就是个很好的例子。我发现在 npm scripts 中阅读并实现一个简短的命令行调用会更加轻松，阻碍更少，也更易于调试（因为并没有抽象层存在）。既然已经知道了痛点，接下来的问题就在于，为何我们觉得自己还需要诸如 Gulp 与 Grunt 之类的任务运行器呢？

我相信个中原因应该是因人而异的。毫无疑问，Gulp 与 Grunt 等任务运行器已经出现很长一段时间了，而且围绕着这些任务运行器的插件生态圈也呈现出欣欣向荣的繁荣景象。依赖于这些插件，很多日常工作都可以实现自动化，并且运行良好。这样，人们就会认为只有通过这些任务运行器才能实现任务的构建、文件的打包、工作流的良好运行等等。另外一个原因就是人们对于 npm scripts 的认识还远远不够；对于 npm scripts 所能完成的事情与任务也流于表面。这也进一步造成了很多人并没有发现 npm scripts 可以实现很多日常开发时的构建任务的结果。相信随着开发者对于 npm scripts 认识的进一步深入，大家会逐步发现原来使用 npm scripts 也可以完成 Gulp 与 Grunt 等任务运行器所能完成的任务，而且配置更加简单，也更加直接，因为它会直接使用目标工具而不必再使用对目标工具的包装器了。

接下来我们谈谈 npm scripts 的强大功能以及人们为何会忽略 npm scripts。

## 在构建时我们为何会忽略掉 npm？

我认为有如下 4 点原因造成 Gulp 与 Grunt 等任务运行器变得如此流行：

- 人们认为 npm scripts 需要强大的命令行技巧
- 人们认为 npm scripts 不够强大
- 人们认为 Gulp 的流对于快速构建来说是不可或缺的
- 人们认为 npm scripts 无法实现跨平台运行
下面我们将按顺序依次解释一下这些误解。

### 误解 1：使用 npm scripts 需要强大的命令行技巧

体验 npm scripts 的强大功能其实并不需要对操作系统的命令行了解太多。当然了，[grep、sed、awk 与管道](http://www.tutorialspoint.com/unix/unix-useful-commands.htm)等是值得你去学习的，令你众生受用的技能；不过，为了使用 npm scripts，你不必非得成为 Unix 或是 Windows 命令行专家才行。你可以通过 npm 中 1000 多个拥有良好文档的脚本来完成工作。

比如说，你可能不知道在 Unix 中，命令 `rm -rf` 会强制删除一个目录，这没问题。你可以使用 [rimraf](https://www.npmjs.com/package/rimraf) 完成同样的事情（它也是跨平台的）。大多数 npm 包都提供了一些接口，这些接口假设你对所用操作系统的命令行了解不多。只需在 npm 中搜索想要使用的包即可，边做边学就行了。过去，我常常会搜索 Gulp 插件，不过现在则是搜索 npm 包了。[libraries.io](https://libraries.io/)是个非常棒的资源。

### 误解 2：npm scripts 不够强大

npm scripts 本身其实是非常强大的。它提供了基于约定的 [pre 与 post 钩子](https://docs.npmjs.com/misc/scripts#description)：

``` json
{
	name: "npm-scripts-example",
	version: "1.0.0",
	description: "npm scripts example",
	scripts: {
		prebuild: "echo I run before the build script",
		build: "cross-env NODE_ENV=production webpack",
		postbuild: "echo I run after the build script"
	}
}
```
你所要做的就是遵循约定。上述脚本会根据其前缀按照顺序运行。prebuild 脚本会在 build 脚本之前运行，因为他们的名字相同，但 prebuild 脚本有 `pre` 前缀。postbuild 脚本会在 build 脚本之后运行，因为它有 `post` 前缀。因此，如果创建了名为 prebuild、build 与 postbuild 的脚本，那么在我输入 `npm run build` 时，他们就会自动按照这个顺序运行。

此外，还可以通过在一个脚本中调用另一个脚本来对大的问题进行分解：

``` json
{
  "name": "npm-scripts-example",
  "version": "1.0.0",
  "description": "npm scripts example",
  "scripts": {
    "clean": "rimraf ./dist && mkdir dist",
    "prebuild": "npm run clean",
    "build": "cross-env NODE_ENV=production webpack"
  }
}
```
在上述示例中，prebuild 任务调用了 clean 任务。这样就可以将脚本分解为更小、命名良好、单职责，单行的脚本。

可以通过 && 在一行连续调用多个脚本。上述示例中，clean 步骤中的脚本会一个接着一个运行。如果你需要在 Gulp 中按照顺序一个接着一个地运行任务列表中的任务，那么这种简洁性肯定会吸引到你。

如果一个命令很复杂，那还可以调用一个单独的文件：

``` json
{
  "name": "npm-scripts-example",
  "version": "1.0.0",
  "description": "npm scripts example",
  "scripts": {
    "build": "node build.js"
  }
}
```
我在上述的 build 任务中调用了一个单独的脚本。该脚本会被 Node 所运行，这样就可以使用我所需的任何 npm 包了，同时还可以利用上 JavaScript 的能力。我还能列出很多，不过感兴趣的读者可以参考这份[核心特性文档](https://docs.npmjs.com/misc/scripts)。

### 误解 3：Gulp 的流对于快速构建来说是不可或缺的

Gulp 出来后，人们之所以很快就被它吸引过去并放弃 Grunt 的原因在于 Gulp 的内存流要比 Grunt 基于文件的方式快很多。不过，要想享受到流的强大功能，实际上并不需要 Gulp。事实上，流早就已经被内建到 Unix 与 Windows 命令行中了。管道(|)运算符会将一个命令的输出以流的方式作为另一个命令的输入。重定向（>）运算符则会将输出重定向到文件。比如说在 Unix 中，我可以 `grep` 一个文件的内容，并将输出重定向到一个新的文件：

``` bash
grep ‘Cory House’ bigFile.txt > linesThatHaveMyName.txt
```

上述过程是流式的，并不会被写入到中间文件中（想知道如何以跨平台的方式实现上面的命令么？请继续往下读）。

在 Unix 中，还可以通过 “&” 运算符同时运行两个命令：

``` bash
npm run script1.js & npm run script2.js
```

上述两个脚本会同时运行。要想以跨平台的方式同时运行脚本，请使用 [npm-run-all](https://www.npmjs.com/package/npm-run-all)。这就造成了下面这个误解。

### 误解 4：npm scripts 无法实现跨平台运行

很多项目都会绑定到特定的操作系统上，因此跨平台是一件并不那么重要的事情。不过，如果需要以跨平台的方式运行，那么 npm scripts 依然可以工作得很好。无数的开源项目就是佐证。下面来介绍一下实现方式。

操作系统的命令行会运行你的 `npm scripts`。因此，在 Linux 与 OS X 上，`npm scripts` 会在 Unix 命令行中运行。在 Windows 上，`npm scripts` 则运行在 Windows 命令行中。这样，如果希望构建脚本能够运行在所有平台上，你需要适配 Unix 与 Windows。下面介绍 3 种实现方式：

#### 方式 1：使用跨平台的命令

有很多跨平台的命令可供我们使用。下面列举一些：

``` bash
&& 链式任务（一个任务接着一个任务运行）
< 将文件内容输入到一个命令
>  将命令输出重定向到文件
| 将一个命令的输出重定向到另一个命令
```
#### 方式 2：使用 node 包

可以使用 node 包来代替 shell 命令。比如说，使用 [rimraf](https://www.npmjs.com/package/rimraf)来代替“rm -rf`”。使用 [cross-env](https://www.npmjs.com/package/cross-env) 以跨平台的方式设置环境变量。搜索 Google、npm 或是 [libraries.io](https://libraries.io/)，寻找你所需要的，几乎都会有相应的 node 包以跨平台的方式实现你的目标。如果命令行调用过长，你可以在单独的脚本中调用 Node 包，就像下面这样：

``` bash
node scriptName.js
```
上述脚本就是普通的 JavaScript，由 Node 运行。既然是在命令行调用了脚本，那么你就不会受限于 .js 文件。你可以运行操作系统所能执行的任何脚本，比如说 Bash、Python、Ruby 或是 Powershell 等等。

#### 方式 3：使用 [ShellJS](https://www.npmjs.com/package/shelljs)

ShellJS 是个通过 Node 来运行 Unix 命令的 npm 包。这样就可以通过它在所有平台上运行 Unix 命令了，也包括 Windows。

本文主要介绍了人们对于 npm scripts 存在的误解，以及 npm scripts 自身所提供的强大功能。借助于操作系统提供的各种基础设施、npm scripts 以及各种命令，我们完全可以通过 npm scripts 以更加轻量级的方式实现 Gulp 与 Grunt 等任务运行器所提供的功能。

接下来将会介绍 npm scripts 中存在的一些痛点以及解决之道。

## 痛点

显然，使用 npm scripts 也存在着一些问题：JSON 规范并不支持注释，因此无法在 package.json 中添加注释。不过有一些办法可以突破这个限制：

- 编写小巧、命名良好、单一目的的脚本
- 分离文档与脚本（比如说放在 README.md 中）
- 调用单独的 .js 文件

我更偏爱第一种解决方案。如果将每个脚本都进行分解，使其保持单一职责，那么注释就变得不那么重要了。脚本的名字应该能完全描述其意图，就像任何短小、命名良好的函数一样。就像我在 “Clean Code: Writing Code for Humans” 中所说的那样，短小、单一职责的函数几乎是不需要注释的。如果觉得有必要添加注释，那么我会使用第 3 种方案，即将脚本移到单独的文件中。这样就可以利用 JavaScript 组合的强大力量了。

Package.json 也不支持变量。这看起来貌似是个大问题，但实际上并非如此，原因有二。首先，很多时候我们所需的变量都涉及到环境，这可以通过命令行进行设置。其次，如果出于其他原因而需要变量，那么你可以调用单独的 .js 文件。

最后，还存在一种风险，那就是使用长长的、复杂的命令行参数，这些参数令人难以理解。代码审查与重构是确保 npm 脚本保持小巧、命名良好、单一职责，且每个人都能容易理解的好方式。如果脚本复杂到需要注释，那么你应该将单个脚本重构为多个命名良好的脚本，或是将其抽取为单独的文件。

### 我们需要证明抽象是有意义的

Gulp 与 Grunt 是对我所使用的工具的抽象。抽象是很有用的，不过抽象是有代价的。它让我们过分依赖于插件维护者与文档，同时随着插件数量的不断攀升，他们也不断引入复杂性。此外不少开发者与此观点不谋而合，比如说下面这些：

- [Task automation with npm run](http://substack.net/task_automation_with_npm_run)—James Holliday
- [Advanced front-end automation with npm scripts](https://www.youtube.com/watch?v=0RYETb9YVrk)—Kate Hudson
- [How to use npm as a build tool](http://blog.keithcirkel.co.uk/how-to-use-npm-as-a-build-tool/)—Kieth Cirkel
- [Introduction to npm as a Build Tool](http://app.pluralsight.com/courses/npm-build-tool-introduction)—Marcus Hammarberg
- [Gulp is awesome, but do we really need it?](http://gon.to/2015/02/26/gulp-is-awesome-but-do-we-really-need-it/)—Gonto
- [NPM Scripts for Build Tooling](http://code.tutsplus.com/courses/npm-scripts-for-build-tooling)—Andrew Burgess


## npm 替换方案参考
近来直接使用 node package 提供的命令行界面的情绪愈发高涨，反之，人们对通过运行任务从而屏蔽抽象功能的热情逐渐降温。在一定程度是，你无论如何都要使用 npm，而同时 npm 也提供了脚本功能，为什么不用呢？

Grunt, Gulp, Broccoli, Brunch 和类似的工具都需要将自己的任务配置的适合他们的范型和结构，这些工具每一个都需要学习他们不同的语法、奇怪的用法和特有的方法。这增加了编码复杂度、构建复杂度，使得你关注修复工具问题更甚于写代码。

由于这些构建工具依赖于包装了核心命令行工具的插件，并基于这个核心工具做了进一步的抽象，这使得出错的情况变得更加复杂。

但是，作者在此建议:

**如果你对当前的构建系统很满意，并且它能够很好的完成你的需求的话，就请继续使用吧！不要因为 npm scripts 越来越流行就盲目的使用它，应该把精力集中在写代码而不是学习更多的工具。如果你开始觉得自己正在和使用的工具战斗，那么这个时候我建议你考虑使用一下 npm scripts。**

如果你现在做好决定想要调研或使用 npm scripts，那么请继续阅读本文！本文将会提供大量的案例任务，同时基于这些任务我也创建了 npm-build-boilerplate 以方便你学习。那么下面让我们开始吧！

### 写 npm scripts

我们会花费大量的时间在 `package.json` 文件上。这个文件描述了我们需要的所有的依赖和脚本。以下是我的 boilerplate 项目中的一部分内容：

``` json
 1 {
 2   "name": "npm-build-boilerplate",
 3   "version": "1.0.0",
 4   "scripts": {
 5     ...
 6   },
 7   "devDependencies": {
 8     ...
 9   }
10 }
```


接下来我们将逐步创建 `package.json` 文件。我们的脚本会写入 scripts 对象中，所有我们要使用的工具都会被安装并且写入 devDependencies 对象中。在开始之前，以下是本文中的项目结构:
![项目结构](https://cdn.css-tricks.com/wp-content/uploads/2016/02/directory.png)

### 编译 scss 为 css

为了将 SCSS 编译成 CSS，我使用了 node-sass。首先，我们需要安装 node-sass，在命令行下运行以下代码：

``` bash
npm install --save-dev node-sass
```
这个命令会在当前目录下安装 node-sass，并添加到 `package.json` 的 devDependencies 对象中。当其他人使用你的项目时会非常方便，因为他们已经有了运行项目所需的所有内容。只要安装过一次，使用时在命令行运行以下代码即可：

``` bash
node-sass --output-style compressed -o dist/css src/scss
```
让我们看一下这个命令做了什么：从后向前看，查找 `src/scss` 目录的 SCSS 文件，输出（-o 标识）编译的 CSS 到`dist/css`目录，压缩输出文件（使用 --output-style 标识，设置选项值为"compressed"）。
现在我们知道了在命令行中如何工作，那么让我们把它放到 npm scirpt 中。在`package.json`的 scripts 对象中添加如下内容:

``` json
"scripts": {
  "scss": "node-sass --output-style compressed -o dist/css src/scss"
}
```
现在回到命令行并运行：

``` bash
npm run scss
```
可以看到这样运行的输出结果和直接在命令行使用 node-sass 命令得到的结果一致。
本文剩余部分创建的任何一个 npm script，都可以像上例一样使用命令行运行。

只要把你想要运行的任务名从 scss 替换成你想要的名字即可。

你将看到，我们使用的很多命令行工具都有很多的配置项，你可以使用配置项来精确完成你想要做的工作。比如，这是node-sass的选项列表，以下展示了传多个配置项的配置方法：

``` json
"scripts": {
  "scss": "node-sass --output-style nested --indent-type tab --indent-width 4 -o dist/css src/scss"
}
```
### 使用 PostCSS 自动给 CSS 加前缀

我们已经能够把 SCSS 编译成 CSS，现在我们希望通过 Autoprefixer 和 PostCSS 自动给 CSS 代码添加厂商前缀，我们可以通过空格分隔的方式从而同时安装多个模块：

``` bash
npm install --save-dev postcss-cli autoprefixer
```
因为 PostCSS 默认不做任何事情，所以我们安装了两个模块。PostCSS 依赖其他的插件来处理你提供的 CSS，比如 Autoprefixer。
安装并保存必要工具到 devDependencies 后，在你的 scripts 对象中添加一个新任务:

``` json
"scripts": {
  ...
  "autoprefixer": "postcss -u autoprefixer -r dist/css/*"
}
```
这个任务的意思是：嗨 postcss，使用（-u 标识符）Autoprefixer 替换（-r标识符）`dist/css`目录下的所有`.css`文件，给他们添加厂商前缀代码。就是这样简单！想要修改默认浏览器前缀？只要给脚本添加如下代码即可:

``` json
"autoprefixer": "postcss -u autoprefixer --autoprefixer.browsers '&gt; 5%, ie 9' -r dist/css/*"
```
再次申明，配置你自己的构建代码有很多选项可以使用：postcss-cli 和 autoprefixer。
### JavaScript 代码检查

对于写代码来说，保持标准格式和样式是非常重要的，它能够确保错误最小化并提高开发效率。"代码检查"帮助我们自动化的完成了这个工作，所以我们通过使用 eslint 来进行代码检查。

再次如上文所述，安装 eslint 的包，这次让我们使用快捷方式:

``` bash
npm i -D eslint
```
这和如下代码是一样的效果：

``` bash
npm install --save-dev eslint
```
安装完成后，我们给 eslint 配置一些运行代码的基本规则。使用如下代码开始一个向导：

``` bash
eslint --init // 译者注：这里直接使用会抛错 eslint 找不到，因为这种使用方法必须安装在全局。
即通过 npm install i -g eslint 方式安装
```

我建议选择"回答代码风格问题"并回答提问的相关问题。这个过程中 eslint 会在你的项目根目录下生成一个新文件，并检测你的相关代码。
现在，让我们把代码风格检测任务添加到 `package.json` 的 scripts 对象中：

``` json
"scripts": {
  ...
  "lint": "eslint src/js"
}
```
我们的任务仅有 13 字符！它会查找 `src/js` 目录下的所有 JavaScript 文件，并根据刚才生成的规则进行代码检测。当然，如果感兴趣的话你可以详细配置各种规则：[get crazy with the options](http://eslint.org/docs/user-guide/command-line-interface#options)

### 混淆压缩 JavaScript 文件

让我们继续，下面我们需要使用 uglify-js 来混淆压缩 JavaScript 文件，首先需要安装 uglify-js：

``` bash
npm i -D uglify-js
```
然后我们可以在 `package.json` 里创建压缩混合任务：

``` json
"scripts": {
  ...
  "uglify": "mkdir -p dist/js && uglifyjs src/js/*.js -m -o dist/js/app.js"
}
```
npm scripts 的任务的本质是：可以重复执行的、命令行任务的快捷方式（别名），这也是 npm scripts 的优点之一。这就意味着你可以直接在脚本里使用标准命令行代码！这个任务使用了两个标准命令行特性：mkdir 和 &&。

这个任务的第一部分“ mkdir -p dist/js ”：如果不存在目录（-p 标识）就创建一个目录结构（mkdir），创建成功后执行 uglifyjs 命令。&& 帮助你连接多个命令，如果前一个命令成功执行的话，就分别顺序执行剩余的命令。

这个任务的第二部分告诉 uglifyjs 针对 `src/js/` 目录下的所有 JS 文件（`*.js`），使用 `mangle`命令（-m 标识），输出结果到 `dist/js/app.js` 文件中。这里是 uglifyjs 工具的全部配置选项 list of options。

让我们来更新一下 uglify 任务，创建一个 `dist/js/app.js` 的压缩版本，链接另外一个 uglifyjs 的命令并传参给 `compress`（-c标识）。

``` json
"scripts": {
  ...
  "uglify": "mkdir -p dist/js && uglifyjs src/js/*.js -m -o dist/js/app.js && uglifyjs src/js/*.js -m -c -o dist/js/app.min.js"
}
```
### 压缩图片

下面我们将进行图片压缩的工作。根据 httparchive.org 的数据统计，网络上前 1000 名的网站平均页面大小为 1.9 M ，其中图片占了 1.1 M（with images accounting for 1.1mb of that total）。所以提高网页加载速度的其中一个好办法就是减小图片大小。

安装 imagemin-cli:

``` bash
npm i -D imagemin-cli
```
Imagemin 非常棒，它可以压缩大多数图片类型，包括 GIF、JPG、PNG 和 SVG。 使用如下代码可以将一整个文件夹的图片全部压缩：

``` json
"scripts": {
  ...
  "imagemin": "imagemin src/images dist/images -p",
}
```
这个任务告诉 imagemin 找到并压缩 `src/images` 中的所有图片并输出到 `dist/images`中。-p 标志在允许的情况下将图片处理成渐进图片。更多配置可查看文档 all available options

### SVG 精灵（Sprites）

关于 SVG 的讨论近年来逐渐火热，SVG 有众多优点：在所有的设备上保持松散结构、可通过 CSS 编辑、对读屏软件友好。然而，SVG 编辑软件经常会产生大量的冗余代码。幸运的是，svgo 可以帮助你自动删除这些冗余信息（我们马上就会安装 svgo）。

接下来我们来安装 svg-sprite-generator，用于自动处理并整合多个 SVG 文件为一个 SVG 文件（更多处理方案：more on that technique here）。

``` bash
npm i -D svgo svg-sprite-generator
```
你现在应该已经熟悉了这个过程——添加一个任务在你的 `package.json` scripts 对象中：

``` json
"scripts": {
  ...
  "icons": "svgo -f src/images/icons && mkdir -p dist/images && svg-sprite-generate -d src/images/icons -o dist/images/icons.svg"
}
```
注意 icons 任务通过两个 && 引导符做了三件事情：
1. 使用 svgo 传参一个 SVG 目录（-f标识），这个操作压缩了目录内的全部 SVG 文件；
2. 如果不存在 'dist/images' 目录则创建该目录（使用 mkdir -p命令）；
3. 使用 svg-sprite-generator，传参一个 SVG 目录（-d 标识）以及输出处理后的 SVG 文件的目录路径名（-o 标识）。

### 通过 BrowserSync 提供服务、自动监测并注入变更

最后一个插件是 BrowserSync，它可以做如下事情：启动一个本地服务，向连接的浏览器自动注入更新的文件，并同步浏览器的点击和滚动。安装并添加任务的代码如下：

``` bash
npm i -D browser-sync
"scripts": {
  ...
  "serve": "browser-sync start --server --files 'dist/css/*.css, dist/js/*.js'"
}
```
BrowserSync 任务默认使用当前根目录下的路径启动一个服务器（`--server` 标识），`--files` 标识告诉 BrowserSync 去监测 `dist` 目录的 CSS 或 JS 文件，一旦有任何变化，则自动向页面注入变化的文件。

你可以同时打开多个浏览器（甚至在不同的设备上），他们都会实时更新文件变化的！

### 分组任务

使用以上任务我们可以做到如下功能：

编译 SCSS 到 CSS 并自动添加厂商前缀
对 Javascript 进行代码检查及混淆压缩
压缩图片
整合整个文件夹内的 SVG 文件为一个 SVG 文件
启动一个本地服务并向连接至该服务的浏览器自动注入更新。
这还不是全部内容！

### 合并 CSS 任务

我们会添加一个新的任务，用于合并两个 CSS 相关的任务（处理 SASS 和执行加前缀的 Autoprefixer），有了这个任务我们就不用分别执行两个相关任务了：

``` json
"scripts": {
  ...
  "build:css": "npm run scss && npm run autoprefixer"
}
```
当你运行 `npm run build:css` 时，这个任务会告诉命令行去执行 `npm run scss`，当这个任务成功完成后，会接着（&&）执行 `npm run autoprefixer`。

就像这个 build:css 任务一样，我们可以把 JavaScript 任务也链接到一起以方便执行：

### 合并 JavaScript 任务

``` json
"scripts": {
  ...
  "build:js": "npm run lint && npm run uglify"
}
```
现在，我们可以通过 `npm run build:js` 一步调用，来进行代码检测、混淆压缩 JavaScript 代码了！

合并剩余任务

对于图片任务、其他剩余构建任务，我们可以用相同的方法把他们变成一个任务：

``` json
"scripts": {
  ...
  "build:images": "npm run imagemin && npm run icons",
  "build:all": "npm run build:css && npm run build:js && npm run build:images",
}
```
### 变更监控

至此，我们的任务不断的需要对文件做一些变更，我们不断的需要切回命令行去运行相应的任务。针对这个问题，我们可以添加一个任务来监听文件变更，让文件在变更的时候自动执行这些任务。这里我推荐使用 onchange 插件，安装方法如下：

``` bash
npm i -D onchange
```
让我们来给CSS和JavaScript设置监控任务：

``` json
"scripts": {
  ...
  "watch:css": "onchange 'src/scss/*.scss' -- npm run build:css",
  "watch:js": "onchange 'src/js/*.js' -- npm run build:js",
}
```
这些任务可以分解如下：onchange 需要你传参想要监控的文件路径（字符串），这里我们传的是 SCSS 和 JS 源文件，我们想要运行的命令跟在--之后，这个命令当路径内的文件发生增删改的时候就会被立即执行。

让我们再添加一个监控命令来完成我们的 npm scripts 构建过程。

再添加一个包，[parallelshell](https://github.com/keithamus/parallelshell	)：

``` bash
npm i -D parallelshell
```
再次给 scripts 对象添加一个新任务：

``` json
"scripts": {
  ...
  "watch:all": "parallelshell 'npm run serve' 'npm run watch:css' 'npm run watch:js'"
}
```
parallelshell 支持多个参数字符串，它会给 npm run 传多个任务用于执行。

为什么时候 parallelshell 去合并多个任务，而不是像之前的任务一样使用 && 呢？最开始我也尝试这么做了，但是问题是：&& 链接多个命令到一块，需要等待每一个命令成功完成后才会执行下一个任务。然而当我们运行 watch 命令时，这些命令一直都不会结束！这样我们就会卡在一个无限循环里。

因此，使用 parallelshell 使得我们可以同时执行多个 watch 命令。（译者注：后来在评论里有人推荐使用 npm-run-all 插件来代替 parallelshell，它支持这种用法可以一次性检测全部 watch 任务更加方便："watch": "npm-run-all --parallel serve watch:*"）

这个任务使用了 BrowserSync 的 npm run serve 任务启动了一个服务，然后对 CSS 和 JavaScript 文件执行了监控命令，一旦 CSS 或 JavaScript 文件有变更，监控任务就会执行相应的构建（build）任务。由于 BrowserSync 被设置成监控 `dist` 目录下的变更，所以它会自动的向相关联的 URL 内注入新的文件，真是太棒了！

### 其他实用任务

npm 有大量可以实用的插件（[lots of baked in tasks](https://docs.npmjs.com/misc/scripts#description) ），让我们再添加一个新的任务来看看这些插件对构建脚本的影响：

``` json
"scripts": {
  ...
  "postinstall": "npm run watch:all"
}
```
当你在命令行中执行 `npm install` 的时候 postinstall 会立即执行，当团队合作时这个功能非常有用。当别人复制了你的项目并运行了 npm install 的时候，我们的 watch:all 任务就会马上执行，别人马上就会准备好一切开发环境：启动一个服务、打开一个浏览器窗口、监控文件变更。

### 打包
万一你忘记了什么知识点，我用以上所有提到的任务创建了一个 [npm-build-boilerplate](https://github.com/damonbauer/npm-build-boilerplate) 项目以方便你学习。
