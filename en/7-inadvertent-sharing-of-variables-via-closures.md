7 inadvertent sharing of variables via closures
=========================

原文：[JavaScript quirk 7 inadvertent sharing of variables via closures](http://www.2ality.com/2013/05/quirk-closures.html)

译文：[JavaScript 的怪癖 7：闭包和自由(外部)变量]()

译者：[未翻译]()

----------------------------------------------------

此文是 [javascript 的 12 个怪癖（quirks）](http://justjavac.com/javascript/2013/04/08/12-javascript-quirks.html) 系列的第七篇。

Closures are a powerful JavaScript feature: If a function leaves the place where it was created, 
it still has access to all variables that existed at that place. 
This blog post explains how closures work and why one has to be careful w.r.t. inadvertent sharing of variables.

## Closures

Let’s start with an example of a closure:

```javascript
function incrementorFactory(start, step) {
    return function () {  // (*)
        start += step;
        return start;
    }
}
```

This is how you use `incrementorFactory`:

```javascript
> var inc = incrementorFactory(20, 2);
> inc()
22
> inc()
24
```

During all of its lifetime, the inner function (*) has access to the variables `start` and `step` of the outer function `incrementorFactory`. 
Thus, `incrementorFactory` returns not only the function, but somehow attaches the variables `start` and `step`. 
The data structure in which both variables are stored is called an `environment`. 
An environment is very similar to an object – it maps names to values. 
The function that is returned above contains a reference to the environment that was active at its birth, its **outer environment**. 
The combination function + environment is called a **closure**. 
The name stems from the fact that an environment “closes over” a function: It provides values for variables that were declared outside the function (so-called **free variables**).

When a function f is invoked, a new environment is created for its parameters and local variables. 
There is always a chain of environments:

* f’s environment
* f’s outer environment
* The outer environment of f’s outer environment
* ...
* The environment for global variables (the **global environment**)

When looking up the value of a variable, the complete chain is searched, starting with f’s environment.

## The quirk: inadvertent sharing

Closures don’t get snapshots of a certain point in time, they get “live” variables. 
The following is an example where that causes a problem.

```javascript
var result = [];
for (var i=0; i < 5; i++) {
    result.push(function () { return i });  // (*)
}
console.log(result[3]()); // 5 (not 3)
```

When a function is created in line (*), the variable `i` has a certain value. 
You might expect that function to always return that value. 
Instead, the connection to the “live” `i` is never broken. 
That is, all functions in the array `result` share the same `i`, via their outer environment.
And after the loop is finished, `i` has the value 5.

One possible fix is to copy the current value of `i` via an IIFE [[1][]]:

```javascript
for (var i=0; i < 5; i++) {
    (function (i2) {  // snaphot of i
        result.push(function () { return i2 });
    }(i));
}
```

You can also use [bind()][bind], with a similar effect:

[bind]: https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Function/bind

```javascript
for (var i=0; i < 5; i++) {
    result.push(function (i2) { return i2 }.bind(null, i));
}
```

Another possibility is using `forEach` and the [range()][range] function of the Underscore.js library:

[range]: http://underscorejs.org/#range

```javascript
_.range(5).forEach(function (i) {
    result.push(function () { return i });
});
```

The above works, because `forEach` creates a new variable `i`, each time it calls its argument.

### A practical example

Let’s conclude with a more practical example. 
Two days ago, I implemented a user interface for the game [Connect Four][connect_four], as a demonstration of the DOM. 
It contained the following code snippet, which adds event listeners to links above the columns of the board.

[connect_four]: http://en.wikipedia.org/wiki/Connect_Four

```javascript
for(var col=0; col < board4.DIM_X; col++) {
    document.getElementById('columnClick'+col)
            .addEventListener('click', function (col) {
                currentState.columnClick(col);
                event.preventDefault();
            }.bind(null, col));
}
```

An alternative is to use CSS classes instead of IDs and rewrite the above code:

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

Again, each invocation of the function (*) creates a new variable `col` and no inadvertent sharing occurs.

The last post in this series will explain how ECMAScript 6 helps with the problem of inadvertent sharing.

## Reference

1. [JavaScript quirk 6: the scope of variables][1]

[1]: http://www.2ality.com/2013/05/quirk-variable-scope.html

