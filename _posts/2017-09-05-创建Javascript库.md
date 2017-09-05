---
layout: post
title: 优雅的创建一个JavaScript库
description: 这篇文章的目的是通过演示一个简单的例子来介绍在JS中实例化和定义一个库的正确方法，以优化他人编写或维护自己的JS库。
categories: 
 - JAVASCRIPT
tags:
 - 翻译
---

>在我们深入之前，我做了两点假设：
>* 你知道简单的JavaScript或C语言。
>* 你不打算使用jQuery。通常情况下，一个JavaScript库不需要任何依赖。

<!-- more -->

## 简单定义
首先，我遇到了第一个麻烦，即如何正确的看待一个JavaScript库。在C/C++中，一个库是功能的集合，并且通常不需要很完美的结构。
而JavaScript的工作方式有所不同，因此我做了一些研究。最后的结论是，一个JavaScript库是一个包含在对象中的独立的模块，不会在自己的作用域以外定义函数来污染全局命名空间。

基于此，我们可以简单的定义一个库：
```javascript
var Library = {
    name: "Meng",
    greet: function() {
        alert("Hello from the " + Library.name + " library.");
    }
}
```

这个库就是我们字面意思上所说的对象。要调用`greet`函数，我们要用：
```javascript
Library.greet()
```

因为`greet`是`Library`对象的成员。现在，如果我们期望name变量为Library私有，应该怎么做呢？遗憾的是，这种库的定义方式无法实现这一点。

而且，不仅仅是在其他文件中要通过`Library.name`来引用`name`变量，在`Library`中也要通过引用自己来引用`name`变量。这种解决方式看起来就很差劲。

## 私有化
解决私有化问题的方法是将`Library`定义为一个函数，在函数中定义对象。这种方式允许我们创建并使用私有的变量和方法。如下：
```javascript
function Library() {
    var name = "Candy";
    this.greet = function() {
        alert("Hello from the " + name + " library.");
    }
}
```
在这种方式下，`Library.name`将不会暴露在全局，并且在`Library`内部也能被简单的引用。`greet`方法依然是公有的，因为通过关键字`this`将`greet`定义为`Library`的一个成员。

> 确定将什么暴露给谁，是架构的简单实践。

## 规避命名冲突
但是，现在我们有了新的问题。在如下场景中，用户已经定义了一个`Library`对象，并且引用了你的库。
```javascript
//用户自己的Library对象
var Library = {
    book_num: 1123,
    category: [
        "science",
        "social"
    ]
}

//调用Library库中的方法
Library.greet();
```
这时，你的`Library`定义被覆盖，用户还将被告知“引用错误”。

我们有一个更好的解决方案：将你的`library`包装在一个函数中，这个函数将在不存在命名冲突时被调用。如下：

```javascript
(function(window) {
    'use strict';
    function define_library() {
        var Library = {};
        var name = "Candy";
        Library.greet = function() {
            alert("Hello from the " + name + " library.");
        }
        return Library;
    }
    
    if (typeof(Library) === "undefined") {
        window.library = define_library();
    } else {
        console.log("Library is already defined.");
    }
})(window);
```
**首先**，我们解释一下包装整个`library`的闭包:
```javascript
(function(window) {
    //CODE
})(window);
```
它的工作原理是，它包装了在它其中定义的代码，因此它拥有自己的命名空间。它也将会自动执行，因为末尾添加的`(window)`。

**接着**，我们介绍一下使用的`use strict`。这是一个可选配置，可以在[这里](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/Strict_mode)了解更多相关知识。由于我们整个定义是一个闭包，所以只需声明一次`use strict`，即可将严格模式应用于整个`library`。

**然后**，我们在`define_library()`函数中定义了`Library`对象，并且在函数最后返回了该对象。

**最后**，我们通过`typeof`判断当前是否有命名冲突，如果没有，我们将初始化我们的`Library`对象。