JavaScript 的怪癖 6：变量的作用范围
---

原文：[JavaScript quirk 6: the scope of variables](http://www.2ality.com/2013/05/quirk-variable-scope.html)

译文：[JavaScript 的怪癖 6：变量的作用范围](#)

译者：[justjavac](http://www.justjavac.com)

----------------------------------------------------

此文是 [javascript 的 12 个怪癖（quirks）](http://justjavac.com/javascript/2013/04/08/12-javascript-quirks.html) 系列的第六篇。

在大多数编程语言中，变量的生命周期是“定义此变量的块（block）”。
但是在 JavaScript 中，变量的作用域却和**函数**息息相关，而不是大括号：

```javascript
function func(x) {
    console.log(tmp); // undefined
    if (x < 0) {
        var tmp = 100 - x;  // (*)
        ...
    }
}
```

（译注：很多程序员会觉得 `tmp` 变量的作用于是 `if` 块，其实不然，javascript 根本没有块作用域。）

上述代码引发的行为是：函数内部，在（*）处声明的变量 `tmp` 被移动到了函数的开头（赋值语句依然保留在原处）。
也就是说，实际上此段代码在 JavaScript 引擎中的运行时，看起来像这样： 

```javascript
function func(x) {
    var tmp;
    console.log(tmp); // undefined
    if (x < 0) {
        tmp = 100 - x;
        ...
    }
}
```

但是，有一招可以把一个变量限制在一个块作用域，它被称为立即函数表达式（IIFE，发音为“iffy”）。

**立即函数表达式**：Immediately Invoked Function Expression。

下面，我们使用一个 IIFE，将 `tmp` 的作用域限制在包含它的 `if` 语句块中。 

```javascript
function func(x) {
    console.log(tmp); // ReferenceError: tmp is not defined
    if (x < 0) {
        (function () {  // open IIFE
            var tmp = 100 - x;
            ...
        }());  // close IIFE
    }
}
```

我们在内部块的外面，写了一个函数，创建了一个新的作用域。
（译注：javascript 没有块作用域，OMG！。只有函数作用域，因此我们必须使用一个匿名函数，而且是立即执行的匿名函数创建了一个新的作用域。）
然后我们立即执行此函数。
`tmp` 仅仅存在于 IIFE 中。
需要注意的是围绕此 IIFE 的小括号是必须的。

如果没有环绕 IIFE 的小括号，函数成了：

```javascript
function () {
    var tmp = 100 - x;
}();
```

执行结果：

    SyntaxError: Unexpected token (

（why？）

作者写道，

> They lead to the function being interpreted as an expression, 
> which is the **only form** in which it can be be immediately invoked.

> 我们需要在 IIFE 的开始和结束的地方，写上小括号，把函数解析成一个**表达式**。
> 这是把函数变成立即调用的**唯一**形式。

到底是不是唯一形式呢？

一、

```javascript
(function () {
    var tmp = 100 - x;
}());
```

二、

```javascript
(function () {
    var tmp = 100 - x;
})();
```

（and why？）

留下了几个疑问，原作者没有写明，由于篇幅关系，我会在随后的博文中解释。

不过关于**表达式**的疑问可以去看我写的『代码之谜』系列之[语句与表达式](http://justjavac.com/codepuzzle/2012/10/28/codepuzzle-expression-and-statement.html)。
最后一个疑问可以读一下 [命名函数表达式探秘](http://justjavac.com/named-function-expressions-demystified.html)
