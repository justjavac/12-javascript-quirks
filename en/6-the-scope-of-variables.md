6: the scope of variables
=========================

原文：[JavaScript quirk 6: the scope of variables](http://www.2ality.com/2013/05/quirk-variable-scope.html)

译文：[JavaScript 的怪癖 6：变量的作用范围](#)

译者：[未翻译](#)

----------------------------------------------------

此文是 [javascript 的 12 个怪癖（quirks）](http://justjavac.com/javascript/2013/04/08/12-javascript-quirks.html) 系列的第六篇。

In most programming languages, variables only exist within the block in which they have been declared. 
In JavaScript, they exist in the complete (innermost) surrounding function:

```javascript
function func(x) {
    console.log(tmp); // undefined
    if (x < 0) {
        var tmp = 100 - x;  // (*)
        ...
    }
}
```

The cause of the above behavior is hoisting: Internally, the declaration at (*) is moved to the beginning of the function (the assignment stays where it is). 
That is, what a JavaScript engine actually executes looks like this:

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

But there is a trick to limit the scope of a variable to a block, it is called Immediately Invoked Function Expression (IIFE, pronounced “iffy”). 
Below, we use an IIFE to restrict the scope of `tmp` to the then-block of the `if` statement.

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

We wrapped a function around the insides of the block, creating a new scope for them. 
Then we immediately called that function. 
`tmp` only exists inside the IIFE. 
Note that the parentheses at the beginning and the end of the IIFE are necessary. 
They lead to the function being interpreted as an expression, which is the only form in which it can be be immediately invoked.
