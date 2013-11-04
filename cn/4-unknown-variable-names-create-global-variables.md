JavaScript 的怪癖 4：未知变量名创建全局变量
---

原文：[JavaScript quirk 4: unknown variable names create global variables](http://http//www.2ality.com/2013/04/quirk-automatic-globals.html)

译者：[justjavac](http://weibo.com/justjavac)

----------------------------------------------------

此文是 [javascript 的 12 个怪癖（quirks）](https://github.com/justjavac/12-javascript-quirks) 系列的第四篇。

当你使用了一个未知的变量名，通常 JavaScript 会自动创建全局变量：

  > function f() { foo = 123 }
  > f()
  > foo
  123

好在你会在 ECMAScript5 的严谨模式得到警告[[1][]]：

  > function f() { 'use strict'; foo = 123 }
  > f()
  ReferenceError: foo is not defined

## 参考

[1] [JavaScript’s strict mode: a summary][1]

[1]: http://www.2ality.com/2011/01/javascripts-strict-mode-summary.html

