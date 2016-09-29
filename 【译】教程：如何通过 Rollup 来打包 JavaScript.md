title: 【译】教程：如何通过 Rollup 来打包 JavaScript
date: 2016-09-29 23:58:14
categories: 前端工程
tags: [Rollup,webpack,构建工具,前端工程]
---
> 原文链接 : [Tutorial: How to Bundle JavaScript With Rollup](https://code.lengstorf.com/learn-rollup-js/)   
> 原文作者 : [jlengstorf](https://github.com/jlengstorf/)   
> 译文出自 : [众成翻译](http://zcfy.cc/)   
> 译者 : [yangzj1992](http://qcyoung.com)   
> 校对者: [lisa](https://www.zhihu.com/people/ha-ha-qiu-52)   
> 首发于: [众成翻译](http://zcfy.cc/article/how-to-bundle-javascript-with-rollup-step-by-step-tutorial-1254.html)

> 本文将通过一步步的系列教学来学习如何使用更小，更高效的工具 Rollup 来代替 Webpack 和 Browserify 打包 JavaScript 文件。

这周，我们将第一次用 [Rollup](http://rollupjs.org/) 来构建我们的项目，它是一款用来打包 Javascript 代码的构建工具（它同样支持样式表，但我们将在下周单独介绍这一点）
> 译者注：原文系列文章如下 ——
[Part I: How to Use Rollup to Process and Bundle JavaScript Files](https://code.lengstorf.com/learn-rollup-js/)
[Part II: How to Use Rollup to Process and Bundle Stylesheets](https://code.lengstorf.com/learn-rollup-css/)
[Part III: How to Use Rollup to Watch and Live Reload Files During Development](https://code.lengstorf.com/learn-rollup-css/#livereload)）

通过本教程，我们会了解到以下 Rollup 的相关配置方式：

*  组合我们的脚本,
*  移除未使用的代码,
*  转译代码使其支持老版本浏览器,
*  在浏览器中支持使用 Node modules,
*  使用环境变量,
*  压缩文件代码使文件大小尽可能最小化。

## 预备知识

*  如果你了解 JavaScript 的相关知识那么对阅读此文会更有帮助。
*  熟悉 [ES2015 modules](https://github.com/getify/You-Dont-Know-JS/blob/master/es6%20%26%20beyond/ch3.md#modules)当然更没有什么坏处。
*  你需要在你的机器上安装 `npm` (还没有安装? [在这安装 Node.js](https://nodejs.org/))

## 什么是 Rollup?

Rollup 是下一代的 JavaScript 模块打包工具。当你使用 ES2015 模块来编写你的应用或者库时，它可以对它们有效的打包来成为一个单独文件供浏览器和 Node.js 使用。

这与 [Browserify](http://browserify.org/) 和 [webpack](https://webpack.github.io) 很相似。

你也可以把 Rollup 称为一个构建工具，类似于 [Grunt](http://gruntjs.com/) 和 [Gulp](https://github.com/gulpjs/gulp)。然而，你需要重点注意的是当你使用 Grunt 和 Gulp 去处理类似创建 JavaScript bundles 的任务时，这些工具在底层也会像 Rollup, Browserify 或是 webpack 一样去使用一些相同的方式来处理。

### 为什么你需要关心 Rollup?

Rollup 如此令人兴奋的原因在于它能够保持文件体积更小。这听上去很傻瓜，所以 **tl;dr** 版本在这：相比于其他工具创建的 JavaScript bundles，Rollup 总是会创建相比之更小，更快的 bundle。

之所以会这样是因为 Rollup 是基于 ES2015 模块的，它相比于 webpack 和 Browserify 所使用的 CommonJS 模块更加具有效率，另外 Rollup 也会使用一种叫 _tree-shaking_ 的特性来更容易的移除模块中未使用的代码，这意味着在最终的 bundle 中只有我们实际需要的代码。

Tree-shaking 在我们引用了包含很多可用的函数或方法的第三方工具或框架时就会变得十分重要。例如我们只使用它们中的一两个方法时 —— 像 [lodash](https://lodash.com/) 或 [jQuery](https://jquery.com/) 这样的库就会在加载时产生**许多**额外的开销。

目前 Browserify 和 webpack 在最终生成时仍会包含大量未使用的代码（译者注：在 webpack2 中也引入了 tree-shaking 特性）。但 Rollup 并不如此 —— 它只会生成我们最终实际使用的代码。

**(2016 年 8 月 22 日更新)** 澄清一下， Rollup 只会在 ES 模块中支持 tree-shaking 特性。目前依照 CommonJS 模块所编写的 lodash 和 jQuery 不能被支持 tree-shaken。然而 tree-shaking **并不是** Rollup 唯一的速度和性能上的优势。可以看这些文章了解更多信息[Rich Harris’s explanation](https://www.reddit.com/r/javascript/comments/4yprc5/how_to_bundle_javascript_with_rollup_stepbystep/d6qzgzm)、[Nolan Lawson’s added info](https://www.reddit.com/r/javascript/comments/4yprc5/how_to_bundle_javascript_with_rollup_stepbystep/d6qzmgh?context=3)

## 第一部分：如何使用 Rollup 来处理并打包 JavaScript 文件

为了展示 Rollup 是如何运作的，让我们一起针对一个十分简单的项目用 Rollup 处理打包 JavaScript。

### 第 0 步: 创建一个包含 JavaScript 和 CSS 的打包项目

开始之前，我们需要一个项目来进行工作。在此教学中，我们将会根据此 [Github 项目](https://github.com/jlengstorf/learn-rollup)来进行展示。

目录结构是这样的：

```
learn-rollup/
├── src/
│   ├── scripts/
│   │   ├── modules/
│   │   │   ├── mod1.js
│   │   │   └── mod2.js
│   │   └── main.js
│   └── styles/
│       └── main.css
└── package.json 
```

你可以在你的终端中执行下面的命令来安装此项目，在本教学中我们将通过此项目进行展示。

```
# Move to the folder where you keep your dev projects.
cd /path/to/your/projects

# Clone the starter branch of the app from GitHub.
git clone -b step-0 --single-branch https://github.com/jlengstorf/learn-rollup.git

# The files are downloaded to /path/to/your/projects/learn-rollup/ 
```

### 步骤 1：安装 Rollup 并创建配置文件

首先，通过下面的命令安装 Rollup:

```
npm install --save-dev rollup
```

接下来，在 `learn-rollup` 文件夹中创建一个新文件 `rollup.config.js`。之后在文件中添加下面的内容：

```
export default {
  entry: 'src/scripts/main.js',
  dest: 'build/js/main.min.js',
  format: 'iife',
  sourceMap: 'inline',
}; 
```

下面是每一个配置选项都做了些什么：

*  `entry ` —— 这是我们希望 Rollup 去执行的文件。在大多数项目中，这将是你主要的 JavaScript 文件，它负责一切的初始化工作并作为开始文件。

*  `dest` —— 这是脚本程序执行后所存储的位置。

*  `format` —— Rollup 支持多种输出格式。因为我们在浏览器中运行，我们希望使用[立即调用的函数表达式](http://benalman.com/news/2010/11/immediately-invoked-function-expression/)(immediately-invoked function expression,IIFE)。[^IIFE]

*  `sourceMap` —— 如果有 sourcemap 的话，那么在调试代码时会提供很大的帮助，这个选项会在生成文件中添加 sourcemap，来让事情变得更加简单。

**注意：** 关于其他的 `format` 选项以及什么场合你可能会需要它们，可以参考 [Rollup’s wiki](https://github.com/rollup/rollup/wiki/JavaScript-API#format)

#### 测试 Rollup 配置

一旦我们创建了配置文件，就可以在我们的终端里运行下面的代码进行测试了：

```
./node_modules/.bin/rollup -c
```

这样会在你的项目中创建一个新的 `build` 文件夹，它包含了一个 `js` 的子文件夹，在 `js` 文件夹中还包含了我们生成的 `main.min.js` 文件。

我们在浏览器中打开 `build/index.html` 可以看到 bundle 已经被正确创建了。

**注意：** 在这个阶段，只有现代浏览器会正常工作并不产生错误。如果要让不支持`ES2015/ES6` 的旧版本的浏览器也正常工作，我们需要添加一些插件。

#### 打包输出的内容

Rollup 如此强大的原因在于它的 “tree-shaking” 特性，它会将我们引用的模块中未使用的代码剥离。例如，在 `src/scripts/modules/mod1.js` 中有一个名为 `sayGoodbyeTo()` 的函数并未在你的项目中使用 —— 既然它不会被使用，那么 Rollup 在最后的 bundle 中就不会包含它：

```
(function () {
'use strict';

/**
 * Says hello.
 * @param  {String} name a name
 * @return {String}      a greeting for `name`
 */
function sayHelloTo( name ) {
  const toSay = `Hello, ${name}!`;

  return toSay;
}

/**
 * Adds all the values in an array.
 * @param  {Array} arr an array of numbers
 * @return {Number}    the sum of all the array values
 */
const addArray = arr => {
  const result = arr.reduce((a, b) => a + b, 0);

  return result;
};

// Import a couple modules for testing.
// Run some functions from our imported modules.
const result1 = sayHelloTo('Jason');
const result2 = addArray([1, 2, 3, 4]);

// Print the results on the page.
const printTarget = document.getElementsByClassName('debug__output')[0];

printTarget.innerText = `sayHelloTo('Jason') => ${result1}\n\n`
printTarget.innerText += `addArray([1, 2, 3, 4]) => ${result2}`;

}());
//# sourceMappingURL=data:application/json;charset=utf-8;base64,... 
```

而在其他的构建工具中却并不会如此，因此如果我们引用了一个很大的库如 [lodash](https://lodash.com/) 却只为了使用它一到两个方法的话，最后的 bundles 会**十分**的巨大。

例如，使用 [webpack](https://webpack.github.io) 的话，`sayGoodbyeTo()` 函数就会被引入，并且最终的 bundle 体积相比于 Rollup 所生成的 bundle 要大两倍还多。[^size]

### 步骤 2： 设置 Babel 来使用新的 JavaScript 特性。

此时，我们获得了可以在现代浏览器中运行的代码包，然而如果访问的浏览器仍是旧版本的话那么就会产生错误 —— 这样不太理想。

幸好，[Babel](https://babeljs.io) 已经提供了支持。它能够帮助我们[转译 JavaScript 新特性](https://scotch.io/tutorials/javascript-transpilers-what-they-are-why-we-need-them)[(ES6/ES2015 等等)](https://github.com/getify/You-Dont-Know-JS/blob/master/es6%20%26%20beyond/ch1.md)到 ES5 版本，这也将支持目前所有的浏览器来正常运行代码。

如果你从未使用过 Babel，那么从今以后你作为开发者的日子就要永远改变了。去尝试 JavaScript 的新特性可以让你的语言变得更简单、干净，让你的开发更加愉快。

所以让我们立刻开始 Rollup 的这一部分吧。

#### 安装必要的模块

首先，我们需要安装 [Babel Rollup 插件](https://github.com/rollup/rollup-plugin-babel) 和 [合适的 Babel preset](https://github.com/rollup/babel-preset-es2015-rollup)

```
# Install Rollup’s Babel plugin.
npm install --save-dev rollup-plugin-babel

# Install the Babel preset for transpiling ES2015 using Rollup.
npm install --save-dev babel-preset-es2015-rollup 
```

**注意:** Babel preset 是一个有关 Babel 插件的集合，它会告诉 Babel 我们实际上想要转译什么。

#### 创建 `.babelrc`

接下来，在你的项目根目录(`learn-rollup/`)创建一个名为 `.babelrc` 的新文件，在它内部添加以下 JSON 内容：

```
{
  "presets": ["es2015-rollup"],
} 
```

这会告诉 Babel 它应该使用哪种 preset 来转译代码。

#### 更新 `rollup.config.js`

要让它能够真正运行，我们需要更新 `rollup.config.js`。

在 `rollup.config.js` 中，我们需要 `import` Babel 插件，将它添加到一个新的配置选项 `plugins` 中，它会管控一个数组形式的插件列表。

```
// Rollup plugins
import babel from 'rollup-plugin-babel';

export default {
  entry: 'src/scripts/main.js',
  dest: 'build/js/main.min.js',
  format: 'iife',
  sourceMap: 'inline',
  plugins: [
    babel({
      exclude: 'node_modules/**',
    }),
  ],
}; 
```

为了避免转译第三方脚本，我们需要设置一个 `exclude` 的配置选项来忽略掉 `node_modules` 目录

#### 检查输出的 bundle

安装和配置完成后，我们可以重新构建 bundle:

```
./node_modules/.bin/rollup -c

// 译者注：若执行报错，运行 npm install --save-dev babel-preset-es2015 具体issue 详情见：https://github.com/jlengstorf/learn-rollup/issues/2
```

当我们观察输出时，它看上去**貌似没有什么**改变。然而实际上它还是有一些细微的区别的：例如， `addArray()` 函数：

```
var addArray = function addArray(arr) {
  var result = arr.reduce(function (a, b) {
    return a + b;
  }, 0);

  return result;
}; 
```

这里可以看到 Babel 如何转换 [箭头表示函数](https://strongloop.com/strongblog/an-introduction-to-javascript-es6-arrow-functions/) (`arr.reduce((a, b) => a + b, 0)`) 到一个常规函数。

在转译运行完成后，程序执行的结果依然相同，但是代码已经支持到了 IE9 之前的浏览器。

**重点:** Babel 也提供了 [`babel-polyfill`](https://babeljs.io/docs/usage/polyfill/)，它可以让类似像 `Array.prototype.reduce()` 的代码可以在 IE8 以及更早的浏览器上能够得到顺利执行。

### 步骤 3: 添加 ESLint 来检查通常的 JavaScript 错误。

在你的代码中使用 linter 无疑是十分好的决定，因为它会强制执行一致的编码规范来帮助你捕捉像是漏掉了括弧这种棘手的 bug。 

在这个项目中，我们将会使用 [ESLint](http://eslint.org/)。

#### 安装模块

为了使用 ESLint，我们将要安装 [ESLint Rollup plugin](https://github.com/TrySound/rollup-plugin-eslint)

```
npm install --save-dev rollup-plugin-eslint
```

#### 生成一个 `.eslintrc.json`。

为了确保我们只获取我们想要的错误，我们需要首先配置 ESLint。这里可以通过下面的代码来自动生成大多数配置：

```
$ ./node_modules/.bin/eslint --init
? How would you like to configure ESLint? Answer questions about your style
? Are you using ECMAScript 6 features? Yes
? Are you using ES6 modules? Yes
? Where will your code run? Browser
? Do you use CommonJS? No
? Do you use JSX? No
? What style of indentation do you use? Spaces
? What quotes do you use for strings? Single
? What line endings do you use? Unix
? Do you require semicolons? Yes
? What format do you want your config file to be in? JSON
Successfully created .eslintrc.json file in /Users/jlengstorf/dev/code.lengstorf.com/projects/learn-rollup 
```

如果你回答了上述的问题，你将会在 `.eslintrc.json` 中获得以下输出内容：

```
{
  "env": {
    "browser": true,
    "es6": true
  },
  "extends": "eslint:recommended",
  "parserOptions": {
    "sourceType": "module"
  },
  "rules": {
    "indent": [
      "error",
      4
    ],
    "linebreak-style": [
      "error",
      "unix"
    ],
    "quotes": [
      "error",
      "single"
    ],
    "semi": [
      "error",
      "always"
    ]
  }
} 
```

#### 调整 `.eslintrc.json`。

然而，我们还需要去做一些调整来避免我们的项目出现问题：

1. 我们要用 2 缩进符来代替 4 缩进符。

2. 我们将使用一个全局变量 `ENV`，所以我们需要为它设置一个白名单。

所以我们来做以下调整 —— 在你的 `.eslintrc.json` 中修改 `globals` 属性和 `indent` 属性：

```
{
  "env": {
    "browser": true,
    "es6": true
  },
  "globals": {
    "ENV": true
  },
  "extends": "eslint:recommended",
  "parserOptions": {
    "sourceType": "module"
  },
  "rules": {
    "indent": [
      "error",
      2
    ],
    "linebreak-style": [
      "error",
      "unix"
    ],
    "quotes": [
      "error",
      "single"
    ],
    "semi": [
      "error",
      "always"
    ]
  }
}
```

#### 更新 `rollup.config.js`。

接下来，`import` ESLint 插件并将它添加到 Rollup 配置中：

```
// Rollup plugins
import babel from 'rollup-plugin-babel';
import eslint from 'rollup-plugin-eslint';

export default {
  entry: 'src/scripts/main.js',
  dest: 'build/js/main.min.js',
  format: 'iife',
  sourceMap: 'inline',
  plugins: [
    babel({
      exclude: 'node_modules/**',
    }),
    eslint({
      exclude: [
        'src/styles/**',
      ]
    }),
  ],
}; 
```

#### 检查控制台输出

首先，我们运行 `./node_modules/.bin/rollup -c`，然而好像并没有发生什么，这是因为在标准设置下，应用代码已经顺利通过了 linter 的检查并且没有发现任何问题。

但是如果我们引入一个问题 —— 比如移除一个分号 —— 我们则会看到 ESLint 的帮助提示:

```
$ ./node_modules/.bin/rollup -c

/Users/jlengstorf/dev/code.lengstorf.com/projects/learn-rollup/src/scripts/main.js
  12:64  error  Missing semicolon  semi

✖ 1 problem (1 error, 0 warnings) 
```

像这样一些无意间引入的神秘 bug 就会被立刻发现，帮助信息中也包含了文件名，行数以及列数。

尽管这并不会消除我们项目中所有需要调试的问题，但对明显的拼写错误和疏忽起到了相当大的帮助。[^help]

### 步骤4：添加插件来处理非 ES 模块。

如果你的依赖项使用了 Node 模式的模块那么下面的插件是很重要的。如果没有它，你将会在 `require` 时产生错误。

#### 添加 Node 模块作为依赖项。

在这个示例项目中如果没有引用第三方模块的话将会变得很麻烦，但这并不是让你将第三方模块剪切到实际项目中。所以，为了使我们的 Rollup **更方便使用**，让我们为代码添加引用第三方模块的功能。

为了简单起见，我们将在代码中添加一个 [`debug`](https://www.npmjs.com/package/debug) 包来简单的记录日志，通过下面的命令安装：

```
npm install --save debug
```

**注意:** 由于这将被引用到主项目中，使用 `--save` 参数是很重要的，这将避免在生产环境中由于 `devDependencies` 没有被安装而导致的错误。

然后，在 `src/scripts/main.js`中，我们添加一些简单的日志：

```
// Import a couple modules for testing.
import { sayHelloTo } from './modules/mod1';
import addArray from './modules/mod2';

// Import a logger for easier debugging.
import debug from 'debug';
const log = debug('app:log');

// Enable the logger.
debug.enable('*');
log('Logging is enabled!');

// Run some functions from our imported modules.
const result1 = sayHelloTo('Jason');
const result2 = addArray([1, 2, 3, 4]);

// Print the results on the page.
const printTarget = document.getElementsByClassName('debug__output')[0];

printTarget.innerText = `sayHelloTo('Jason') => ${result1}\n\n`;
printTarget.innerText += `addArray([1, 2, 3, 4]) => ${result2}`; 
```

到目前为止一切顺利，但是当我们运行 rollup 时我们会得到一个警告：

```
$ ./node_modules/.bin/rollup -c
Treating 'debug' as external dependency
No name was provided for external module 'debug' in options.globals – guessing 'debug' 
```

如果我们再一次检查我们的 `index.html` ，我们会发现 `debug` 会抛出一个 `ReferenceError`

通常情况下，第三方 Node 模块并不会被 Rollup 正确加载

这是由于 Node 模块使用的是 [CommonJS](http://wiki.commonjs.org/wiki/Modules/1.1)，它并不被 Rollup 兼容因此不能直接使用。为了解决它，我们需要添加一些插件来处理 Node 依赖和 CommonJS 模块。

#### 安装模块。

为了解决这个问题，我们准备为 Rollup 添加两个插件:

1.  [`rollup-plugin-node-resolve`](https://github.com/rollup/rollup-plugin-node-resolve), 它会允许加载在 `node_modules` 中的第三方模块。

2.  [`rollup-plugin-commonjs`](https://github.com/rollup/rollup-plugin-commonjs), 它会将 CommonJS 模块转换为 ES6,来为 Rollup 获得兼容。

用下面的命令安装这两个插件：

```
npm install --save-dev rollup-plugin-node-resolve rollup-plugin-commonjs
```

#### 更新 `rollup.config.js`.

接下来，在 Rollup 配置中 `import` 来添加插件：

```
// Rollup plugins
import babel from 'rollup-plugin-babel';
import eslint from 'rollup-plugin-eslint';
import resolve from 'rollup-plugin-node-resolve';
import commonjs from 'rollup-plugin-commonjs';

export default {
  entry: 'src/scripts/main.js',
  dest: 'build/js/main.min.js',
  format: 'iife',
  sourceMap: 'inline',
  plugins: [
    resolve({
      jsnext: true,
      main: true,
      browser: true,
    }),
    commonjs(),
    eslint({
      exclude: [
        'src/styles/**',
      ]
    }),
    babel({
      exclude: 'node_modules/**',
    }),
  ],
}; 
```

**注意：** `jsnext` 属性是指定[将 Node 包转换为 ES2015 模块](https://github.com/rollup/rollup/wiki/jsnext:main)。`main` 和 `browser` 属性将使插件决定将哪些文件应用到 bundle。

#### 检查控制台输出。

用 `./node_modules/.bin/rollup -c` 重新构建 bundle，然后在浏览器中再次检查输出：

好的！我们的日志现在正常展示了。

### 步骤5：添加插件来替代环境变量

环境变量能为我们的开发流程提供很大的帮助，我们可以通过它来执行像是关闭或开启日志、注入开发环境脚本等功能。

所以让我们来确保 Rollup 能够使用这一特性。

#### 在 `main.js` 中添加基础配置 `ENV`。

让我们添加一个环境变量来使我们的日志脚本只在非 `production` 环境下才会执行。在  `src/scripts/main.js` 中让我们改变 `log()` 初始化后的逻辑：

```
// Import a logger for easier debugging.
import debug from 'debug';
const log = debug('app:log');

// The logger should only be disabled if we’re not in production.
if (ENV !== 'production') {

  // Enable the logger.
  debug.enable('*');
  log('Logging is enabled!');
} else {
  debug.disable();
} 
```

然而，当我们重新构建我们的 bundle (`./node_modules/.bin/rollup -c`) 并检查浏览器，我们可以看到 `ENV` 报出了 `ReferenceError` 的错误。

这并不奇怪，因为我们并没有在所有位置定义它，我们试着运行 `ENV=production ./node_modules/.bin/rollup -c` ，然而它仍然没有正常工作。这是由于用这种方式设置环境变量只会对 Rollup 生效，对 Rollup 生成的 bundle 并不起作用。 

我们仍需要一个插件来将我们的环境变量作用到 bundle 中。

#### 安装模块

首先安装 [`rollup-plugin-replace`](https://github.com/rollup/rollup-plugin-replace),它本质上是一个用来查找和替换的工具。它可以做很多事，但对我们来说只需要找到目前的环境变量并用实际值来替代就可以了。（例如：在 bundle 中出现的所有 `ENV` 将被 `"production"` 替换）

```
npm install --save-dev rollup-plugin-replace
```

### 更新 `rollup.config.js`

在 `rollup.config.js` 中, 让我们 `import` 插件并添加到我们的插件列表里。

配置很简单：我们可以添加一个 `key:value` 的配对表，`key` 值是准备被替换的键值，而 `value` 是将要被替换的值。

```
// Rollup plugins
import babel from 'rollup-plugin-babel';
import eslint from 'rollup-plugin-eslint';
import resolve from 'rollup-plugin-node-resolve';
import commonjs from 'rollup-plugin-commonjs';
import replace from 'rollup-plugin-replace';

export default {
  entry: 'src/scripts/main.js',
  dest: 'build/js/main.min.js',
  format: 'iife',
  sourceMap: 'inline',
  plugins: [
    resolve({
      jsnext: true,
      main: true,
      browser: true,
    }),
    commonjs(),
    eslint({
      exclude: [
        'src/styles/**',
      ]
    }),
    babel({
      exclude: 'node_modules/**',
    }),
    replace({
      ENV: JSON.stringify(process.env.NODE_ENV || 'development'),
    }),
  ],
}; 
```

在我们的配置中，我们会找到每一个 `ENV` 并用 `process.env.NODE_ENV` 去替换 —— 在 Node 应用或是 `development` 中我们会用传统的方式去设置环境。我们会使用 `JSON.stringify` 来确保值是双引号的，不像 `ENV` 这样。

#### 检查结果

首先，重新构建 bundle 并在浏览器中检查。此时控制台应该像以前一样已经输出显示了 —— 这意味着我们的默认值被接受了。

为了查看真实的效果，让我们在 `production` 环境中运行下面代码

```
`NODE_ENV=production ./node_modules/.bin/rollup -c` 
```

**注意:** 在 Windows 中，使用 `SET NODE_ENV=production ./node_modules/.bin/rollup -c` 来避免在设置环境变量时产生错误。

当我们重新加载浏览器时，在控制台中就不再有输出内容了：

这样我们通过零代码修改，仅使用一个环境变量就禁用了日志记录。

### 步骤 6: 添加 UglifyJS 来最小化压缩我们生成脚本的体积。

本教程最后一步的任务是添加 UglifyJS 来最小化压缩 bundle。它可以通过移除注上释、缩短变量名、重整代码来**极大程度的**减少 bundle 的体积大小 —— 这样在一定程度降低了代码的可读性，但是在网络通信上变得更有效率。

#### 安装插件。

我们将会使用 [UglifyJS](https://github.com/mishoo/UglifyJS2/) 来压缩 bundle，这里我们通过 [`rollup-plugin-uglify`](https://github.com/TrySound/rollup-plugin-uglify) 来实现。

用下面的命令来安装：

```
npm install --save-dev rollup-plugin-uglify
```

#### 更新 `rollup.config.js`

接下来，让我们在 Rollup 配置中添加 Uglify 。然而，为了在开发中使代码更具可读性，让我们来设置只在生产环境中压缩混淆代码：

```
// Rollup plugins
import babel from 'rollup-plugin-babel';
import eslint from 'rollup-plugin-eslint';
import resolve from 'rollup-plugin-node-resolve';
import commonjs from 'rollup-plugin-commonjs';
import replace from 'rollup-plugin-replace';
import uglify from 'rollup-plugin-uglify';

export default {
  entry: 'src/scripts/main.js',
  dest: 'build/js/main.min.js',
  format: 'iife',
  sourceMap: 'inline',
  plugins: [
    resolve({
      jsnext: true,
      main: true,
      browser: true,
    }),
    commonjs(),
    eslint({
      exclude: [
        'src/styles/**',
      ]
    }),
    babel({
      exclude: 'node_modules/**',
    }),
    replace({
      ENV: JSON.stringify(process.env.NODE_ENV || 'development'),
    }),
    (process.env.NODE_ENV === 'production' && uglify()),
  ],
}; 
```

我们使用了[短路计算](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_Operators#Short-Circuit_Evaluation)策略，这是一种常见的（尽管[有些讨厌](http://stackoverflow.com/questions/5049006/using-s-short-circuiting-as-an-if-statement))捷径来在条件性的情况下设置一个值。[^short-circuit]

在我们的例子中，我们只会在 `NODE_ENV` 设置为 `production` 时加载 `uglify()`。

#### 检查最小化 bundle。

当配置保存后，让我们设置 `NODE_ENV` 为 production 并运行 Rollup:

```
`NODE_ENV=production ./node_modules/.bin/rollup -c` 
```

**注意:** 在 Windows 中, 使用 `SET NODE_ENV=production ./node_modules/.bin/rollup -c` 来避免在设置环境变量时产生错误。

这样的输出并不整洁，但它的体积**十分**小。

在之前，我们的 bundle 大小是 42KB。 在通过 UglifyJS 运行后，它减少到了 29KB —— 这样我们在没有额外付出的情况下就节省了超过 30% 的文件体积。

##  扩展阅读 

*  [The cost of small modules](https://nolanlawson.com/2016/08/15/the-cost-of-small-modules/) —— 是这篇文章让我对 Rollup 产生了兴趣，因为它展示了 Rollup 相比 webpack 和 Browserify 的一些显著的优势。

*   [Rollup’s getting started guide](http://rollupjs.org/guide/)

*   [Rollup’s CLI docs](https://github.com/rollup/rollup/wiki/Command-Line-Interface)

*   [A list of Rollup plugins](https://github.com/rollup/rollup/wiki/Plugins)

* * *

[^IIFE]: 这是一个相当复杂难以理解的概念, 所以如果不能完全明白也不要感到有压力。简而言之，我们希望我们的代码能在我们的作用域内，从而避免与其他的脚本产生冲突。这里 IIFE 是一个包含我们的代码并在它自身作用域产生的[闭包](http://skilldrick.co.uk/2011/04/closures-explained-with-javascript/)

[^size]: 重要的是需要记住，目前我们处理的是这样的一个小例子，这并不是很复杂就使文件大小大了一倍。在这时文件大小的比对是 3KB 和 8KB。

[^help]: 就像之前花费无数个小时去追踪一个 bug，最终发现原因仅仅是傻傻的变量名拼写错误一样。linter 对工作效率的提高十分明显，这并不是我们夸大其词。

[^short-circuit]: 例如，我们很常见到用这样的方式来指定默认值(例如: `var foo = maybeThisExists || 'default';`)。


## 有问题？有想法？发现了一个BUG?

本文中的代码被托管在 GitHub 上。你可以 [fork 项目](https://github.com/jlengstorf/learn-rollup) 来修改并测试它，[开启一个 issue](https://github.com/jlengstorf/learn-rollup/issues) 来报告 bug，或是 [创建一个 PR](https://github.com/jlengstorf/learn-rollup/compare)来提出改进或修改。