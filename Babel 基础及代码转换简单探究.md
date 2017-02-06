---
title: Babel 基础及代码转换简单探究
date: 2017-02-06 11:52:03
categories: JavaScript
tags: [Babel,ES6,JavaScript,编译]
---

本文基于 `babel-preset-es2015 : 6.18.0` ，`babel-core : 6.21.0`版本进行。

## 基础知识

[Babel](https://babeljs.io/) 是 JavaScript 编译器，更确切地说是源码到源码的编译器，通常也叫做“转换编译器(transpiler)”，它的工作流可以用下面一张图来表示，代码首先经由 babylon 解析成抽象语法树（AST），后经一些遍历和分析转换（主要过程），最后根据转换后的 AST 生成新的常规代码。

![](https://img.alicdn.com/tps/TB1nP2ONpXXXXb_XpXXXXXXXXXX-1958-812.png)

### 抽象语法树(AST)

在这其中，理解 AST 十分重要，为了让计算机能够更好地理解，所以需要将代码转换为 AST 。我们可以来看看下面这段代码被解析成 AST 后对应的结构图：

``` js
function abs(number) {
  if (number >= 0) {  // test
    return number;  // consequent
  } else {
    return -number; // alternate
  }
}
```

![](https://img.alicdn.com/tps/TB1NnDQNpXXXXXmXFXXXXXXXXXX-1015-756.png)

在 [Babylon 的 AST 规范文档中](https://github.com/babel/babylon/blob/master/ast/spec.md)对每个节点类型都做了详细的说明，你可以对照各个节点类型在这查找到所需要的信息。在这个例子中，我们主要关注函数声明里的内容， `IfStatement` 对应代码中的 `if...else` 区块的内容，我们先对条件（test）进行判断，这里是个简单的二元表达式，我们的分支也会从这个条件继续进行下去，consequent 代表条件值为 true 的分支，alternate 代表条件值为 false 的分支，最后两条分支各自在 ReturnStatement 节点进行返回。

了解 AST 各个节点的类型是理解 Babel 、编写插件的关键，AST 通常情况下都是比较复杂的，上述一段简单的函数定义也生成了比较大的 AST，对于一些复杂的程序，我们可以借助 [astexplorer](http://astexplorer.net/) 来帮我们分析 AST 的结构。 [这里](http://astexplorer.net/#/Z1exs6BWMq/36)是上述代码的一个示例链接。

### Babel 的处理步骤

Babel 的三个主要处理步骤分别是： `解析（parse）`，`转换（transform）`，`生成（generate）`。

#### 解析

解析步骤接收代码并输出 AST。 这个步骤分为两个阶段：[词法分析（Lexical Analysis）](https://zh.wikipedia.org/wiki/%E8%AF%8D%E6%B3%95%E5%88%86%E6%9E%90)(把字符串形式的代码转换为 令牌（tokens） 流) 和 [语法分析（Syntactic Analysis）](https://zh.wikipedia.org/wiki/%E8%AA%9E%E6%B3%95%E5%88%86%E6%9E%90%E5%99%A8)(令牌流转换成 AST 的形式)。

#### 转换

[程序转换(Program transformation)](https://en.wikipedia.org/wiki/Program_transformation)步骤接收 AST 并对其进行遍历，在此过程中对节点进行添加、更新及移除等操作。 这是 Babel 或是其他编译器中最复杂的过程 同时也是插件将要介入工作的部分。

#### 生成

[代码生成(Code generation)](https://en.wikipedia.org/wiki/Code_generation_(compiler))步骤把最终（经过一系列转换之后）的 AST 转换成字符串形式的代码，同时还会创建源码映射（source maps）。.

代码生成其实很简单：深度优先遍历(DFS) 整个 AST，然后构建可以表示转换后代码的字符串。

### 遍历

想要转换 AST 你需要进行递归的树形遍历。在进行节点遍历前还需要先了解 visitor 和 path 的概念，前者相当于从众多节点类型中选择开发者所需要的节点，后者相当于对节点之间的关系的访问。

#### visitor

Babel 使用 `babel-traverse` 进行树状的遍历，基本的树的遍历分为 DFS 和 BFS。AST 树的遍历使用 DFS，这里 Babel 提供了一个 visitor 对象来供我们获取 AST 里的具体节点，比如在我只想访问 if...else 生成的节点，我们可以在 visitor 里指定获取它所对应的节点：

``` js
  const visitor = {
    IfStatement() {
      console.log('get if');
    }
  };
```

之所以使用这样的术语是因为有一个[访问者模式（visitor）](https://zh.wikipedia.org/wiki/%E8%AE%BF%E9%97%AE%E8%80%85%E6%A8%A1%E5%BC%8F)的概念。对于每个结点，有向下遍历进入结点（enter)和向上退出结点(exit)两个时刻（对应递归调用的入栈、出栈），我们可以在此时来访问结点。

``` js
const visitor = {
  IfStatement: {
    enter() {},
    exit() {}
  }
}
```

#### path

visitor 模式中我们对节点的访问实际上是对节点路径的访问，在这个模式中我们一般把path 当作参数传入节点选择器中。path 表示两个节点之间的连接，通过这个对象我们可以访问到节点、父节点以及进行一系列跟节点操作相关的方法（类似 DOM 的操作）。

``` js
var babel = require('babel-core');
var t = require('babel-types');

const code = `d = a + b + c`;

const visitor = {
  Identifier(path) {
    console.log(path.node.name); // d a b c
  }
};

const result = babel.transform(code, {
  plugins: [{
    visitor: visitor
  }]
});
```

以上面的例子，我们有一个 `FunctionDeclaration` 类型如下。它有几个属性：`id`，`params`，和 `body`，每一个都有一些内嵌节点。我们依次遍历每个节点即可。Babel 的转换步骤就是循环这样的遍历过程。

``` js
  {
    type: "FunctionDeclaration",
    id: {
      type: "Identifier",
      name: "abs"
    },
    params: [{
      type: "Identifier",
      name: "number"
    }],
    body: {
      type: "BlockStatement",
      body: [{
        type: "IfStatement",
        test: {
          type: "BinaryExpression",
          left: {
            type: "Identifier",
            name: "number"
          },
          operator: ">=",
          right: {
            type: "Literal",
            value: "0"
          }
        },
        consequent:{
          type: "BlockStatement",
          body: [{
            type: "ReturnStatement",
            argument: {
              type: "Identifier",
              name: "number"
            }
          }]
        },
        alternate: {
          type: "BlockStatement",
          body: [{
            type: "ReturnStatement",
            argument: {
              type: "UnaryExpression",
              opertaor: "-",
              argument: {
                type: "Identifier",
                name: "number"
              },
              name: "number"
            }
          }]
        }
      }]
    }
  }
```

### 对应代码

我们统称的 babel 源码实质上对应于 npm 上的多个包，具体可以参考对应的说明文件，
主要包括 [核心代码(babel-core)](https://www.npmjs.com/package/babel-core)、工具包（[babel-cli](https://www.npmjs.com/package/babel-cli) 等）、Preset（[babel-preset-es2015](https://www.npmjs.com/package/babel-preset-es2015) 等），另外还有一系列 helper 包。

#### babel-core 核心包

babel 中最核心的是 babel-core 这个包，它向外暴露出 `babel.transform` 接口，供类似 `babel.transform(code, options);` 的方式调用，而核心代码位于 `transformation/pipeline.js` 文件中，所有信息都会挂在到 `transformation/file` 所暴露出来的数据结构 `File` 上。

`File` 维护的是一个文件的所有信息，包括 babel 处理所用的插件等信息。babel 的繁荣与其强大的插件管理机制是密不可分的，而插件主要由 `pluginPasses` 和 `pluginVisitors` 来维护。

为了保证在遍历路径的时候能够快速访问对应的插件处理方法，babel 对 `pluginVisitors` 做了一定的预处理，将所有同类型 Identifier 的处理流程合并到了一起。具体用代码的角度来看，可以简化成这样：

``` js 
const PluginA = {
  Identifier() {},
  FunctionDeclaration() {}
}
const PluginB = {
  BinaryExpression() {},
  FunctionDeclaration() {}
}
// 进行处理后得到(源码可见 babel-traverse/lib/visitors.js 中的 function merge())
let rootVisitor = {
  Identifier: [PluginA.Identifier],
  BinaryExpression: [PluginB.Identifier],
  FunctionDeclaration: [PluginA.FunctionDeclaration, PluginB.FunctionDeclaration]
}
```

再看一下后续的内部转码流程，其最外层 pipeline 中的代码很简短：

``` js 
file.wrap(code, function () {
  file.addCode(code);
  file.parseCode(code); // 去除顶部的 #!/usr/bin/env node 信息，之后使用 babylon 解析出 ast，使用 babel-traverse 进行遍历
  return file.transform();
});
```

其过程分解用语言描述的话，对应上文步骤如下：

1. 使用 babylon 解析器对输入的源代码字符串进行解析并生成初始 AST（`File.prototype.parse`）
2. `set AST` 过程：利用 `babel-traverse` 对 AST 进行遍历，并解析出整个树的 path，通过挂载的 `metadataVisitor` 读取对应的元信息。
3. `transform` 过程：遍历 AST 树并应用各 transformers(plugin) 生成转换后的 AST 树
4. 利用 babel-generator 将 AST 树生成为转码后的代码字符串。
> 注：以上面的代码片断为例，为了详细了解到整个编译过程，可以使用 `DEBUG=babel node main.js` 运行代码，这样就可以看到整个过程中的输出日志了。

## 转换结果

由于 Babel 默认只转换新的 JavaScript 语法，而不转换新的 API，比如 Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise 等全局对象，以及一些定义在全局对象上的方法（比如 `Object.assign`）都不会转码。
Babel 默认不转码的 API 非常多，具体可以查看[详细清单](
https://github.com/babel/babel/blob/master/packages/babel-plugin-transform-runtime/src/definitions.js)

这里参考 进行补充

### babel-preset-2015

目前版本下插件列表如下：

- `babel-plugin-check-es2015-constants` => 验证 es2015 常量
- `babel-plugin-transform-es2015-arrow-functions` => 箭头函数
- `babel-plugin-transform-es2015-block-scoped-functions` => 函数块级作用域
- `babel-plugin-transform-es2015-block-scoping` => let 和 const 块级作用域
- `babel-plugin-transform-es2015-classes` => class类
- `babel-plugin-transform-es2015-computed-properties` => 动态计算属性，如 `var x = 1;var obj = {[x]: 3};`
- `babel-plugin-transform-es2015-destructuring` => 解构赋值
- `babel-plugin-transform-es2015-duplicate-keys` => 对象中重复的 key 转换成计算属性
- `babel-plugin-transform-es2015-for-of` => 对象 `for ... of` 遍历
- `babel-plugin-transform-es2015-function-name` => 函数 name 属性
- `babel-plugin-transform-es2015-literals` => unicode 字符串和数字字面值
- `babel-plugin-transform-es2015-modules-amd` => amd 模块转换
- `babel-plugin-transform-es2015-modules-commonjs` => commonjs 模块转换
- `babel-plugin-transform-es2015-modules-systemjs` => systemjs 模块转换
- `babel-plugin-transform-es2015-modules-umd` => umd 模块转换
- `babel-plugin-transform-es2015-object-super` => super 方法调用 prototype 
- `babel-plugin-transform-es2015-parameters` => 函数参数默认值及扩展运算符
- `babel-plugin-transform-es2015-shorthand-properties` => 对象属性的快捷定义，如obj = { x, y }
- `babel-plugin-transform-es2015-spread` => 对象扩展运算符属性，如 `...foobar`
- `babel-plugin-transform-es2015-sticky-regex` => 粘连修饰符 y.
- `babel-plugin-transform-es2015-template-literals` => es2015 模板
- `babel-plugin-transform-es2015-typeof-symbol` => symbol 特性
- `babel-plugin-transform-es2015-unicode-regex` => unicode 正则
- `babel-plugin-transform-regenerator` => generator特性

### var, const and let 

#### var_const_let.js

首先基础的，转换前：

``` js
var a = 1;
let b = 2;
const c = 3;
```

转换后：

``` js
var a = 1;
var b = 2;
var c = 3;
```

#### let.js

let的块级作用域怎么体现呢？转换前：

``` js
let a1 = 1;
let a2 = 6;
{
    let a1 = 2;
    let a2 = 5;
    {
        let a1 = 4;
        let a2 = 5;
    }
}
a1 = 3;
```

转换后：

``` js
var a1 = 1;
var a2 = 6;
{
    var _a = 2;
    var _a2 = 5;
    {
        var _a3 = 4;
        var _a4 = 5;
    }
}
a1 = 3;
```

可见这样的例子实质就是改变一下变量名，使之与外层不同。

#### let_for.js

那么看一下经典的 let for 场景，这里大家都知道如果 let 换成 var ，那么输出将会是 10，那么这样 babel 怎么处理呢？转换前：

``` js
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
```

转换后：

``` js
var a = [];

var _loop = function _loop(i) {
  a[i] = function () {
    console.log(i);
  };
};

for (var i = 0; i < 10; i++) {
  _loop(i);
}
a[6](); // 6
```

可见这里用了闭包做了处理。经典的 for 循环闭包处理方式。

### 解构赋值

#### destructuring.js

对于普通的解构赋值，转换前：

``` js
let [foo, [[bar], baz]] = [1, [[2], 3]];

let [ , , third] = ["foo", "bar", "baz"];
// third = "baz"

let [x, , y] = [1, 2, 3];
// x = 1 y = 3

let [head, ...tail] = [1, 2, 3, 4];
// head = 1 tail = [2, 3, 4]

let [i, j, ...k] = ['a'];
// i = "a" j = undefined k = []

let [m, n] = [1];
// n = undefined

let [a, [b], d] = [1, [2, 3], 4];
// a = 1 b = 2 d = 4

const [a, b, c, d, e] = 'hello';

let {length : len} = 'hello';

```

转换后：

``` js
var _slicedToArray = function () { function sliceIterator(arr, i) { var _arr = []; var _n = true; var _d = false; var _e = undefined; try { for (var _i = arr[Symbol.iterator](), _s; !(_n = (_s = _i.next()).done); _n = true) { _arr.push(_s.value); if (i && _arr.length === i) break; } } catch (err) { _d = true; _e = err; } finally { try { if (!_n && _i["return"]) _i["return"](); } finally { if (_d) throw _e; } } return _arr; } return function (arr, i) { if (Array.isArray(arr)) { return arr; } else if (Symbol.iterator in Object(arr)) { return sliceIterator(arr, i); } else { throw new TypeError("Invalid attempt to destructure non-iterable instance"); } }; }();

var foo = 1,
    bar = 2,
    baz = 3;
    
var _ref = ["foo", "bar", "baz"],
    third = _ref[2];

var _ref2 = [1, 2, 3],
    x = _ref2[0],
    y = _ref2[2];

var head = 1,
    tail = [2, 3, 4];

var _ref3 = ['a'],
    i = _ref3[0],
    j = _ref3[1],
    k = _ref3.slice(2);

var _ref4 = [1],
    m = _ref4[0],
    n = _ref4[1];

var a = 1,
    _ref5 = [2, 3],
    b = _ref5[0],
    d = 4;

var _hello = 'hello',
    _hello2 = _slicedToArray(_hello, 5),
    x1 = _hello2[0],
    x2 = _hello2[1],
    x3 = _hello2[2],
    x4 = _hello2[3],
    x5 = _hello2[4];

var _hello3 = 'hello',
    len = _hello3.length;
```

可以看到，这里 babel 就是很正常的采用一一赋值的方式进行的，对于匿名数组等情况，babel 会帮你先定义一个变量存放这个数组，然后再对需要赋值的变量进行赋值。

#### destructuring_object.js

还有一种对象深层次的解构赋值，转换前：

``` js
var obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};

var { p: [x, { y }] } = obj;
// x = "Hello" y = "World"
```

转换后：

``` js
var _slicedToArray = (function() {
    function sliceIterator(arr, i) {
        var _arr = [];
        var _n = true;
        var _d = false;
        var _e = undefined;
        try {
           // 用 Symbol.iterator 造了一个可遍历对象，然后进去遍历。
            for (var _i = arr[Symbol.iterator](), _s; !(_n = (_s = _i.next()).done); _n = true) {
                _arr.push(_s.value);
                if (i && _arr.length === i) break;
            }
        } catch (err) {
            _d = true;
            _e = err;
        } finally {
            try {
                if (!_n && _i["return"]) _i["return"]();
            } finally {
                if (_d) throw _e;
            }
        }
        return _arr;
    }
    return function(arr, i) {
        if (Array.isArray(arr)) {
            return arr;
        } else if (Symbol.iterator in Object(arr)) {
            return sliceIterator(arr, i);
        } else {
            throw new TypeError("Invalid attempt to destructure non-iterable instance");
        }
    };
})();

var obj = {
  p: ['Hello', { y: 'World' }]
};

var _obj$p = _slicedToArray(obj.p, 2),
    x = _obj$p[0],
    y = _obj$p[1].y;
// x = "Hello" y = "World"
```

这里和上一个示例中对字符串进行赋值解构时 babel 都在代码顶部生产了一个公共的代码 `_slicedToArray`。它的作用主要是对对象里的属性转换成数组形式，并依靠 i 变量来取出我们真正需要的值，方便解构赋值的进行。

#### destructuring_function.js

转换前：

``` js
function move({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]
```

转换后：

``` js
function move() {
  var _ref = arguments.length > 0 && arguments[0] !== undefined ? arguments[0] : { x: 0, y: 0 },
      x = _ref.x,
      y = _ref.y;

  return [x, y];
}

move({ x: 3, y: 8 }); // [3, 8]
move({ x: 3 }); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]
```

### 函数的扩展

转换前：

``` js
// default parameter values
function log(x, y = 'World') {
  console.log(x, y);
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello

// rest parameter
function push(array, ...items) {
  items.forEach(function(item) {
    array.push(item);
    console.log(item);
  });
}

var a = [];
push(a, 1, 2, 3)

// arrow function
var obj = {
    prop: 1,
    func: function() {
        var _this = this;
 
        var innerFunc = () => {
            this.prop = 1;
        };
 
        var innerFunc1 = function() {
            this.prop = 1;
        };
    },
 
};
```

转换后：

``` js
// default parameter values
function log(x) {
    var y = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : 'World';

    console.log(x, y);
}

log('Hello'); // Hello World
log('Hello', 'China'); // Hello China
log('Hello', ''); // Hello

// rest parameter
function push(array) {
    for (var _len = arguments.length, items = Array(_len > 1 ? _len - 1 : 0), _key = 1; _key < _len; _key++) {
        items[_key - 1] = arguments[_key];
    }

    items.forEach(function (item) {
        array.push(item);
        console.log(item);
    });
}

var a = [];
push(a, 1, 2, 3);

// arrow function
var obj = {
    prop: 1,
    func: function func() {
        var _this2 = this;

        var _this = this;

        var innerFunc = function innerFunc() {
            _this2.prop = 1;
        };

        var innerFunc1 = function innerFunc1() {
            this.prop = 1;
        };
    }

};
```

这里通过默认参数的转换方式并结合上例中解构赋值函数的例子就看的很明白了，主要使用 arguments 来做处理。

而 rest 参数则同样依靠 arguments 来遍历处理既定参数之后的所有参数。

而箭头函数主要是省了写 function 的代码，同时能够直接用使外层的 this 而不用担心 context 切换的问题。以前我们一般都要在外层多写一个 _this/self 直向 this。babel 的转换方法跟大家平时所了解的基本一致。

### 对象的扩展

转换前：

``` js
const prop2 = "PROP2";
var obj = {
    ['prop']: 1,
    ['func']: function() {
        console.log('func');
    },
        [prop2]: 3
};

var obj = {
    toString() {
     // Super calls
     return "d " + super.toString();
    },
};
```

转换后：

``` js
var _get = function get(object, property, receiver) {
    // 如果 prototype 为空，则往 Function 的 prototype 上寻找
    if (object === null) object = Function.prototype;
    var desc = Object.getOwnPropertyDescriptor(object, property);
    if (desc === undefined) {
        var parent = Object.getPrototypeOf(object);
        // 如果在本层 prototype 找不到，再往更深层的 prototype 上找
        if (parent === null) {
            return undefined;
        } else {
            return get(parent, property, receiver);
        }
    // 如果是属性，则直接返回
    } else if ("value" in desc) {
        return desc.value;
    // 如果是方法，则用 call 来调用，receiver 是调用的对象 
    } else {
        var getter = desc.get;// getOwnPropertyDescriptor 返回的 getter 方法
        if (getter === undefined) {
            return undefined;
        }
        return getter.call(receiver);
    }
};

var _obj, _obj2;

function _defineProperty(obj, key, value) {
    if (key in obj) {
        Object.defineProperty(obj, key, {
            value: value,
            enumerable: true,
            configurable: true,
            writable: true
        });
    } else {
        obj[key] = value;
    }
    return obj;
}

var prop2 = "PROP2";
var obj = (_obj = {}, _defineProperty(_obj, 'prop', 1), _defineProperty(_obj, 'func', function func() {
    console.log('func');
}), _defineProperty(_obj, prop2, 3), _obj);

var obj = _obj2 = {
    toString: function toString() {
        // Super calls
        return "d " + _get(_obj2.__proto__ || Object.getPrototypeOf(_obj2), 'toString', this).call(this);
    }
};
```

对于对象中用中括号解释属性的功能，babel 新增了一个 `_defineProperty` 函数，给新建的 `_obj = {}`进行属性定义。除此之外使用小括号包住一系列从左到右的运算使整个定义更简洁。

对于对象字面量中使用 `super` 去调用 prototype，babel 通过 `_get` 方法来在 prototype 链上寻找方法/属性。

### 字符串模板

转换前：

``` js
var a = 5;
var b = 10;
 
function tag(strings, ...values) {
  console.log(strings[0]); // "Hello "
  console.log(strings[1]); // " world "
  console.log(values[0]);  // 15
  console.log(values[1]);  // 50
}
 
tag`Hello ${ a + b } world ${ a * b }`;
```

转换后：

``` js
var _templateObject = _taggedTemplateLiteral(["Hello ", " world ", ""], ["Hello ", " world ", ""]);

// _templateObject = ["Hello ", " world ", "" , raw: Array[3]]

// 给传入的 object 定义了 strings 和 raw 两个不可变的属性。
function _taggedTemplateLiteral(strings, raw) {
  return Object.freeze(Object.defineProperties(strings, {
    raw: {
      value: Object.freeze(raw)
    }
  }));
}

var a = 5;
var b = 10;

function tag(strings) {
  // strings = ["Hello ", " world ", "", raw: Array[3]]    arguments = [Array[3], 15, 50]
  console.log(strings[0]); // "Hello "
  console.log(strings[1]); // " world "
  console.log(arguments.length <= 1 ? undefined : arguments[1]); // 15
  console.log(arguments.length <= 2 ? undefined : arguments[2]); // 50
}

tag(_templateObject, a + b, a * b);

es6 的这种新特性给模板处理赋予更强大的功能，一改以往对模板进行各种 replace 的处理办法，用一个统一的 handler 去处理。babel 的转换主要是添加了 2 个属性，通过捕获传参和 arguments 变量来获取具体的值。
```

### 模块化与类

#### 类 Class

js 实现 oo 一直是非常热门的话题。从最原始时代的手动维护构造函数来调用父类构造函数,到后来封装好函数进行 extend 继承，再到 babel 出现之后可以像其它面向对象的语言一样直接写 class。es2015 的类方案仍然算是过渡方案，它所支持的特性仍然没有涵盖类的所有特性。目前主要支持的有：

- constructor
- static 方法
- get 方法
- set 方法
- 类继承
- super 调用父类方法。


转换前：

``` js
class Animal {
  constructor(name, type) {
    this.name = name;
    this.type = type;
  }
  walk() {
    console.log('walk');
  }
  run() {
    console.log('run')
  }
  static getType() {
    return this.type;
  }
  get getName() {
    return this.name;
  }
  set setName(name) {
    this.name = name;
  }
}

class Dog extends Animal {
  constructor(name, type) {
    super(name, type);
  }
  get getName() {
    return super.getName();
  }
}
```

转换后：

``` js
// 与上同
var _get = function get(object, property, receiver) { if (object === null) object = Function.prototype; var desc = Object.getOwnPropertyDescriptor(object, property); if (desc === undefined) { var parent = Object.getPrototypeOf(object); if (parent === null) { return undefined; } else { return get(parent, property, receiver); } } else if ("value" in desc) { return desc.value; } else { var getter = desc.get; if (getter === undefined) { return undefined; } return getter.call(receiver); } };

// _creatClass 用于创建类及其对应的方法
var _createClass = function () {
  function defineProperties(target, props) {
    for (var i = 0; i < props.length; i++) {
      var descriptor = props[i];
      // es6 规范要求类方法为 non-enumerable
      descriptor.enumerable = descriptor.enumerable || false;
      descriptor.configurable = true;
      // 对于 setter 和 getter 方法，writable 为 false
      if ("value" in descriptor) descriptor.writable = true;
      Object.defineProperty(target, descriptor.key, descriptor);
    }
  }
  return function (Constructor, protoProps, staticProps) {
    // 非静态方法定义在原型链上
    if (protoProps) defineProperties(Constructor.prototype, protoProps);
    // 静态方法直接定义在 constructor 函数上
    if (staticProps) defineProperties(Constructor, staticProps);
    return Constructor;
  };
}();

// 子类实现 constructor
// babel 会强制子类在 constructor 中使用 super，否则编译会报错
function _possibleConstructorReturn(self, call) {
  if (!self) {
    throw new ReferenceError("this hasn't been initialised - super() hasn't been called");
  }
  // 若call是函数/对象则返回
  return call && (typeof call === "object" || typeof call === "function") ? call : self;
}

// 子类继承父类
function _inherits(subClass, superClass) {
  // 父类一定要是 function 类型
  if (typeof superClass !== "function" && superClass !== null) {
    throw new TypeError("Super expression must either be null or a function, not " + typeof superClass);
  }
  // 使原型链 subClass.prototype.__proto__ 指向父类 superClass，同时保证 constructor 是 subClass 自己
  subClass.prototype = Object.create(superClass && superClass.prototype, {
    constructor: {
      value: subClass,
      enumerable: false,
      writable: true,
      configurable: true
    }
  });
  // 保证 subClass.__proto__ 指向父类 superClass
  if (superClass) Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;
}

// 检测 constructor 正确与否
function _classCallCheck(instance, Constructor) {
  if (!(instance instanceof Constructor)) {
    throw new TypeError("Cannot call a class as a function");
  }
}

var Animal = function () {
  function Animal(name, type) {
    // 此处是 constructor 的实现
    _classCallCheck(this, Animal);

    this.name = name;
    this.type = type;
  }

  _createClass(Animal, [{
    key: 'walk',
    value: function walk() {
      console.log('walk');
    }
  }, {
    key: 'run',
    value: function run() {
      console.log('run');
    }
  }, {
    key: 'getName',
    get: function get() {
      return this.name;
    }
  }, {
    key: 'setName',
    set: function set(name) {
      this.name = name;
    }
  }], [{
    key: 'getType',
    value: function getType() {
      return this.type;
    }
  }]);

  return Animal;
}();

var Dog = function (_Animal) {
  _inherits(Dog, _Animal);

  function Dog(name, type) {
    _classCallCheck(this, Dog);

    return _possibleConstructorReturn(this, (Dog.__proto__ || Object.getPrototypeOf(Dog)).call(this, name, type));
  }

  _createClass(Dog, [{
    key: 'getName',
    get: function get() {
      return _get(Dog.prototype.__proto__ || Object.getPrototypeOf(Dog.prototype), 'getName', this).call(this);
    }
  }]);

  return Dog;
}(Animal);
```

#### 模块化

示例：

``` js
// module.js
import { Animal as Ani, catwalk } from "./t1";
import * as All from "./t2";
 
class Cat extends Ani {
  constructor() {
    super();
  }
}
 
class Dog extends Ani {
  constructor() {
    super();
  }
}
```

``` js
// t1.js
export class Animal {
  constructor() {
  }
}
 
export function catwal() {
  console.log('cat walk');
};
```

``` js
// t2.js
export class Person {
  constructor() {
  }
}
 
export class Plane {
  constructor() {
  }
}
```

转换后：

``` js
// t1.js 模块

Object.defineProperty(exports, "__esModule", {
  value: true
});
exports.catwal = catwal;

var Animal = exports.Animal = function Animal() {
  _classCallCheck(this, Animal);
};

function catwal() {
  console.log('cat walk');
};

// t2.js 模块

Object.defineProperty(exports, "__esModule", {
  value: true
});

var Person = exports.Person = function Person() {
  _classCallCheck(this, Person);
};

var Plane = exports.Plane = function Plane() {
  _classCallCheck(this, Plane);
};

// module.js

var _t = require("./t1");

var _t2 = require("./t2"); // 返回的都是exports上返回的对象属性

var All = _interopRequireWildcard(_t2);

function _interopRequireWildcard(obj) {
  // 发现是babel编译的， 直接返回
  if (obj && obj.__esModule) {
    return obj;
  // 非 babel 编译， 猜测可能是第三方模块，为了不报错，让 default 指向它自己
  } else {
    var newObj = {};
    if (obj != null) {
      for (var key in obj) {
        if (Object.prototype.hasOwnProperty.call(obj, key)) newObj[key] = obj[key];
      }
    }
    newObj.default = obj;
    return newObj;
  }
}

var Cat = function (_Ani) {
  _inherits(Cat, _Ani);

  function Cat() {
    _classCallCheck(this, Cat);

    return _possibleConstructorReturn(this, (Cat.__proto__ || Object.getPrototypeOf(Cat)).call(this));
  }

  return Cat;
}(_t.Animal);

var Dog = function (_Ani2) {
  _inherits(Dog, _Ani2);

  function Dog() {
    _classCallCheck(this, Dog);

    return _possibleConstructorReturn(this, (Dog.__proto__ || Object.getPrototypeOf(Dog)).call(this));
  }

  return Dog;
}(_t.Animal);
```

es6 的模块加载是属于多对象多加载，而 commonjs 则属于单对象单加载。babel 需要做一些手脚才能将 es6 的模块写法写成 commonjs 的写法。主要是通过定义 `__esModule` 这个属性来判断这个模块是否经过 babel 的编译。然后通过 `_interopRequireWildcard` 对各个模块的引用进行相应的处理。


## 参考资料

[理解 Babel 插件](http://taobaofed.org/blog/2016/09/29/babel-plugins/)

[babel到底将代码转换成什么鸟样？](https://github.com/lcxfs1991/blog/issues/9)

[Babel 插件手册](https://github.com/thejameskyle/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md)