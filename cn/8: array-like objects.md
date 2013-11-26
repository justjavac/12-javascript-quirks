JavaScript 的怪癖 8：“类数组对象”
===

原文： [JavaScript quirk 8 array-like objects](http://www.2ality.com/2013/05/quirk-array-like-objects.html)

译者： [Malcolm Yu](https://github.com/malcolmyu)

---

此文是 [javascript 的 12 个怪癖（quirks）](https://github.com/justjavac/12-javascript-quirks) 系列的第八篇。

JavaScript中有一些看起来像却又不是数组的对象，唤作**类数组**。
本文旨在探究类数组的确切含义和高效的使用方式。

## 类数组

一个类数组对象：

* 具有：指向对象元素的数字索引下标以及 `length` 属性告诉我们对象的元素个数
* 不具有：诸如 `push` 、 `forEach` 以及 `indexOf` 等数组对象具有的方法

两个典型的类数组的例子是：DOM方法 `document.getElementsByClassName()` 的返回结果（实际上许多DOM方法的返回值都是类数组）以及特殊变量 `arguments` [[1][]]。例如你可以通过以下方法确定函数参数的个数：

```javascript
arguments.length
```

你也可以获取单个参数值，例如读取第一个参数：

```javascript
arguments[0]
```

如果这些对象想使用数组的方法，就必须要用某种方式“借用”。由于大部分的数组方法都是通用的，因此我们可以这样做。

## 通用方法

所谓的通用方法就是不强制要求函数的调用对象 `this` 必须为数组，仅需要其拥有 `length` 属性和数字索引下标即可。
通常来讲，你可以用如下的方式在数组 `arr` 上调用方法 `m` ：

```javascript
arr.m(arg0, arg1, ...)
```

所有的函数都拥有一个 `call` 方法来让我们用这样一种方式进行上述调用：

```javascript
Array.prototype.m.call(arr, arg0, arg1, ...)
```

`call` 方法的第一个参数就是函数 `m` 的调用对象 `this` 的值（在这个例子里就是 `arr`）。
因为我们直接调用方法 `m` ，而非通过数组对象 `arr` ，因此我们可以为本方法更改任意的 `this` 值。

例如改为 `arguments` :

```javascript
Array.prototype.m.call(arguments, arg0, arg1, ...)
```

### 例子

让我们来看一个具体的例子。
下面的 `printArgs` 列出了函数的全部参数值。

```javascript
function printArgs() {
    Array.prototype.forEach.call(arguments,
        function (arg, i) {
            console.log(i+'. '+arg);
        });
}
```

我们“通用地”使用了方法 `forEach`。
`printArgs` 的运行结果如下：

```shell
    > printArgs()
    > printArgs('a')
    0. a
    > printArgs('a', 'b')
    0. a
    1. b
```

你甚至可以应用通用方法给普通的对象：

```shell
    > var obj = {};
    > Array.prototype.push.call(obj, 'a');
    1
    > obj
    { '0': 'a', length: 1 }
```

在上述例子中，`length` 属性原本不存在并以0为初始值自动创建。

### 将类数组对象转化为数组

有时候处理类数组对象的最好方法是将其转化为数组。
这项工作也可以使用通用方法来完成：

```javascript
Array.prototype.slice.call(arguments)
```
对于正常复制数组对象而言，我们额外使用了 `call` 的方法。

```javascript
arr.slice()
```

## 引用

1. [JavaScript 的怪癖 5：参数的处理][1]

[1]: https://github.com/justjavac/12-javascript-quirks/blob/master/cn/5-parameter-handling.md
