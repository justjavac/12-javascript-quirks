JavaScript 的怪癖 7：变量闭包之后的无意识共享
---

原文：[JavaScript quirk 7: inadvertent sharing of variables via closures](http://www.2ality.com/2013/05/quirk-closures.html)

译者：[zhmengqing](http://me.9tech.cn/index.php/homepage/main/96622)

----------------------------------------------------

此文是 [javascript 的 12 个怪癖（quirks）](https://github.com/justjavac/12-javascript-quirks) 系列的第七篇。

闭包是 JavaScript 一个强大的特性：当函数离开了创建它的位置，仍然可以获取到该位置上存在的所有变量。
本文主要解释闭包的工作方式以及我们为什么要对于变量的无意识共享多加小心。

## 1、闭包

我们从一个闭包的例子开始吧：

```javascript
function incrementorFactory(start, step) {
    return function () {  // (*)
        start += step;
        return start;
    }
}
```

下面调用 `incrementorFactory`：

```
> var inc = incrementorFactory(20, 2);
> inc()
22
> inc()
24
```

在运行阶段，内部函数(*)能获取到外部函数 `incrementorFactory` 的变量 `start` 与 `step`，而且 `incrementorFactory` 不只返回函数，
也连带返回了变量 `start` 与 `step`。
存储这两个变量的数据结构叫做 `environment`，`environment` 与 `object` 非常相似——它将键名映射到键值。
以上返回的函数包含了 `environment` 的引用，它在父级即外部的 `environment` 时就已经激活。
组合函数 + `environment` 就叫做闭包。
这名称来源于当 `environment` “关闭”函数时：它为变量提供了可声明在函数外的值（这就是所谓的自由变量）。

当函数被请求，就会为它的参数和局部变量创建一个新的 `environment`。
所以总会有一连串的`environment`：

* `f` 的 `environment`
* `f` 的外部 `environment`
* `f` 外的 `environment` 外部的 `environment`
* ......
* 全局变量的 `environment`（全局 `environment`）

以上是从 `f` 的 `environment`开始，完全搜索 `environment` 链查看的所有变量值。

## 2、怪癖：无意识共享

闭包并不是在特定的时间点获得快照，它是获取动态的变量，以下是这个问题的例子：

```javascript
var result = [];
for (var i=0; i < 5; i++) {
    result.push(function () { return i });  // (*)
}
console.log(result[3]()); // 5 (not 3)
```

当函数在这里(*)创建的时候，变量 `i` 有一个确定的值，你可能会觉得那个函数返回的会一直是那个值。
相反，它与动态的i是一直关联着的，就是说所有 `result` 数组中的函数都是通过它们的外部 `environment` 关联同一个 `i`，当循环结束时，`i` 的值就是 `5`。

一种可行的解决方案就是通过一个返回值(Immediately Invoked Function Expression)[1]来复制 `i` 的当前值：

```javascript
for (var i=0; i < 5; i++) {
   (function (i2) {  // snaphot of i
       result.push(function () { return i2 });
   }(i));
}
```

你也可以用 `bind()` 函数，也有相似的效果：

```javascript
for (var i=0; i < 5; i++) {
    result.push(function (i2) { return i2 }.bind(null, i));
}
```

用 `forEach` 和 `Underscore.js` 库中的 `range()` 函数也可以办到：

```javascript
_.range(5).forEach(function (i) {
    result.push(function () { return i });
});
```

以上的代码都可行，因为每次请求参数时，`forEach` 都创建了一个新的i变量。

### 2.1 一个实际应用的例子

下面我们用一个更加实用的例子来总结下。
两天前，我做了一 个Connect Four 游戏的 UI 来作为 DOM 的示例，它包含了以下的代码片段，
添加了事件侦听来连接到游戏板的行列上。

```javascript
for(var col=0; col < board4.DIM_X; col++) {
    document.getElementById('columnClick'+col)
            .addEventListener('click', function (col) {
                currentState.columnClick(col);
                event.preventDefault();
            }.bind(null, col));
}
```

另一种方式是用 CSS 类来代替 ID，重写以上代码：

```javascript
Array.prototype.forEach.call(
  document.getElementsByClassName('columnClick'),
  function (elem, col) {  // (*)
      elem.addEventListener('click', function () {
          currentState.columnClick(col);
          event.preventDefault();
      });
  });
```

这样，函数(*)的每次调用都会创建一个新的变量 `col`，而且不会有无意识共享发现。

本系列的最后一篇文章会讲解用 ECMAScript6 来处理无意识共享的问题。

## 3、引用

[1] [JavaScript quirk 6: the scope of variables][1]

[1]: 6-the-scope-of-variables.md
