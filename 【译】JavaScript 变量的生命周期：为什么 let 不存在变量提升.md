title: 【译】JavaScript 变量的生命周期：为什么 let 不存在变量提升
date: 2016-08-03 13:21:20
categories: JavaScript
tags: [JavaScript,ECMAScript6]
---
>** 原文链接 : [JavaScript variables lifecycle: why let is not hoisted](https://rainsoft.io/variables-lifecycle-and-why-let-is-not-hoisted/)**
** 原文作者 : [Dmitri Pavlutin](https://rainsoft.io/author/dmitri-pavlutin/)**
** 译文出自 : [众成翻译](http://zcfy.cc/)**
** 译者 : [yangzj1992](http://qcyoung.com)**
** 校对者: [lisa](https://www.zhihu.com/people/ha-ha-qiu-52)**
** 首发于: [众成翻译](http://zcfy.cc/article/javascript-variables-lifecycle-why-let-is-not-hoisted-976.html)**

变量提升是一个将变量或者声明函数提升到作用域起始处的过程，通常指的是变量声明 `var` 和函数声明 `function fun() {...}` 

当 `let`（以及具备了和 `let` 相似声明行为的 `const` 和 `class`）等声明方式在 ES2015 中被引入后，许多的开发者包括我都使用了_变量提升_的定义来描述变量是如何被访问的。但经过对这个问题更多的搜索后，我十分惊讶的发现_变量提升_并不是可以用来准确描述 `let` 变量初始化和可用性的合适术语。

ES2015 为 `let` 提供了一个不同的改进机制。它要求了更严格的变量声明方式（你在定义变量前是无法访问它的）并且这也在结果上保证了更好的代码质量。

现在让我们一起深入了解关于这个过程的更多细节。

### 容易出错的 `var` 变量提升

有时我会在作用域下的任何位置上看到一个奇怪的变量声明 `var varname` 和函数声明 `function funName() {...}` 。

[Try in JS Bin](http://jsbin.com/fewiri/1/edit?js,console)

```
// var hoisting
num;     // => undefined
var num;
num = 10;
num;     // => 10
// function hoisting
getPi;   // => function getPi() {...}
getPi(); // => 3.14
function getPi() {
  return 3.14;
} 
```

变量 `num` 在它的声明语句 `var num` 之前就被访问了，所以它的值为 `undefined`

函数 `function getPi() {...}` 是定义在文件的末尾的。然而函数可以在它声明 `getPi()` 之前就被调用，因为它被提升到了作用域的顶部。

这就是典型的_变量提升_。

事实证明，在首次使用变量或函数后才声明变量或函数会很容易产生困惑。假设你正滚动查看一个大文件，然后发现了一个未声明的变量...你肯定会想它到底为什么在这里出现并且它是在哪定义的呢？

当然一个熟练的 JavaScript 开发者并不会这样编写代码。但在成千上万个 JavaScript Github 库中却可能存在着相当数量的这样的代码。

甚至在上面给出的代码示例中，我们也很难去明白代码中的声明流程。

我们应当自然地首先声明或是描述一个未知的术语。在这之后再对它进行使用。`let` 便是鼓励你遵循这种方法来设置变量。

### 深层内容: 变量的生命周期

当引擎使用变量时，它们的生命周期包含以下阶段： 

1.  **声明阶段** 这一阶段在作用域中注册了一个变量。

2.  **初始化阶段** 这一阶段分配了内存并在作用域中让内存与变量建立了一个绑定。在这一步变量会被自动初始化为 `undefined` 。

3.  **赋值阶段** 这一阶段为初始化变量分配具体的一个值。

一个变量在通过声明阶段时它还是处于 **未初始化的** 状态，这时它仍然还没有到达初始化阶段。

![Infographic](http://p0.qhimg.com/t01ea63a0e0c145b0f3.jpg)

注意，按照变量的生命周期过程，_声明阶段_与我们通常所说的_变量声明_是不同的术语。简单来讲，引擎处理变量声明需要经过完整的这 3 个阶段：声明阶段，初始化阶段和赋值阶段。

### `var` 变量的生命周期

稍微熟悉下这些生命周期阶段，现在让我们用它们来描述引擎是如何处理 `var` 变量的。

![Infographic](http://p0.qhimg.com/t014b5f030a2ed83760.jpg)

假设一个场景，当 JavaScript 遇到了一个函数作用域，其中包含了 `var variable` 的语句。则在任何语句执行之前，这个变量在作用域的开头就通过了_声明阶段_并马上来到了_初始化阶段_（步骤一）。 

同时 `var variable` 在函数作用域中的位置并不会影响它的声明和初始化阶段的进行。

在声明和初始化阶段之后，赋值阶段之前，变量的值便是 `undefined` 并已经可以被使用了。

在_赋值阶段_ `variable = 'value'` 语句使变量接受了它的初始化值（步骤二）。

这里的_变量提升_严格的说是指变量在函数作用域的_开始位置就完成了声明和初始化阶段_。在这里这两个阶段之间并没有任何的间隙。

让我们参考一个示例来研究。下面的代码创建了一个包含 `var` 语句的函数作用域：

[Try in JS Bin](http://jsbin.com/karuxe/3/edit?js,console)

```
function multiplyByTen(number) {
  console.log(ten); // => undefined
  var ten;
  ten = 10;
  console.log(ten); // => 10
  return number * ten;
}
multiplyByTen(4); // => 40 
```

当 JavaScript 开始执行 `multipleByTen(4)` 时进入了函数作用域中，变量 `ten` 在第一个语句之前就经过了声明和初始化阶段，所以当调用 `console.log(ten)` 时打印为 `undefined`。

当语句 `ten = 10` 为变量赋值了初始化值。在赋值后，语句 `console.log(ten)` 打印了正确的 `10` 值。

### 函数声明的生命周期

对于一个 _函数声明语句_ `function funName() {...}` 那就更简单了。

![Infographic](http://p0.qhimg.com/t0134485377a7130217.jpg)

_声明、初始化和赋值阶段_在封闭的函数作用域的开头便立刻进行（只有一步）。 `funName()` 可以在作用域中的任意位置被调用，这与其声明语句所在的位置无关（它甚至可以被放在程序的最底部）。

下面的代码是一个函数提升的演示：

[Try in JS Bin](http://jsbin.com/ximedo/2/edit?js,console)

```
function sumArray(array) {
  return array.reduce(sum);
  function sum(a, b) {
    return a + b;
  }
}
sumArray([5, 10, 8]); // => 23
```

当 JavaScript 执行 `sumArray([5, 10, 8])` 时，它便进入了 `sumArray` 的函数作用域。在作用域内，任何语句执行之前的瞬间，`sum` 就经过了所有的三个阶段：声明，初始化和赋值阶段。

这样 `array.reduce(sum)` 即使在它的声明语句 `function sum(a, b) {...}` 之前也可以使用 `sum`。

### `let` 变量的生命周期

`let` 变量的处理方式不同于 `var`。它的主要区分点在于声明和初始化阶段是分开的。

![Infographic](http://p0.qhimg.com/t0122846bdbec4513d7.jpg)

现在让我们研究这样一个场景，当解释器进入了一个包含 `let variable` 语句的块级作用域中。这个变量立即通过了_声明阶段_，并在作用域内注册了它的名称（步骤一）。

然后解释器继续逐行解析块语句。

这时如果你在这个阶段尝试访问 `variable`，JavaScript 将会抛出 `ReferenceError: variable is not defined`。因为这个变量的状态依然是_未初始化_的。

此时 `variable` 处于_临时死区_中。

当解释器到达语句 `let variable` 时，此时变量通过了初始化阶段（步骤二）。现在变量状态是_初始化的_并且访问它的值是 `undefined`。

同时变量在此时也离开了_临时死区_。

之后当到达赋值语句 `variable = 'value'` 时，变量通过了赋值阶段（步骤三）。

如果 JavaScript 遇到这样的语句 `let variable = 'value'` ，那么变量会在这一条语句中同时经过初始化和赋值阶段。

让我们继续看一个示例。这里 `let` 变量 `number` 被创建在了一个块级作用域中：

[Try in JS Bin](http://jsbin.com/qixoko/2/edit?js,console)

```
let condition = true;
if (condition) {
  // console.log(number); // => Throws ReferenceError
  let number;
  console.log(number); // => undefined
  number = 5;
  console.log(number); // => 5
}
```

当 JavaScript 进入 `if (condition) {...}` 块级作用域中，`number` 立即通过了声明阶段。

因为 `number` 尚未初始化并且处于临时死区，此时试图访问该变量会抛出 `ReferenceError: number is not defined`. 

之后语句 `let number` 使其得以初始化。现在变量可以被访问，但它的值是 `undefined`。

之后赋值语句 `number = 5` 当然也使变量经过了赋值阶段。

`const` 和 `class` 类型与 `let` 有着相同的生命周期，除了它们的赋值语句只会发生一次。

#### 为什么变量提升在 `let` 的生命周期中无效

如上所述，_变量提升_是变量的_耦合_声明并且在作用域的顶部完成初始化。

然而 `let` 生命周期中将声明和初始化阶段_解耦_。这一解耦使 `let` 的_变量提升_现象消失。

由于两个阶段之间的间隙创建了临时死区，在此时变量无法被访问。

这就像科幻的风格一样，在 `let` 生命周期中由于_变量提升_失效所以产生了临时死区。

### 结论

使用 `var` 自由的去声明变量很容易出现错误。

基于这一点，ES2015 引进了 `let`。它使用了一种改进的算法来声明变量并添加了块作用域。

因为声明和初始化阶段是解耦的，变量提升对于 `let` 变量（也包括 `const` 和 `class`)是无效的。在初始化之前，变量处于临时死区中并不可被访问。

为了保证平稳的变量声明，推荐这些技巧以供参考：

*   声明，初始化变量后再使用变量。这个流程才是正确并易于遵循的。

*   尽可能的减少变量数。你暴露的变量越少，你的代码则会变得更加模块化。

这就是今天所有的内容。我们在下一篇文章再见。