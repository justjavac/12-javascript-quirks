JavaScript 的怪癖 5：参数的处理
---

原文：[JavaScript quirk 5: parameter handling](http://www.2ality.com/2013/05/quirk-parameters.html)

译者：[zhmengqing](http://me.9tech.cn/index.php/homepage/main/96622)

----------------------------------------------------

此文是 [javascript 的 12 个怪癖（quirks）](http://justjavac.com/javascript/2013/04/08/12-javascript-quirks.html) 系列的第五篇。

JavaScript 参数处理的基本原理很简单，高级的任务都需要手动操作。
本文首先关注其基本原理然后再行扩展。

## 1、参数处理的基本原理

JavaScript 的参数处理包括两个要点

### 1.1、要点：你可以传递任意数量的参数

当调用一个 `function` 时，你想传递多少参数都可以，这与该函数声明了多少个正式的参数无关。
缺失参数的值是 `undefined`，多出来的参数则直接被忽略掉。

我们用以下的函数做个示范：

```javascript
function f(x, y) {
    console.log('x: '+x);
    console.log('y: '+y);
}
```
    
你可以用任意数量的参数调用这个 `function`：

```javascript
> f()
x: undefined
y: undefined

> f('a')
x: a
y: undefined

> f('a', 'b')
x: a
y: b

> f('a', 'b', 'c')
x: a
y: b
```
  
### 1.2要点：所有传递的参数都储存在 arguments 中

所有传递的参数都储存在一个很特别、很像 `Array`（继续看就能知道为什么了）的变量里，`arguments`。
通过下面的 `function` 我们来看下这个变量怎么用的：

```javascript
function g() {
    console.log('Length: '+arguments.length);
    console.log('Elements: '+fromArray(arguments));
}
```

下面是 `fromArray` 函数，它把 `arguments` 转换成 `array` 这样就能存入数据了，调用 `g()`：

```javascript
> g()
Length: 0
Elements:
> g('a')
Length: 1
Elements: a
> g('a', 'b')
Length: 2
Elements: a,b
```

无论明确声明了多少个参数，`arguments` 是永远在那里的，它总是包含所有实际的参数。

## 2、参数传递了吗？

如果调用者没有提供参数，那么 `undefined` 就会传递给 `function`。
因为 `undefined` 是一个虚拟值[[1][]]，你可以用一个 `if` 条件语句来检验它是否存在：

```javascript
function hasParameter(param) {
    if (param) {
        return 'yes';
    } else {
        return 'no';
    }
}
```

这样，你不传参数与传入 `undefined` 获得的结果是一样的：

```javascript
'no'
> hasParameter(undefined)
'no'
```

测试代码对真实值(truthy)同样有效：

```javascript
> hasParameter([ 'a', 'b' ])
'yes'
> hasParameter({ name: 'Jane' })
'yes'
> hasParameter('Hello')
'yes'
```

而对于虚拟值(falsy)的会用是需要多加小心的。
比如 `false`、`0` 以及空字符串都被解析为缺失参数：

```javascript
> hasParameter(false)
'no'
> hasParameter(0)
'no'
> hasParameter('')
'no'
```

这段代码足以证明。
你必须要多加注意，因为代码变得更加紧凑与调用者是否忽略了一个参数还是传递了 `undefined` 或者 `null` 都无关。

## 3、参数的默认值

以下的 `function` 可以传入 `0` 或者其他参数，`x` 和 `y` 如果未传参数则会是 `0`，以下是一种表现方式：

```javascript
function add(x, y) {
    if (!x) x = 0;
    if (!y) y = 0;
    return x + y;
}
```

交互后：

```javascript
> add()
0
> add(5)
5
> add(2, 7)
9
```

你可以用 `or` 运算符（||）使 `add()` 更简洁。
如果为真这个运算符会返回第一个值否则返回第二个。

例如：

```javascript
> 'abc' || 'def'
'abc'
> '' || 'def'
'def'
> undefined || { foo: 123 }
{ foo: 123 }
> { foo: 123 } || 'def'
{ foo: 123 }
```

我们用 `||` 来指定参数默认值：

```javascript
function add(x, y) {
    x = x || 0;
    y = y || 0;
    return x + y;
}
```

## 4、任意数量的参数

你也可以用 `arguments` 来接收任意数量的参数，其中一个例子是以下的函数 `format()`，它在 C 函数 `sprintf` 之后输出语句：

```javascript
> format('Hello %s! You have %s new message(s).', 'Jane', 5)
'Hello Jane! You have 5 new message(s).'
```

第一个参数是一个样式，由 `%s` 标记空白，后面的参数则填入这些标记，简单的 `format` 函数实现如下：

```javascript
function format(pattern) {
    for(var i=1; i < arguments.length; i++) {
        pattern = pattern.replace('%s', arguments[i]);
    }
    return pattern;
}
```

注意：循环跳过了第一个参数(`arguments[0]`) 并且忽略了 `pattern`。

## 5、强制执行一定数量的参数

如果你想要强制调用者执行一定数量的参数，你就要在运行阶段检查 `arguments.length`：

```javascript
function add(x, y) {
    if (arguments.length > 2) {
        throw new Error('Need at most 2 parameters');
    }
    return x + y;
}
```

## 6、arguments 不是 array

`arguments` 并不是 `array`，它只是很像 `array`，你可以获取第 `i` 个参数比如 `arguments[i]`，
你也可以检查它有多少个参数比如 `arguments.length`。
但是你不能用 `Array` 的方法如 `forEach` 或者 `indexOf`。
更多详情与解答会在「怪癖8（未翻译）」中进行讨论，作为一个预习，以下函数能将一个类似 `array` 的值转换为 `array`：

```javascript
function fromArray(arrayLikeValue) {
    return Array.prototype.slice.call(arrayLikeValue);
}
```

## 7、参考

[1] [JavaScript quirk 1: implicit conversion of values][1] [解释了“真实值(truthy)”与“虚拟值(falsy)”]

[1]: 1-implicit-conversion-of-values.md
