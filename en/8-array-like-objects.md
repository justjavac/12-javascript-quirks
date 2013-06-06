8: array-like objects
=========================

原文：[JavaScript quirk 8 array-like objects](http://www.2ality.com/2013/05/quirk-array-like-objects.html)

译文：[JavaScript 的怪癖 8：“对象、数组——傻傻分不清楚”（Array-like objects）]()

译者：[未翻译]()

----------------------------------------------------

此文是 [javascript 的 12 个怪癖（quirks）](http://justjavac.com/javascript/2013/04/08/12-javascript-quirks.html) 系列的第八篇。

Some objects in JavaScript look like arrays, but aren’t. 
They are called **array-like**. 
This blog post looks at what exactly that means and how to best work with those objects.

## Array-like objects

An array-like object

* has: indexed access to elements and the property `length` that tells us how many elements the object has.

* does not have: array methods such as `push`, `forEach` and `indexOf`.

Two examples of array-like objects is the result of the DOM method `document.getElementsByClassName()` (many DOM methods return array-like objects) and the special variable arguments [[1][]]. You can determine the number of arguments via

```javascript
arguments.length
```

And you can access a single argument, e.g. read the first argument:

```javascript
arguments[0]
```

Array methods, however, have to be borrowed. You can do that, because most of those methods are generic.

## Generic methods

A generic method does not require `this` to be an array, it only requires `this` to have `length` and indexed element access. 
Normally, you invoke a method `m` on an array `arr` as follows.

```javascript
arr.m(arg0, arg1, ...)
```

All functions have a method `call` that allows you to perform the above invocation differently:

```javascript
Array.prototype.m.call(arr, arg0, arg1, ...)
```

The first argument of `call` is the value for `this` that `m` receives (in this case, `arr`). 
Because we access `m` directly and not via `arr`, we can now hand any `this` to that method. 

For example, `arguments`:

```javascript
Array.prototype.m.call(arguments, arg0, arg1, ...)
```

### Examples

Let’s continue with a concrete example. 
The following function `printArgs` logs all arguments that it receives. 

```javascript
function printArgs() {
    Array.prototype.forEach.call(arguments,
        function (arg, i) {
            console.log(i+'. '+arg);
        });
}
```

We have used method `forEach` generically. 
`printArgs` in use:

```shell
    > printArgs()
    > printArgs('a')
    0. a
    > printArgs('a', 'b')
    0. a
    1. b
```

You can even apply generic methods to ordinary objects:

```shell
    > var obj = {};
    > Array.prototype.push.call(obj, 'a');
    1
    > obj
    { '0': 'a', length: 1 }
```

In the above case, property `length` did not exist and was automatically created, with the initial value zero.

### Converting an array-like object to an array

Sometimes the best way to work with an array-like object is to convert it to an array. 
That can also be done via a generic method:

```javascript
Array.prototype.slice.call(arguments)
```

Compare: to create a copy of an array `arr`, you make the method `call`

```javascript
arr.slice()
```

## Reference

1. [JavaScript quirk 5: parameter handling][1]

[1]: http://www.2ality.com/2013/05/quirk-parameters.html
